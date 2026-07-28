---
title: "The Algorithmic Meeting: Stochastic Discourse Descent"
categories:
  - Blog
tags:
  - AI
  - LLM
  - debate
---

### Summary

- **Why:** Hard decisions deserve a meeting. LLMs can run that meeting at scale — if you structure disagreement instead of averaging coin flips.
- **What:** On GPQA Diamond with small models, debate beat the best solo debater by ~9 points and closed ~44% of the oracle gap. 53% of items were *debate lift* — correct finals after round-1 disagreement, mostly by rescuing a minority-correct view.
- **How:** Heterogeneous parallel debaters, temperature annealing, coordinator convergence + synthesis, optional optimizer-style steering, and scoped RAG memory — shipped as a plugin you can run, benchmark, and inspect.

GPQA is the showcase. The method is for everyone preparing the meeting before the meeting — and wanting a better answer than any single model could give alone.

### The meeting you already run, but reproducible

Before a hard decision, good teams do something predictable. They don’t ask one person in a quiet room and call it done. They hold a meeting, a workshop, create a task force in order to surface the uncomfortable angles, let people with different instincts and knowledge argue pros and cons, and only then commit.

That rhythm is old and valuable. What’s new is that large language models can somehow play every seat at the table, “cheaply”, in parallel, with a written record you can audit.

The problem with mono models harnesses is, if the model is wrong, you find out later, during implementation, management, or client interactions. Sampling the *same* model again might retrace the same blind spots. And therefore asking the model the same question twice is like flipping a sophisticated coin.

The alternative is not “ask GPT twice and vote.” It’s **heterogeneous debate**: different models, different personas, explicit rebuttal, and a coordinator who synthesizes what survives scrutiny. I (okay we: a horde of AI helpers were involved) built that loop as an open engine — [Stochastic Discourse Descent](https://github.com/bolirev/ai-sdd) — and treat it as an **algorithmic meeting** for anyone who needs rigor and different angles before they decide:

- **Product and architecture reviews** — trade-offs argued from multiple lenses before you lock a design.
- **Research and analysis** — one model decomposes, another stress-tests assumptions, a third looks for counterexamples.
- **Strategy and policy** — dissent is recorded, not buried; the final answer names what still disagrees.
- **Any hard question where explanation quality matters** — not trivia, but reasoning you can inspect.

We tested this approach on GPQA Diamond — graduate-level science multiple choice. But this is our showcase, not our ceiling. We deliberately ran small, cheap models (two Claude Haiku variants, two Gemini Flash variants, coordinated by one Haiku) because frontier models already perform well on GPQA alone. Showing lift with inexpensive debaters is the point: if debate helps *here*, it should help wherever your team mixes models on genuinely tough, open-ended work.

The question we care about is universal:

*When models disagree in round one, does structured argument create value or does group thinking destroy it?*

### Debate lifts and debates hurts (A little experiment)

We ran the debate harness on 198 GPQA Diamond items: “Google-proof” graduate science questions with four-way multiple choice. Each item gets four heterogeneous debaters in temperature-annealed rounds, then a coordinator synthesis. We compare the final answer to each debater’s solo round-1 answer (the “what would this model have said if we never met?”).

#### Four buckets: only two are interesting

Every question falls into one of four outcomes:

- **Trivial — all correct:** Every debater right in round 1; final answer correct. Cheap win, often early stop.
- **Trivial — all wrong and aligned:** No debater had the right path in round 1; final answer wrong. The ceiling when nobody had signal.
- **Debate lift** Final answer *correct*, but at least one debater was wrong in round 1.
- **Debate hurt** At least one debater was right in round 1, but the final answer is wrong. The risk to monitor.

For context, the original GPQA paper reports PhD-domain experts at ~65% on the main set. Four small models debating cleared that bar on the harder Diamond subset, while individually averaging ~56%.

The largest bucket is **debate lift: 106 items (53%)**. More than half of all questions are cases where models disagreed in round one and the coordinator still landed on the right answer. Trivial agreement (all right or all wrong) is the minority.

![](/assets/images/posts/the-algorithmic-meeting-stochastic-discourse-descent/01-e199588d01c8.png)

**Headline numbers:**

- **Debate final accuracy: 76.4%**
- **Best solo debater: 67.8%** → **+8.6 percentage points** from debate
- **Oracle (any debater right in R1): 87.4%** — the panel usually *contains* the answer
- **Gap closed: ~44%** of the distance from best solo to oracle

#### Mechanism: where the final answer came from

Figure 1 showed *that* debate helps on aggregate. Figure 2 asks *how* — it zooms into the two non-trivial buckets only: **debate lift** (final correct after round-1 disagreement) and **debate hurt** (someone was right in round 1, final wrong).

![](/assets/images/posts/the-algorithmic-meeting-stochastic-discourse-descent/02-7ebe2a9e5e9a.png)

**Panel A** splits those cases by round-1 truth: blue bars are wins where debate rescued a correct view (especially when only a minority — or one debater — had it); orange bars are failures where convergence overrode someone who was right. **Panel B** checks the transcripts: on lift items, debaters change their stated answer more often between rounds, and use more synthesis language (“combine,” “reconcile,” “consensus”) than in solo round 1 — evidence the meeting phase is doing work, not rubber-stamping.

Drill into lift and hurt:

- **~80% of lift cases rescue a minority-correct view.** One model had the right path; others didn’t. The coordinator found it. This is the core pre-meeting value — the dissenting engineer was right, and the meeting surfaced it.
- **Debate hurt is ~4× rarer than lift** (13% vs 53% of items). Convergence can amplify a shared mistake or override a lone correct debater — worth watching on high-stakes decisions, but not the dominant dynamic.
- **Debate rounds do real work.** By analysing full transcripts, items that lifted show higher synthesis language and non-trivial answer flips between rounds — not rubber-stamping round one.

#### What this means beyond GPQA

GPQA is a stress test with a crisp grade. Your meeting, your debate, will not have A/B/C/D keys — but the pattern should transfer wherever:

1.  Round-1 disagreement is common (different models, different personas).
2.  Explanation quality drives the outcome (not pattern-matching trivia).
3.  The cost of a wrong consensus exceeds the cost of extra tokens.

That’s architecture reviews, due diligence, scientific reasoning, policy drafts — anywhere you’d schedule a meeting before you ship.

### What’s under the hood

The engine is an [opencode](https://opencode.ai) plugin. You type /debate Should we adopt a monorepo? or call a debate tool from an agent. Underneath, x agents do the work.

#### 1. Heterogeneous parallel debaters

Each debater is a separate model session (optionally with a persona and a tool allowlist). They answer in parallel each round. Crucially, they see the *other* debaters’ previous-round positions inline — not a shared filesystem, but explicit argument exchange.\\ Divergence is a feature: if everyone agrees in round one, debate has nothing to exploit.

#### 2. Temperature annealing — argue hard, then converge

Round 1 runs hot ( **FIRM**: challenge assumptions, stake a position). Later rounds cool toward **CONSENSUS-SEEKING**. Temperature (not the one used by LLMS, but as an analogy to annealing processes) maps to a behavioral mode in one place: above 0.66 → firm; above 0.33 → balanced; else → consensus-seeking. Early rounds surface conflict; late rounds seek what survived scrutiny. It’s the algorithmic version of “let’s argue, then let’s decide.”

#### 3. A coordinator who judges and synthesizes

After each round, a coordinator model reads all positions and returns a structured **convergence judgment**: agreement score, whether debate has converged, and a summary of remaining disagreements. The debate stops early only when *both* convergence is declared *and* agreement clears a threshold (default 0.7). Otherwise it runs to maxRounds. At the end, the coordinator produces a single **final answer** with confidence, a machine-readable finalValue, and explicit **dissenting points** - so you know what still disagrees.

#### 4. Debate as gradient descent (optional steering)

We treat the debate loop loosely as **gradient descent toward consensus**:

- Parameters θ: Each debater’s position, embedded as a vector
- Gradient g: Drift of the group centroid between rounds
- Learning rate
- Temperature anneal (1.0 → 0.0)

Optimizer

A *coordinator strategy* that emits per-debater steering directives. Per-round position spread (how far each position sits from the centroid). Pluggable strategies — SGD (no steering, default), Momentum, RMSprop, Adam, Adagrad — inject a --- Coordinator Steering --- block into the next round's prompt. On GPQA with ~2 rounds average, simple SGD and Momentum matched or beat heavier optimizers; steering is a second-order knob once debate itself is working. Use maxRounds ≥ 4 if you want embedding-based strategies to activate (they need two snapshots, so they first steer in round 3).

#### 5. Memory — debates that learn from past debates

Completed debates are embedded (transformers.js, Xenova/all-MiniLM-L6-v2) and stored in a local RAG index per *scope*. Similar past debates are retrieved and injected as context - so your meeting can build on prior conclusions in the same domain.

#### 6. final results

Everything returns a DebateResult: final answer, rounds run, convergence flag, loss curve, token usage, and each debater's round-1 solo position - so you can compute **debate lift** on your own tasks the same way we did on GPQA.

### When to use it (and what it costs)

Debate is deliberately expensive: several models × several rounds. On our GPQA run, a single question consumed roughly **0.6M-1.2M tokens** — still cents with small models, but not free. Reach for it when:

- The question is hard and high-value (design, science, ambiguous trade-offs).
- You want an auditable trail of argument, not a black-box completion.
- Round-1 disagreement is likely — different models, different personas, different tools.

Skip it for tasks a single fast call already nails, or when a model cannot profit from other models.

**Tools matter on quantitative work.** Our reference GPQA run let debaters execute code; that likely helped on numeric questions. The default roster is read-only — enable bash/code in your debater config when the task demands it.

### Try it

Add the plugin in your opencode.json

    {
      "$schema": "https://opencode.ai/config.json",
      "plugin": ["discourse-descent"],
      "command": {
        "debate": {
          "template": "Use the `debate` tool to run a consensus debate on: $ARGUMENTS",
          "description": "Run a multi-agent consensus debate"
        }
      }
    }

Repo, configs, can be found at: [github.com/bolirev/ai-sdd](https://github.com/bolirev/ai-sdd)
