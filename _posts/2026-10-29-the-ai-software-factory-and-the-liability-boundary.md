---
title: "The AI Software Factory and the Liability Boundary"
categories:
  - Blog
tags:
  - AI
  - software engineering
  - liability
---

Every few years, software engineering attempts to formalize itself by borrowing metaphors from physical manufacturing. We talk about assembly lines, blueprints, tooling, and factories. Claude, Codex, Opencode, and any other harnesses may be seen as a versatile "Software Factory", supporting vibe coding or AI-assisted coding. For fun projects, aka non-productive projects, these are powerful multipliers; but for productive applications within a company which may interact with other systems, it does not seem to be there yet. Non-technical users potentially need a stricter, more opinionated setup.

By wrapping modern AI harnesses (Claude, Codex, Antigravity, etc.) in an opinionated harness of software engineering practices, including but not limited to DDD (Domain-Driven Design), BDD (Behavior-Driven Development), Example Mapping, OpenSpecs, Event Modeling, continuous integration, and strict test runners, we theoretically create a "safe sandbox." Within this sandbox, business managers, domain experts, and non-devs can build working software without shooting themselves in the foot at some point during building or productive usage.

It sounds like the holy grail of democratized software development. But there is a fundamental flaw in how we think about rule-bending, AI, and engineering guarantees.

### Software Engineering is an Art of Bending Rules

Unlike civil or mechanical engineering, software engineering is rarely a hard science. It is a discipline of structured opinions, tradeoffs, and context-dependent rules.

- **Rule:** Never write tight coupling between modules.
- **Senior Dev Reality:** "Except in this isolated micro-service hotpath where network latency is killing us, so we'll inline this contract for now."
- **Rule:** Always stick to strict BDD and write specs first.
- **Senior Dev Reality:** "Except for this exploratory work where we don't know what the data structure looks like yet, and need to clarify."

*Note: you might differ in opinion, and I will as well in some cases.*

A senior software engineer isn't just someone who knows the rules; they are someone who knows when, where, and how far the rules can be bent without destroying the system. They understand the implications of rule-bending and are willing to take these risks.

Now, bring LLMs into the picture. AI harnesses excel at pattern matching, and if you allow them to bend rules they may bend the bent rules. When an AI encounters a tricky constraint or an edge case, it doesn't bend the rule with surgical precision. It writes shortcuts, bypasses guarantees, or silently breaks invariants across boundary lines.

In many current AI Dev Stacks (skills, agents, workbenches, or harnesses), we have an explosion of enabling tools but almost no guaranteeing tools (except perhaps some well-designed hooks and CI/CD).

### What Happens When You Try to Build an Airplane in a Bicycle Factory?

Let's go back to a physical-world analogy for a moment. Imagine a physical factory designed to manufacture bicycles.

It has specialized tooling: jig setups for welding steel or carbon frames, tensioners for spoked wheels, alignment rigs for derailleur gears. The processes are clear, the tolerance limits are established, and the safety specs are defined and approved.

Now, imagine a customer steps onto the factory floor and says: "Great setup! Let's use this line to build a commercial passenger jet."

Any reasonable plant manager would stop them instantly:

> This factory is built to guarantee bicycle safety within specific tolerance limits. It cannot guarantee aerodynamic stability, jet engine mounts, or pressurized cabin integrity. So we cannot build a plane.

In software development, however, we routinely let non-technical users attempt to build passenger jets inside a bicycle factory. Because software code looks visually homogeneous, non-devs assume that if the AI factory can scaffold a CRUD application or an internal dashboard, it can safely build a distributed transaction processor or a high-compliance healthcare portal.

### The Contract of Liability

If we are going to build software factories for non-technical operators, we need to talk about operating boundaries and liability.

A software factory cannot promise universal software creation; it can only promise stability within a strictly bounded domain. Beyond that boundary, the AI harness will bend the rules and at some point too far, and the architecture or system will collapse.

To make this explicit, every AI software factory needs a Contract of Liability:

```
                 +-----------------------------------+
                 |    AI Software Factory Boundary   |
                 |  (BDD, Example Mapping, Specs)    |
                 +-----------------------------------+
                                   |
           +-----------------------+-----------------------+
           |                                               |
  [ Inside Domain ]                               [ Outside Domain ]
  - Guaranteed specs                              - High risk of breaking rules
  - Deterministic tooling                         - Unknown architectural drift
  - Automated CI/CD guardrails                    - Unmapped invariants
           |                                               |
           v                                               v
  ( Standard Support )                         ( SIGNED LIABILITY WAIVER )
                                               "Engineering will NOT come
                                                to rescue you."
```

### The Rule of the Waiver

**Inside the Guarantee Zone:** The software factory provides high-friction guardrails (BDD specs, rigid example mapping, pre-built domain events, automated regression suites). If the user stays within these parameters, the factory guarantees functional continuity.

**Crossing the Boundary:** The moment a non-technical user asks the factory to build features outside its proven domain (e.g., adding custom cryptography, manual memory management, or non-standard protocol integrations), the system must stop and raise an explicit halt.

If the user insists on proceeding outside the factory's proven operational capabilities, they must digitally "sign" a contract of liability:

> You are stepping outside the factory's verified capabilities. By continuing, you acknowledge that you are bending engineering rules without architectural safety nets. Software Engineering will not come to the rescue when things break.

### Conclusion: Engineering is Boundary Management

Building a software factory isn't about giving an AI harness unlimited freedom to turn prompts into code. It is about defining the exact structural tolerances of what the factory can predictably produce.

Until we build tools that don't just enable generation, but actively guarantee function and enforce liability boundaries, non-tech company members will keep trying to fly carbon-fiber bicycles into the stratosphere and wondering why the wings fall off.
