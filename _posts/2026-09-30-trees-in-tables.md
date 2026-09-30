---
title: "Trees in Tables: What Four Hierarchy Models Taught Me"
categories:
 - Blog
tags:
 - data engineering
 - data modeling
 - hierarchy
---


Hierarchies are everywhere in data: organisation charts, product catalogues, cost centres, sales regions, geographies. They look innocent. A tree is a tree, you just need to put it in a table. And yet, few things have broken as many of my pipelines as a hierarchy that changed.


Over the years, I have worked with four different ways of storing a hierarchy. I describe them here from worst to best, together with what each one broke, so that I (and maybe you) stop repeating the same mistakes. I close with the piece that was still missing in the best of them, and how I would add it.


To keep the examples concrete, I will use a small organisation: divisions contain departments, departments contain teams, and employees belong to teams.


### Experience 1: One column per level


The first hierarchy lived in a single file. Each column was a level, and reading from left to right went from ancestor to descendant. The columns were named, so the order was mostly there for humans. The hierarchy was dense: every row was filled down to the last level, and the leaf was unique. In other word a team did not exist if it had no employees in it. A final column recorded which entity belonged to which leaf.


| division | department  | team          | employee |
|----------|-------------|---------------|----------|
| Europe   | Sales       | Sales DACH    | alice    |
| Europe   | Sales       | Sales Nordics | bob      |
| Americas | Sales       | Sales US      | carol    |
| Americas | Engineering | Platform      | dave     |


It is easy to read and easy to open in a spreadsheet. It is also a trap, because the names must act as the identifiers.


**Renaming a leaf broke code.** When "Sales DACH" became "Sales Central Europe", every filter, mapping and downstream table that mentioned the old name had to be found and changed. There was no id to fall back on.


**Renaming an upper level was fine... until it wasn't.** Renaming a department only touched this file, unless that department name was also used somewhere else, for example as a key in a budget file. Then that other place broke.


**Upper-level names were not unique.** "Sales" exists under Europe *and* under Americas. Joining on the level and the node name was not enough:


```sql
-- Looks right, but matches both "Sales" departments
JOIN budget b
 ON b.department = h.department


-- What every join actually needed: the full path
JOIN budget b
 ON b.division   = h.division
AND b.department = h.department
```


Every join had to carry the full path down to the level of interest. Forget one column and your numbers quietly double.


**Temporal changes were barely feasible.** Merging two teams or splitting one meant rewriting names across rows, and the only history was older versions of the file. Worse, in a name-keyed world a rename and a "delete one team, create another" look exactly the same. Nothing in the data tells you which one happened.


*Lesson: a name is a label, not an identity. Names change.*


### Experience 2: Ids that encode the path


The second hierarchy was an improvement. Every node had an id from a numbering scheme. The catch: the id of a node was built from the ids of its ancestors. The full path to the node, including the node itself, was encoded in it.


| node_id  | name          |
|----------|---------------|
| A0       | Europe        |
| A0B1     | Sales         |
| A0B1C003 | Sales DACH    |
| A0B1C004 | Sales Nordics |
| A1       | Americas      |
| A1B1     | Sales         |
| A1B1C001 | Sales US      |


This solved the renaming problem. When the text attached to `A0B1C003` was corrected, nothing downstream had to change, because downstream processes used the id, not the name. The two "Sales" departments were no longer ambiguous either: `A0B1` and `A1B1` are different keys. As a bonus, finding all descendants of a node is a simple prefix match (`node_id LIKE 'A0B1%'`).


The problem showed up as soon as the structure moved. When a node changed parent, its id changed. Move "Sales DACH" from Sales (`A0B1`) to a new department `A0B2`, and it becomes `A0B2C003`, or something else entirely if `C003` was already taken there. And if the moved node is not a leaf, all of its descendants get new ids too.


For everything referencing the node, `A0B1C003` simply disappeared and `A0B2C003` appeared from nowhere. To follow a team through a reorganisation, we needed a mapping table from old ids to new ids. Just as in experience 1, a move and a "delete plus create" were indistinguishable in the data, only this time for structure instead of names.


*Lesson: an identifier should identify, not describe. If the position is part of the id, the id breaks when the position changes.*


### Experience 3: Nodes with a parent pointer


The third hierarchy separated these concerns:


- Each level has an id (a UUID), a code, and a name.
- Each node has an id, a code, and a name, and points to its level id and to its parent id (null for a root).
- The node id and the node code are both globally unique.


```sql
CREATE TABLE hierarchy_level (
   level_id  UUID PRIMARY KEY,
   code      TEXT NOT NULL UNIQUE,
   name      TEXT NOT NULL
);


CREATE TABLE hierarchy_node (
   node_id   UUID PRIMARY KEY,
   code      TEXT NOT NULL UNIQUE,
   name      TEXT NOT NULL,
   level_id  UUID NOT NULL REFERENCES hierarchy_level (level_id),
   parent_id UUID NULL     REFERENCES hierarchy_node (node_id)  -- NULL for roots
);
```


Employees reference `node_id`, and nothing else. Now each kind of change touches exactly what it should.


**Rename:** update `name`. The id and the code are untouched, so nothing downstream moves.


**Move to another parent:** update a single `parent_id`. The node keeps its id, its descendants keep theirs, and every reference stays valid.


```sql
UPDATE hierarchy_node
  SET parent_id = :new_department_id
WHERE node_id   = :sales_dach_id;
```


**Change of level:** this is the one change that still costs something. The node's `level_id` changes, and so does the `level_id` of all its descendants.


The code sits next to the UUID for a reason: the UUID is for machines and joins, while the code is a stable, readable key that people can put in a file or a ticket without depending on the display name.


Interestingly, the wide format from experience 1 did not disappear. It became an *output*. When a report needs one column per level, a recursive query rebuilds it from the parent pointers:


```sql
WITH RECURSIVE tree AS (
   SELECT node_id, name, parent_id, name::TEXT AS path
     FROM hierarchy_node
    WHERE parent_id IS NULL
   UNION ALL
   SELECT n.node_id, n.name, n.parent_id, t.path || ' > ' || n.name
     FROM hierarchy_node n
     JOIN tree t ON n.parent_id = t.node_id
)
SELECT node_id, path FROM tree;
```


The flat table is a fine view. It is a terrible source of truth.


What this model lacked was time. The `UPDATE` above overwrites the old parent: once Sales DACH has moved, nothing remembers where it used to be.


*Lesson: store the relationships, derive the paths.*


### Experience 4: Nodes with a validity period


The fourth hierarchy kept the model of experience 3 and added four columns to every node: `valid_from`, `valid_to`, `created_at`, and `updated_at`.


The first two describe *business time*: the period during which a version of the node was true in the real world. The last two describe *system time*: when the row was written and when it was last touched.


A rename or a move no longer overwrites anything. The current version is closed, and a new version with the same `node_id` is opened (parents shown by name for readability):


| node_id | name       | parent     | valid_from | valid_to   |
|---------|------------|------------|------------|------------|
| n42     | Sales DACH | Sales      | 2022-01-01 | 2026-03-01 |
| n42     | Sales DACH | Commercial | 2026-03-01 | 9999-12-31 |


The key of the table becomes `(node_id, valid_from)`, and the tree as it was on any given date is one filter away:


```sql
SELECT *
 FROM hierarchy_node
WHERE valid_from <= :as_of
  AND valid_to   >  :as_of;
```


Treating the interval as half-open, `[valid_from, valid_to)`, with a far-future `valid_to` for the current version, keeps this query simple and avoids overlapping or missing days at each change.


This finally answered part of the question left open by experience 1. A rename is the same id with a new name. A move is the same id with a new parent. Both are visible, with their dates. And because `created_at` is separate from `valid_from`, we could tell when a reorganisation took effect from when it was entered, which matters when a change is recorded after the fact.


A level change is still the expensive one: it now creates a new version for the node and for every one of its descendants.


*Lesson: keep versions instead of overwriting, and keep "when it was true" apart from "when we recorded it".*


### What experience 4 still does not solve: merges and splits


Imagine Sales DACH and Sales Nordics merge into Sales Europe North on 1 April. In experience 4, the data shows two nodes closing and a new one opening on the same day. That they are related is a guess based on matching dates. Nothing actually states it.


A split is worse. Platform closes; Platform Core and Data Platform open. Which part of Platform went where? If you want to compare this year's Data Platform costs with last year's, the model has nothing to offer.


The reason is structural. A merge or a split is not an attribute of a node; it is a relationship *between* nodes, across time. It is many-to-one for a merge, one-to-many for a split, and sometimes many-to-many when a reorganisation does both at once. A `replaced_by` column on the old node handles merges but not splits. A `replaces` column on the new node handles splits but not merges. No column on a single row can hold both.


### How I would model merges and splits


The approach I would take next is to record the transformation itself as data: an event table, and a lineage table that links predecessor nodes to successor nodes.


```sql
CREATE TABLE hierarchy_event (
   event_id       UUID PRIMARY KEY,
   event_type     TEXT NOT NULL,   -- 'merge', 'split', 'reorganisation', ...
   effective_date DATE NOT NULL,
   description    TEXT,
   created_at     TIMESTAMP NOT NULL
);


CREATE TABLE hierarchy_lineage (
   event_id       UUID    NOT NULL REFERENCES hierarchy_event (event_id),
   predecessor_id UUID    NOT NULL,  -- node_id of the node that ends
   successor_id   UUID    NOT NULL,  -- node_id of the node that takes over (part of) it
   share          NUMERIC NOT NULL CHECK (share > 0 AND share <= 1),
   PRIMARY KEY (event_id, predecessor_id, successor_id)
);
```


The merge and the split from above become:


| event       | predecessor   | successor          | share |
|-------------|---------------|--------------------|-------|
| merge 04-01 | Sales DACH    | Sales Europe North | 1.0   |
| merge 04-01 | Sales Nordics | Sales Europe North | 1.0   |
| split 07-01 | Platform      | Platform Core      | 0.6   |
| split 07-01 | Platform      | Data Platform      | 0.4   |


`share` says how much of a predecessor went into each successor. In a merge, each predecessor goes entirely into the new node. In a split, the shares of a predecessor add up to 1.


With that, restating history in today's structure becomes a query: follow the lineage forward and multiply the shares along the way. It has to be recursive, because chains happen: a team that was merged last year may be split this year.


```sql
WITH RECURSIVE forward AS (
   SELECT predecessor_id AS origin_id, successor_id AS node_id, share
     FROM hierarchy_lineage
   UNION ALL
   SELECT f.origin_id, l.successor_id, f.share * l.share
     FROM forward f
     JOIN hierarchy_lineage l ON l.predecessor_id = f.node_id
)
SELECT origin_id, node_id AS current_node_id, share
 FROM forward
WHERE node_id NOT IN (SELECT predecessor_id FROM hierarchy_lineage);
```


This gives, for every retired node, the current nodes it ended up in and in which proportion. Join historical facts on `origin_id`, multiply by `share`, and last year's Platform costs appear under Platform Core and Data Platform.


A few rules make this work in practice:


- **Decide when identity continues.** If one team absorbs a small one and its scope barely changes, it keeps its id: only the absorbed team closes, with a lineage row pointing to the survivor. If the meaning changes, close all the nodes involved and open new ones. Either way, write the rule down before the first reorganisation, not during it.
- **Shares depend on the measure, and are ideally not needed.** A 60/40 split of costs may be 80/20 for headcount. When facts hang off members, such as employees, rather than off nodes, you can restate history by following the members and skip the shares entirely. Shares are only needed for facts recorded directly at node level, such as budgets or targets.
- **Validate the lineage.** It must be acyclic. Predecessors must close on the effective date and successors must open on it. Within an event, the shares of each predecessor must add up to 1.


None of this is new. Statistical offices have done it for decades with correspondence tables, or crosswalks, whenever regions or classifications are redrawn. It just rarely makes it into the hierarchies we build ourselves.


### Takeaways


Looking back, each step removed one thing that was pretending to be an identity, or one change that was pretending not to happen:


1. **Names are not identities.** Give every node a stable id; names are attributes, and they will change.
2. **Positions are not identities.** Do not encode the path in the id; the structure will change too.
3. **Store edges, derive paths.** A parent pointer turns a move into a one-row change. The flat, one-column-per-level table is something you generate, not something you maintain.
4. **Keep a human code next to the technical id.** People need a readable key that is not the display name.
5. **Version instead of overwriting.** Keep "when it was true" (`valid_from`, `valid_to`) apart from "when we recorded it" (`created_at`, `updated_at`).
6. **Model changes, not only states.** Merges and splits are relationships between nodes; give them their own table.


The real test, I think, is this. Can your data tell the difference between "this team was renamed", "this team was moved", and "this team was replaced", and, if it was replaced, by what? If not, your hierarchy will hurt you the first time the organisation changes.



