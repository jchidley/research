# Karpathy's Autoresearch: The Autonomous Experiment Loop

**Date:** 2026-03-13
**Source session:** pi session, 2026-03-13, working directory `C:\Users\jackc\AppData\Local\Temp`

---

## What It Is

Autoresearch is a 630-line open-source Python script released by Andrej Karpathy on ~March 8, 2026. It implements an autonomous experiment loop: an AI agent modifies ML training code, runs a short experiment, evaluates the result against a single metric, keeps improvements, discards failures, and repeats — indefinitely, without human involvement.

The repo is built around three files: `prepare.py` (locked-down data prep/eval), `train.py` (the single file the agent may edit), and `program.md` (a Markdown instruction file written by the human that defines goals, constraints, and loop rules). The human's role shifts from writing code to designing the research environment.

**Repo:** https://github.com/karpathy/autoresearch

## The Core Pattern

Five ingredients make this work:

1. **A trainable artifact with a bounded search space.** The agent edits one file only (`train.py`). Everything else — data prep, evaluation — is locked down. This keeps the search space narrow enough for an LLM agent to operate reliably.

2. **A clear, single scalar metric.** Karpathy uses `val_bpb` (validation bits per byte, lower is better). The metric must be unambiguous (binary keep/discard, no human judgment), honest (validated against held-out data so the loop can't cheat), and comparable across runs.

3. **Sufficient held-out data.** Training data broad enough to generalize, with a completely untouched validation set the agent never trains on. This is the overfitting guard — and the component most at risk as experiment counts grow.

4. **A fixed compute budget per experiment.** Every run gets a 5-minute wall-clock cap. This makes results apples-to-apples regardless of what the agent changes (architecture, hyperparams, optimizer). It yields ~12 experiments/hour, ~100 overnight. The framing is "best model for this platform under this budget," not "best model in the abstract."

5. **Git as memory.** Each experiment is a commit. The agent reads branch history to see what worked, what didn't, and what's been tried. This prevents redundant exploration and enables cumulative improvement.

## Documented Results

### Karpathy — 700 experiments, 2 days on nanochat

- Pointed autoresearch at nanochat, his already well-tuned GPT-2 training codebase
- Overnight: 126 experiments reduced loss from 0.9979 → 0.9697
- Over 2 days: ~700 experiments found ~20 additive improvements
- "Time to GPT-2" dropped from 2.02 → 1.80 hours (11% faster)
- Specific catches the agent found that Karpathy had missed:
  - QKNorm had no scaler multiplier → attention too diffuse
  - Value Embeddings had no regularization
  - Banded attention window too conservative
  - Suboptimal AdamW betas, weight decay schedule, and initialization
- All depth-12 improvements transferred cleanly to depth-24 models

### Tobi Lütke (Shopify CEO) — query expansion model

- Adapted autoresearch for his QMD open-source project
- Told an AI agent to read the autoresearch repo and build a version for QMD, went to sleep
- 37 experiments in 8 hours → a 0.8B model scored 19% higher than his previous 1.6B model
- A smaller model outperformed one twice its size
- Then pointed the same loop at a reranker and beat that baseline too

### Hyperspace AI (Varun Mathur) — distributed swarm, 35 agents

- Distributed the single-agent loop across a peer-to-peer network
- 35 agents ran 333 experiments unsupervised in one night
- Hardware diversity became a feature: H100s used brute force, CPU-only laptop agents found cleverer initialization strategies (Kaiming, Xavier init)
- Gossip-based discovery (GossipSub protocol): when one agent found Kaiming init dropped loss by 21%, 23 others incorporated it within hours
- In 17 hours, agents independently rediscovered ML milestones (RMSNorm, tied embeddings) that took human researchers ~8 years to formalize

### "Witcheer" — Mac Mini M4 overnight

- Consumer hardware run: 26 of 35 experiments failed or crashed, but 7 succeeded
- Key insight: "the model got better by getting simpler" — reached without human intervention
- Demonstrates the high failure rate on non-H100 hardware (74% crash rate)

## Relationship to the Ralph Wiggum Loop

Autoresearch is a domain-specific instance of the **Ralph Wiggum Loop** — an informal pattern (coined by Geoffrey Huntley, popularised mid-2025) for running an AI agent in an iterative loop that repeatedly attempts a task, checks against a concrete criterion, and feeds failure back into the next attempt.

The canonical Ralph loop is: `while :; do cat PROMPT.md | claude-code ; done` — progress lives in files and git, not the LLM's context window.

**Shared structure:**

| Element | Ralph Wiggum Loop | Autoresearch |
|---------|-------------------|--------------|
| Action | Agent edits code | Agent edits `train.py` |
| Feedback | Test output / error logs | `val_bpb` after 5-min training run |
| Memory | Git history / files | Git commits per experiment |
| Stop condition | Tests pass / safety limit | Loss improves → keep; else discard |
| Human role | Write spec + constraints | Write `program.md` |

**Key difference:** The Ralph loop is general-purpose (coding, refactoring, builds). Autoresearch specialises it for ML research by adding a fixed compute budget and a scalar metric, which makes the feedback signal much cleaner than "did the tests pass?" This tighter feedback loop is why autoresearch achieves hundreds of productive iterations where a general Ralph loop might stall or oscillate.

**Shared failure modes** (from the Ralph loop literature, directly applicable):

- **Metric gaming / reward hacking:** If success is defined narrowly, the agent optimises for the proxy, not the intent. In autoresearch: the agent could find tricks that lower `val_bpb` on the specific validation set without genuinely improving the model.
- **Validation set spoilage:** With enough experiments, the agent effectively hill-climbs against validation data. This is the autoresearch-specific variant of metric gaming — the community flagged it immediately (GitHub Discussion #43).
- **Context overload / mode collapse:** The agent converges to repetitive, narrow behaviours. Autoresearch partially mitigates this by resetting context each run (git as memory, not LLM context), but the risk remains in the `program.md` framing.
- **Oscillation:** Fix A breaks B, fix B reintroduces A. Autoresearch's binary keep/discard with git rollback handles this better than a naive Ralph loop.

## Gamification Risks

The autoresearch pattern has inherent gamification problems that compound with scale:

- **Validation set overfitting at scale.** 700 experiments against one val set is already pushing it. The more experiments, the more the agent implicitly hill-climbs on validation data quirks. The eval becomes the bottleneck — as Philipp Schmid noted, "the loop hinges on your eval."
- **Metric narrowness.** `val_bpb` is a single number. Real model quality is multi-dimensional (coherence, safety, instruction-following, etc.). Optimising a single scalar can silently degrade unmeasured dimensions.
- **Transfer fragility.** Karpathy's depth-12 → depth-24 transfer worked. But there's no guarantee — improvements found at small scale under a 5-minute budget may not generalise to production training.
- **Results not comparable across machines.** Budget is time-based, not FLOP-based. An agent on an H100 explores a fundamentally different search space than one on an RTX 3080.

## Community Forks and Extensions

- RTX 3080 fork for consumer GPUs
- Apple Silicon / Mac support PRs
- Jetson / SDPA fallbacks
- Multi-agent variants: one agent hypothesises, another runs, a third evaluates
- Lobehub skill for easy setup
- Karpathy hints at meta-research: using autoresearch to optimise `program.md` itself

## Sources

- VentureBeat: [Andrej Karpathy's new open source 'autoresearch'...](https://venturebeat.com/technology/andrej-karpathys-new-open-source-autoresearch-lets-you-run-hundreds-of-ai) (2026-03-10)
- Philipp Schmid: [How Autoresearch will change Small Language Models adoption](https://www.philschmid.de/autoresearch) (2026-03-09)
- The Neuron: [Karpathy's autoresearch Lets AI Run Experiments Overnight](https://www.theneuron.ai/explainer-articles/andrej-karpathys-autoresearch-tiny-repo-big-implications/) (2026-03-09)
- beuke.org: [Ralph Wiggum Loop](https://beuke.org/ralph-wiggum-loop/)
- GitHub: [karpathy/autoresearch](https://github.com/karpathy/autoresearch)
- GitHub Discussion #43: [Session report: 0.9979 → 0.9697 in 126 experiments](https://github.com/karpathy/autoresearch/discussions/43)
