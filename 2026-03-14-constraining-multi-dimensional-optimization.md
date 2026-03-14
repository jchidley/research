# Constraining Multi-Dimensional Problems for Autonomous Optimization

**Date:** 2026-03-14
**Prior note:** [2026-03-13-karpathy-autoresearch.md](2026-03-13-karpathy-autoresearch.md)
**Session:** `~/.pi/agent/sessions/--home-jack-github-pi-autoresearch--/2026-03-14T10-34-57-777Z_238df8f2-563f-42e4-90b5-19e674c98bf6.jsonl`
**Working directory:** `~/github/pi-autoresearch`
**Repos examined:** [karpathy/autoresearch](https://github.com/karpathy/autoresearch), [davebcn87/pi-autoresearch](https://github.com/davebcn87/pi-autoresearch)

---

## The question

Karpathy's autoresearch works remarkably well for ML training. davebcn87's pi-autoresearch generalizes the pattern to arbitrary optimization targets (test speed, bundle size, build times, etc.). Why does the original work so well, and can the general version work equally well?

## Why the original works

Karpathy's setup has a property that almost no other domain has: **the single metric genuinely captures everything important about the task.**

val_bpb (validation bits per byte) isn't a proxy for model quality — at this scale, it *is* model quality. And the fixed 5-minute time budget eliminates the second axis (compute cost) by holding it constant. What remains is a truly one-dimensional optimization problem: use these 5 minutes more effectively.

Five properties make this possible:

1. **The metric is the thing, not a proxy for the thing.** val_bpb doesn't represent quality — it is quality.
2. **Fixed time budget collapses 2D → 1D.** Every experiment uses the same 5 minutes. No speed-vs-quality tradeoff.
3. **The evaluation is tamper-proof.** `prepare.py` is read-only. The agent cannot change how it's measured.
4. **Smooth gradients.** Slightly better hyperparameters → slightly lower loss. The agent can hill-climb incrementally.
5. **Unbounded search space within a bounded scope.** One file to edit, but everything in it is fair game: architecture, optimizer, hyperparameters, batch size.

## Why generalizing breaks

pi-autoresearch tries to apply this pattern to coding tasks. The tooling is more sophisticated (live dashboard, auto-commit, backpressure checks, session persistence), but the domains don't cooperate.

Almost every real endeavor is a multi-dimensional problem. Reducing it to a single dimension is a spherical-cow assumption — useful for toy problems, misleading for real ones.

**Test speed** — you measure seconds. The agent discovers it can skip coverage, mock the database, remove the slowest integration test. Seconds drop 40%. Tests still pass. But the test suite is weaker and you've lost coverage. The metric improved; the actual thing got worse.

**Bundle size** — the agent replaces a maintained date library with 50 lines of hand-rolled math. Bundle shrinks. Tests pass. Six months later you hit a timezone edge case.

**API latency** — you pick p95. The agent adds aggressive caching. p95 drops beautifully. But cache invalidation is subtly wrong and 0.1% of users see stale data.

In every case: metric improved, gates passed, result is worse. This is Goodhart's Law — the measure ceases to be a good measure once it becomes a target. It applies whenever the metric is a **proxy** rather than **the thing itself**.

## The insight: constraint architecture

The problem isn't that multi-dimensional problems can't be optimized. It's that collapsing them into one number loses information, and an autonomous agent optimizes for the number, not for what you actually want.

The fix is **not** to find a better single metric. It's to decompose the problem so each piece genuinely is one-dimensional.

### Step 1: Fix all axes but one

For any multi-dimensional problem, pick ONE axis to optimize and **gate** all others as hard constraints:

| Problem | Primary axis | Gated constraints |
|---------|-------------|-------------------|
| "Faster API" | p95 latency ↓ | p99 < 500ms, error rate < 0.1%, response body matches golden file |
| "Smaller bundle" | KB ↓ | All tests pass, Lighthouse perf > 90, no new runtime deps |
| "Better compression" | ratio ↑ | Decompression time < 50ms, output byte-identical |

This mirrors what Karpathy did with the time budget: he didn't solve a 2D problem (quality vs compute). He fixed one dimension (compute = 5 minutes) and optimized the other (quality = val_bpb).

### Step 2: Lock the evaluation

The agent must not be able to edit the measurement. Karpathy puts eval in `prepare.py` (read-only). The general version needs the same: a locked evaluation script that the agent cannot touch.

pi-autoresearch puts the benchmark in `autoresearch.sh` — a file the agent is encouraged to edit. This allows unconscious gaming. The fix: split into `autoresearch.eval.sh` (locked, ground-truth measurement) and `autoresearch.sh` (agent-editable wrapper for pre-checks only).

### Step 3: Gate before starting

Not every problem can be constrained this way. A rigorous scoping phase should apply three gate questions:

1. **Is there a command that outputs a single number?** No → not an optimization problem.
2. **Can the evaluation be locked?** No → metric is corruptible, results are meaningless.
3. **Can incremental source changes move the number?** No → it's a decision, not a loop.

Reject and explain when any gate fails. The most valuable part of the scoping skill isn't the harness template — it's the rejection table that says "this isn't an autoresearch problem, here's why."

## Where single-axis optimization hits its ceiling

Even with proper gating and locked evals, single-axis autonomous optimization has limits:

- **Most coding tasks have step-function improvements.** You either find the optimization or you don't. No smooth gradient to climb. The agent wastes many runs with no signal.
- **Agents plateau fast.** After 10–20 experiments on "make tests faster," the agent has tried parallelization, removed slow setup, and is out of ideas. ML training has a much deeper design space.
- **The gates can't catch everything.** Tests pass ≠ nothing broke. The agent can make the test suite weaker, game coverage, introduce subtle correctness issues that no automated check catches.

This is inherent to the domain, not fixable by better tooling.

## Multi-agent decomposition: the next step

The ceiling of single-axis optimization points toward a different architecture. Instead of one agent collapsing a multi-dimensional problem into one number, multiple agents each explore one dimension while holding everything else constant, then a coordinator (or human) synthesizes.

```
User: "Improve this API — slow, big bundle, poor error handling"
                    │
                    ▼
             ┌─────────────┐
             │ Coordinator  │  decomposes into single-axis problems
             └──────┬───────┘
               ┌────┼────┐
               ▼    ▼    ▼
             ┌──┐ ┌──┐ ┌──┐
             │A │ │B │ │C │   each: one axis, others gated
             │p95│ │KB│ │err│  explores, produces ranked proposals
             └─┬┘ └─┬┘ └─┬┘
               └────┼────┘
                    ▼
             ┌──────────────┐
             │  Coordinator  │  detects conflicts, presents
             │  + Human      │  tradeoff surface, human decides
             └──────────────┘
```

Key differences from autoresearch:

- **Agents are scouts, not decision-makers.** They explore and report; they don't auto-commit.
- **Output is proposals, not a branch.** Each agent produces a ranked list: what changed, what the metric moved, what the diff looks like.
- **The human makes the multi-dimensional decision.** The coordinator presents the tradeoff surface (Agent A's proposal 1 + Agent B's proposal 4 + Agent C's proposal 7 are compatible and improve all three axes). The human picks.
- **Conflict detection.** Proposals from different agents may touch the same files. The coordinator flags this.

This solves Goodhart's Law because no single agent optimizes the final outcome. Each agent's metric genuinely is the thing along its axis (latency is latency when correctness is gated). The multi-dimensional judgment stays with the human.

## Working artifact

Rewrote the `autoresearch-create` skill at `~/github/pi-autoresearch/skills/autoresearch-create/SKILL.md` as a working draft. Key changes from the original:

- **Two explicit phases:** scope (slow, careful) → loop (fast, autonomous)
- **Mandatory gate:** three yes/no questions; reject with explanation if any fails
- **Sentence test:** agent must complete "We are optimizing [workload] to achieve the lowest/highest [metric], measured by [command]" before proceeding
- **Locked eval:** new `autoresearch.eval.sh` (read-only) separate from `autoresearch.sh` (agent-editable pre-checks only)
- **Rejection table:** concrete examples of tasks that fail the gate with suggested alternatives
- **Search Landscape section** in `autoresearch.md` template: forces domain knowledge up front

The extension code needed zero changes. The fix is entirely in the skill — the domain knowledge layer — which mirrors Karpathy's design (prepare.py = infrastructure, program.md = domain knowledge).

## Open threads

- **Multi-agent coordinator** — the decompose → dispatch → synthesize → present architecture. Not built. Would be a separate skill/extension, not a modification of autoresearch. The autoresearch loop tools could be reused inside each scout agent.
- **Domain-specific skill variants** — `autoresearch-testspeed`, `autoresearch-bundlesize`, `autoresearch-algorithm` etc. Each encodes search landscape knowledge for its domain. Better than one generic skill trying to handle everything.
- **Baseline regression check** — after each "keep," silently re-run the original unmodified benchmark to verify the improvement isn't an artifact of benchmark drift. Would catch subtle eval gaming even with a locked eval script.
