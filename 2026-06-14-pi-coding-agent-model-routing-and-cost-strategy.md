# pi Coding-Agent Model Routing and Cost Strategy

**Updated: June 14, 2026**

> **Document role:** current operational strategy for pi model routing, cost control, provider choice, and Codex reasoning effort. For the separate policy/entitlement question — what subscription credentials may safely be used through which tools — see [`2026-04-16-subscription-tooling-policies-anthropic-openai-google.md`](2026-04-16-subscription-tooling-policies-anthropic-openai-google.md).
>
> **Privacy note:** this memo contains personal spend and usage data. GitHub visibility for `jchidley/research` was checked on 2026-06-14 and is currently **PUBLIC**, so do not push/publish this memo unless the repo is made private or the personal figures are redacted.

This document is a decision guide for a specific user profile:

- UK hobbyist / power user
- primary workflow is **pi coding agent**
- heavy coding-agent usage
- recent API spend: **$2,272.82 in 30 days**
- split roughly between GPT and Claude APIs
- usage pattern is heavily cache-read dominated
- goal: decide when subscriptions make sense, when direct API wins, and which Chinese models should be used as cheaper alternatives

The core question is:

> Given this usage pattern, what is the cheapest workable stack that still preserves coding quality?

The answer is not one model or one subscription. It is a routing strategy: which model/provider/subscription to use for each class of coding-agent work, and at what reasoning effort.

## 0. Executive decision

Current working strategy:

| Role | Default choice | Why |
|---|---|---|
| Daily interactive pi work | **OpenAI Codex / GPT-5.5, medium reasoning** | Existing subscription, strong workflow, current experiment to test quality gain over low |
| Routine fallback if medium hits limits | **OpenAI Codex / GPT-5.5, low reasoning** | Local logs showed low was materially better than minimal without obvious token/cost explosion |
| Mechanical/simple work | **GPT-5.5 minimal** or **DeepSeek V4 Flash** | Fast, cheap, sufficient for extraction/formatting/simple edits |
| Bulk API implementation / long repo sweeps | **DeepSeek V4 Pro** | Best cost profile for cache-heavy pi-style workloads |
| Cheap background / first pass | **DeepSeek V4 Flash** | Very low cost, good enough for simple exploration and retries |
| Premium review / hard judgement | **OpenAI medium/high or Claude** | Use when correctness, architecture, or subtle intent matters more than cost |
| Experimental comparison lanes | **MiniMax, Kimi, Qwen, GLM** | Useful for second opinions and provider diversification, not primary defaults yet |

Immediate operating rule:

> Use OpenAI Codex GPT-5.5 medium while the subscription remains usable. If quota pressure appears, drop routine work to low and move bulk implementation/retry loops to DeepSeek V4 Pro/Flash. Keep Copilot on the watchlist for policy-clean agent-platform integration, and test GLM 5.2 only as an experimental Chinese-model lane until quality/reliability evidence accumulates.

The main measurement still needed is whether **medium** improves completed-task quality enough to justify any extra subscription quota pressure compared with **low**.

### 0.1 Direction of travel: Codex, Copilot, Chinese models

Current web-search and local usage analysis suggest three different market directions:

| Lane | Direction of travel | Practical meaning for this user |
|---|---|---|
| **OpenAI Codex** | First-party coding product with increasingly explicit quota/rate-card/credit controls | Keep using as the interactive premium lane while quota works; expect metering to get clearer, not looser |
| **GitHub Copilot** | GitHub-native agent platform: CLI, cloud agents, ACP, SDKs, third-party agents | Watch closely as the cleanest policy route for custom front ends and GitHub-integrated agent workflows |
| **Chinese/open models** | Cheap agent infrastructure: long context, coding-first models, OpenAI/Anthropic-compatible APIs, coding plans, some open weights | Use for bulk implementation, long-context sweeps, retries, and comparison lanes |

Codex appears durable but not unlimited. Recent web-search points toward token/rate-card-managed capacity, model-dependent message/task limits, paid/credit overflow, and banked rate-limit resets. That is a product OpenAI intends to grow, but also one it intends to meter.

Copilot is morphing differently: less like a single model subscription and more like an agent layer inside GitHub's development platform. ACP support, Copilot CLI, third-party coding agents, and Copilot SDKs make it strategically important for `pi`-style orchestration if GitHub keeps those integration surfaces open.

Chinese models are moving from "cheap alternatives" toward a real infrastructure tier. DeepSeek remains the economic baseline; MiniMax is a cheap coding-worker comparison lane; Kimi is a stronger long-context fallback; Qwen is the open/Alibaba ecosystem lane; GLM is becoming more interesting with the GLM 5.2 coding-plan rollout.

GLM 5.2 specific note: current search says it is live in Z.ai's Coding Plan with **1M context**, coding-first positioning, and high/max thinking modes, with max recommended for coding; reports mention API and MIT weights following shortly, but also note limited/no solid benchmark evidence at launch. Treat GLM 5.2 as worth testing, not yet a primary default.

## 1. Update log: May 23 / June 14 data refresh

The main conclusions are unchanged. Newer pricing/docs add these updates:

- **DeepSeek V4 Pro** is no longer just a temporary-discount story. DeepSeek API docs now say V4 Pro pricing will be adjusted to **1/4 of the original price after the 75% promotion ends on 2026-05-31 15:59 UTC**. Break-even calculations below therefore use the new durable 1/4-price benchmark.
- **OpenAI Codex Pro $100** still has a temporary **2× Codex usage promo through 2026-05-31**. Standard $100 Pro is 5× Plus; promo is 10× Plus. **Pro $200** continues to be described as **20× Plus**. User reports still flag opaque/fast quota drain, so treat Codex subscriptions as useful first-party capacity rather than unlimited compute.
- **Anthropic** has split subscription use more clearly. Normal interactive Claude / Claude Code remains subscription-oriented, while **Agent SDK**, `claude -p`, Claude Code GitHub Actions, and third-party Agent SDK apps move toward a separate monthly Agent SDK credit pool / usage-credit model from **2026-06-15**. This ends much of the old “Claude subscription as cheap programmatic backend” arbitrage.
- **Google Gemini 3.1 Pro** API pricing remains materially cheaper than OpenAI/Anthropic frontier API, but much more expensive than DeepSeek V4 Pro durable 1/4-price for this cache-heavy workload. Gemini 3.5 Flash has appeared as a newer cheaper Google option, but it is not a direct frontier/API-equivalent comparison to GPT-5.5 / Opus / DeepSeek V4 Pro.
- **MiniMax M2.7** pricing is still roughly **$0.28–$0.30 / 1M input** and **$1.20 / 1M output**, depending on provider/route.
- **Kimi K2.6** remains a premium Chinese option, commonly around **$0.75–$0.95 / 1M input** and **$3.50–$4.65 / 1M output** depending on route.
- **Alibaba Coding Plan**, **Z.ai GLM Coding Plan**, **MiniMax Token Plan**, and **OpenCode Go** remain quota/subscription bundles rather than unlimited access.
- **OpenAI GPT-5.5 / Codex reasoning effort** should be treated as a quality/limit trade-off, not just a latency setting. OpenAI documentation says GPT-5.5 now defaults to **medium** reasoning as the balanced starting point. Minimal reasoning is intended for deterministic/lightweight tasks where speed matters. Community experiments on real coding tasks report that increasing effort changes the *kind* of patch produced: low→medium tends to reduce heuristic/partial implementations and increase repo/domain modelling. Local pi transcript review showed that this user's workload clearly benefits from **low** over **minimal**; after observing little practical token/cost penalty from low, pi has now been set to **medium** as an experiment.

### 1.1 Current user/community read

Public discussion around Anthropic's June Agent SDK credit split is broadly negative among heavy automation users. The recurring interpretation is that Anthropic is ending subscription arbitrage: Claude remains high-quality, but subscription-backed programmatic agent work is no longer safe to treat as cheap bulk compute.

Observed migration pattern:

- keep **Claude** for interactive Claude Code, hard debugging, architecture, and final review
- try **OpenAI Codex / ChatGPT Pro** as the strongest first-party subscription alternative, while watching quota drain carefully
- move bulk pi-style agent work toward **DeepSeek V4 Pro / Flash**, especially now that the 1/4-price V4 Pro level appears durable
- use **MiniMax, Qwen, Kimi, GLM, OpenCode Go** as comparison/fallback lanes rather than assuming any one bundle is unlimited

There is little public discussion specifically about **pi-coding-agent** users, but pi's bring-your-own-key, model-router design fits the emerging pattern: cheap API lane for implementation, premium subscription/API lane for review and hard cases.

### 1.2 June 14 reasoning-effort refresh

A review of exported ChatGPT/pi transcripts in `~/.pi/agent/session-transcripts/all_2026-05-23T12-01-08-705Z` found that this user's ChatGPT/Codex work is heavily weighted toward tasks where `minimal` reasoning is likely to leave quality on the table:

| Task class | Approx sessions matched | Why higher effort helps |
|---|---:|---|
| Review / audit / meta-analysis | 377 | distinguish strategic vs tactical quality; avoid generic praise; find missed seams |
| Data / model analysis | 325 | separate signal from noise; question assumptions; avoid false causal claims |
| Planning / architecture | 278 | preserve invariants; sequence work; identify stale plans and docs |
| Debug / diagnosis | 274 | diagnose root cause rather than local symptoms |
| Ops / config / deployment | 160 | verify code/config/binary/service/runtime all agree |
| Maths / optimisation | 60 | formulate the objective before solving |

The most informative local comparison was the wood-cutting optimisation sessions. `minimal` eventually converged, but only after repeated correction, inefficient search, and objective confusion. A `low` review/redo got to the real issue faster: the problem was not just bin packing, but choosing the right lexicographic objective — complete sets, preserve long boards, use small offcuts first, then minimise waste.

Policy update for this user:

- previous default ChatGPT/Codex reasoning in pi: **minimal**
- first update: **low**, because local sessions showed minimal caused extra steering, objective confusion, and plausible-but-local answers
- observed after switching to low: no obvious practical token/cost explosion; low calls averaged roughly **56K tokens / ~$0.06 API-equivalent per call** in the sampled period
- current experiment: **medium** is now the pi default for OpenAI/Codex, to test whether better repo/domain modelling and fewer partial implementations justify any quota cost
- use **minimal** only for simple summaries, mechanical edits, extraction, formatting, and cheap exploration
- use **low** if medium proves quota-heavy for routine coding/debugging
- keep **medium** for architecture, migration planning, final review of important diffs, telemetry/model conclusions, and "why did the previous session go wrong?" reviews

After the low switch, the OpenAI/Codex token/cost profile was still cache-heavy:

| Bucket | Tokens | Token share | API-equiv cost | Cost share |
|---|---:|---:|---:|---:|
| Input | 4.16M | 10.2% | $20.82 | 47.2% |
| Output | 165K | 0.4% | $4.96 | 11.2% |
| Cache read | 36.66M | 89.4% | $18.33 | 41.6% |
| Cache write | 0 | 0% | $0.00 | 0% |
| **Total** | **40.99M** | **100%** | **$44.11** | **100%** |

---

## 2. Actual 30-day usage baseline

Recent 30-day token usage:

| Bucket | Tokens | Cost | Share of spend |
|---|---:|---:|---:|
| Input | 139.1M | $345.74 | 15% |
| Output | 13.9M | $246.31 | 11% |
| Cache read | 4.0B | $1,418.48 | 62% |
| Cache write | 43.4M | $262.29 | 12% |
| **Total** | **4.2B** | **$2,272.82** | **100%** |

The key feature is the **4.0B cache-read tokens**.

For this user, cache-read pricing is not a footnote. It is the economic centre of the workflow.

---

## 3. Equivalent API cost by provider

Using the same usage shape, approximate equivalent monthly API costs are:

| Provider / model | Estimated cost for same usage | Relative to current spend |
|---|---:|---:|
| **DeepSeek V4 Pro durable 1/4-price** | **~$106** | ~21× cheaper |
| **Google Gemini 3.1 Pro**, if mostly ≤200K context | **~$1,332** | ~1.7× cheaper |
| **Google Gemini 3.1 Pro**, if >200K context pricing applies | **~$2,580** | slightly more expensive |
| **Anthropic Claude Opus 4.7** | **~$3,314** | ~1.5× more expensive |
| **OpenAI GPT-5.5** | **~$3,330** | ~1.5× more expensive |

DeepSeek V4 Pro durable 1/4-price economics are now the important benchmark because DeepSeek says the 75% discount becomes the adjusted price after the promotion window ends.

### 3.1 Why DeepSeek wins on this usage pattern

DeepSeek V4 Pro durable 1/4-price pricing:

| Bucket | Price / 1M tokens |
|---|---:|
| Cache-miss input | ~$0.435 |
| Cache-hit input | ~$0.003625 |
| Output | ~$0.87 |

OpenAI / Anthropic frontier API pricing is much higher, especially on output and cache reads.

For the 4.0B cache-read tokens alone:

| Model | Cache-read cost |
|---|---:|
| **DeepSeek V4 Pro durable 1/4-price** | **~$15** |
| OpenAI GPT-5.5 | ~$2,000 |
| Claude Opus 4.7 | ~$2,000 |
| Gemini 3.1 Pro ≤200K | ~$800 |
| Gemini 3.1 Pro >200K | ~$1,600 |

That single row explains most of the economic difference.

---

## 4. Subscription logic: when does a subscription make sense?

A subscription only makes sense if:

> included useful work > equivalent API spend at the same price

For this user, compare every subscription against **DeepSeek V4 Pro durable 1/4-price**, not against OpenAI/Anthropic API.

The DeepSeek V4 Pro equivalent for the current monthly workload is now about:

> **$106/month**

So a subscription is economically rational only if it covers enough useful work to beat that number, or if it delivers enough quality/workflow value to justify paying more.

### 4.1 Break-even against DeepSeek V4 Pro durable 1/4-price

| Subscription price | Must cover at least this share of current workload to beat DeepSeek V4 Pro |
|---:|---:|
| $20/month | ~19% |
| $50/month | ~47% |
| $90/month | ~85% |
| $100/month | ~94% |
| $200/month | ~189% |
| $300/month | ~283% |

Formula:

> break-even workload share = subscription price / $106

So:

- a **$100/month** subscription must cover almost **all** current usage before paid overflow to beat DeepSeek on raw cost
- a **$200/month** subscription cannot beat DeepSeek on raw bulk-compute cost for this workload; it must be justified by superior quality, workflow, or non-coding value
- above **$106/month**, DeepSeek V4 Pro is cheaper for bulk work unless the subscription gives materially better quality or workflow

### 4.2 The real catch: API spillover

The subscription is cheap only while it stays inside included usage.

Once it spills into:

- OpenAI credits
- Anthropic usage credits / Extra Usage
- API-key mode
- premium request overages
- token-priced add-ons

then it must be compared directly against Chinese-model API pricing.

At that point, DeepSeek/MiniMax/Qwen/Kimi often win on cost.

---

## 5. Available subscription options

### 5.1 OpenAI ChatGPT / Codex

### What it offers

OpenAI currently has the strongest subscription case for this user because Codex is meant to be used across:

- web
- CLI
- IDE extension
- Codex cloud tasks
- ChatGPT account-backed access

Current relevant tiers include:

| Tier | Approx role |
|---|---|
| Plus | light daily use |
| Pro $100 | middle power-user tier; standard 5× Plus Codex allowance, temporarily 10× through 2026-05-31 |
| Pro $200 | highest individual tier, roughly 20× Plus Codex allowance |
| Business / Enterprise | team and governance route, no training on business data by default |

### Front-end requirement

To use subscription value, the practical front ends are:

- Codex CLI
- Codex IDE extension
- Codex web
- ChatGPT/Codex first-party surfaces

For pi, this means OpenAI subscription value is useful only if pi can either:

1. invoke a supported Codex path, or
2. coexist with Codex as a separate first-party agent, or
3. use OpenAI API separately when subscription limits are exhausted

### Economic view

The $100 or $200 plan is attractive if it absorbs a meaningful share of your real coding work.

Against DeepSeek V4 Pro durable 1/4-price:

- $100 plan must cover >94% of current monthly workload to win on raw cost
- $200 plan cannot win on raw bulk-compute cost for this workload; it must win on quality/workflow/non-coding value

If Codex limits force frequent credit/API purchases, route routine work to DeepSeek/MiniMax instead. Current user reports also mention opaque or unexpectedly fast Codex quota drain, so measure actual useful work per billing period rather than assuming advertised multipliers translate directly into workload coverage.

### Best use

Use OpenAI subscription for:

- hard tasks
- final review
- architecture
- tasks where Codex workflow succeeds with fewer retries
- non-coding ChatGPT work that has independent value

### Reasoning-effort policy for OpenAI/Codex

OpenAI documentation describes reasoning effort as a tuning knob and says GPT-5.5 defaults to **medium** as the balanced setting for quality, reliability, latency, and cost. Minimal reasoning is explicitly for lightweight deterministic work such as extraction, formatting, short rewrites, and simple classification.

For this user's pi workflow, local transcript review first supported **low** over **minimal**. The work is often diagnostic, architectural, operational, or data/model-heavy. In those cases, the quality risk of `minimal` is not obvious nonsense; it is plausible-but-local answers that miss the true objective, root cause, or verification step.

After switching from minimal to low, observed pi logs did not show a large practical token/cost increase. Low was therefore a clear improvement. The current configuration now tests **medium** as the OpenAI/Codex default, because OpenAI itself treats medium as the balanced GPT-5.5 starting point and community coding experiments report better repo/domain modelling above low.

Recommended OpenAI/Codex effort hierarchy:

| Task | Effort |
|---|---|
| extraction, formatting, short docs, simple summaries | minimal |
| routine coding, normal debugging, telemetry interpretation, ops/config | medium while experimenting; low if quota becomes tight |
| architecture, migration plans, important diff review, failure analysis | medium |
| rare high-stakes or repeatedly failing tasks | high / xhigh |

Because subscription limits are opaque, measure useful completed work per quota window rather than assuming effort levels map linearly to cost. For this user, `minimal` as a global default appears too weak for the actual task mix; `low` was justified by both local transcript review and observed usage. The next experiment is whether `medium` improves quality enough to justify any additional quota pressure.

Do not use OpenAI as the default for all bulk API work if DeepSeek quality is adequate, but when using the OpenAI subscription, prefer `medium` during the current experiment, falling back to `low` if quota becomes tight.

---

### 5.2 Anthropic Claude / Claude Code

### What it offers

Relevant individual tiers:

| Tier | Role |
|---|---|
| Pro | light/frequent Claude use |
| Max 5x | heavier individual use |
| Max 20x | daily power-user use |
| Usage credits / Extra Usage | API-priced overflow after included usage |

Claude Code is included with Pro and Max, but usage is shared across Claude and Claude Code.

### Front-end requirement

To use subscription value, use:

- Claude Code
- Claude web/app
- first-party Claude surfaces

Third-party harness reuse of subscription-backed access is fragile. Anthropic has pushed third-party agent usage toward usage credits / Agent SDK credits / API-style billing.

### Agent SDK / programmatic-use change

From **2026-06-15**, Anthropic's current documentation and reporting indicate that **Agent SDK**, `claude -p`, Claude Code GitHub Actions, and third-party Agent SDK apps draw from a separate monthly Agent SDK credit pool rather than the ordinary interactive subscription bucket. Public reaction is that this largely ends the old "Claude subscription as cheap programmatic backend" arbitrage.

Interactive Claude Code and Claude web/app remain valuable, but automated pi-style or third-party programmatic use should be assumed to need API/credit economics unless proven otherwise.

### Economic view

Claude quality remains very strong, but API, usage-credit overflow, and Agent SDK credits are expensive compared with DeepSeek.

Against DeepSeek V4 Pro durable 1/4-price:

- $100 Max 5x must cover >94% of current monthly workload to win on raw cost
- $200 Max 20x cannot win on raw bulk-compute cost for this workload; it must win on quality/workflow/non-coding value

The moment Claude work becomes usage-credit/API-priced, it becomes a premium escalation lane, not the default lane.

### Best use

Use Claude subscription for:

- high-confidence code review
- architecture
- complex debugging
- tasks where Claude Code’s workflow is materially better
- final pass over DeepSeek/MiniMax implementations

Do not rely on Claude subscription as a generic cheap backend for pi. Use it mainly for first-party interactive Claude Code, hard debugging, architecture, and review; assume programmatic/third-party agent usage needs separate credit/API accounting.

---

### 5.3 Google Gemini / Gemini CLI / Code Assist

### What it offers

Google offers Gemini CLI / Code Assist with Google account or paid subscriptions such as AI Pro / Ultra.

### Front-end requirement

Use:

- Gemini CLI
- Gemini Code Assist IDE integration
- official Google paths
- API key / Vertex for direct automation

Google explicitly disallows third-party OAuth piggybacking on Gemini CLI-backed services.

### Economic view

Gemini API can be cheaper than OpenAI/Anthropic, but is still much more expensive than DeepSeek V4 Pro durable 1/4-price for this cache-heavy workload.

Estimated equivalent for this usage:

- ~$1,332 if mostly below the cheaper context tier
- ~$2,580 if large-context pricing applies

### Best use

Use Gemini as:

- official Google first-party tool
- backup model
- multimodal/long-context comparison lane
- API only when model quality justifies the extra cost over DeepSeek

---

### 5.4 GitHub Copilot

### What it offers

Copilot is a product subscription, not a general API backend.

It offers:

- IDE integration
- Copilot Chat
- Copilot CLI
- cloud agents
- partner agents such as Claude and Codex in GitHub/VS Code contexts
- premium request accounting
- token/session/weekly limits

### Front-end requirement

Use:

- VS Code / supported IDEs
- GitHub Copilot CLI
- GitHub cloud agent
- GitHub Agent HQ / partner agents

### Economic view

Copilot is not directly comparable to DeepSeek API because it is a product layer.

It makes sense if:

- IDE integration saves time
- included models handle routine tasks
- premium limits are sufficient
- GitHub workflow integration matters

It does not replace a pi API-routing strategy.

---

### 5.5 Cursor / Windsurf

These are full coding products, not backend subscriptions for pi.

They may be useful if the product experience is worth it, but they should be evaluated separately from pi’s model-routing economics.

Current market direction:

- included usage
- quota/budget limits
- on-demand or API-list-price overflow

For pi-first work, they are not primary backends.

---

## 6. Chinese model alternatives

### 6.1 DeepSeek V4 Pro / V4 Flash

### Access model

- free web/app
- paid API
- OpenAI-compatible API
- Anthropic-compatible API
- no obvious ChatGPT/Claude-style subscription emphasis

### Current models

| Model | Role |
|---|---|
| DeepSeek V4 Pro | serious coding/reasoning/agent lane |
| DeepSeek V4 Flash | cheap fast worker lane |

### Why it matters

DeepSeek is the default economic benchmark.

At list price, for this user’s 30-day workload:

> DeepSeek V4 Pro ≈ **$106/month**

That is the price other subscriptions and APIs must beat for bulk work.

As of the May 23 refresh, DeepSeek API docs say V4 Pro pricing will be adjusted to **1/4 of the original price** after the 75% promotion ends on **2026-05-31 15:59 UTC**. This document therefore treats the 1/4-price level as the durable planning number.

### Best use in pi

- default API model for serious coding
- long-context repo analysis
- large-volume agent work
- cheap implementation passes
- cheap retry loops

Use Flash for:

- simple tasks
- background work
- first-pass exploration
- low-risk refactors
- test generation

---

### 6.2 MiniMax M2.7

### Access model

- API
- Anthropic-compatible coding-tool integration
- Token Plan subscription

MiniMax Token Plan examples:

| Plan | Price | M2.7 limit |
|---|---:|---:|
| Starter | $10/month | 1,500 requests / 5h |
| Plus | $20/month | 4,500 requests / 5h |
| Max | $50/month | 15,000 requests / 5h |

### API economics

MiniMax M2.7 API pricing is roughly:

- $0.28–$0.30 / 1M input
- $1.20 / 1M output

This can be cheaper than DeepSeek V4 Pro on normal input/output depending on route, though DeepSeek’s cache-hit economics can dominate for cache-heavy workflows.

### Best use in pi

- everyday coding alternative
- cheap Claude-like coding lane
- second implementation attempt
- comparison model against DeepSeek

MiniMax is one of the most important models to test next.

---

### 6.3 Kimi K2.6

### Access model

- Kimi web/app
- Kimi Code
- Kimi membership credits
- API
- third-party routes such as OpenRouter / OpenCode Go

Kimi membership now uses a unified credit pool across:

- Kimi Chat
- Kimi Code
- Kimi Claw
- agent features
- document/PPT/spreadsheet tools
- image generation

### API economics

Kimi K2.6 is not the cheapest Chinese option.

Approx API pricing:

- input: ~$0.75–$0.95 / 1M
- output: ~$3.50–$4.65 / 1M

### Best use in pi

- long-context codebase reasoning
- hard agentic coding fallback
- when DeepSeek gets stuck
- front-end/full-stack generation experiments
- second opinion on complex repo changes

Kimi is a premium Chinese fallback, not the default cheap worker.

---

### 6.4 Qwen3.6 / Qwen Code / Alibaba Model Studio

### Access model

- Alibaba Model Studio API
- Qwen Code
- Qwen Agent
- open weights for some models
- Coding Plan subscription

Alibaba Coding Plan currently offers a Pro plan:

| Plan | Price | Quota |
|---|---:|---:|
| Pro | $50/month | 6,000 requests / 5h; 45,000 / week; 90,000 / month |

The old Lite plan no longer accepts new subscriptions.

Important restriction:

> Coding Plan is for coding tools, not arbitrary API calls, scripts, custom backends, or batch automation.

### Best use in pi

- Qwen Code experiments
- open-weight/local experiments
- alternative coding lane
- model ecosystem exploration
- coding-plan test if fixed monthly quota is attractive

Qwen’s model/provider choice matters a lot. Do not treat all Qwen variants as equivalent.

---

### 6.5 Z.ai / GLM-5.1 / GLM 5.2

### Access model

- API
- GLM Coding Plan
- Claude Code / Cline / OpenCode compatible tooling

GLM Coding Plan claims:

- starts around $18/month
- 5-hour and weekly limits
- quota roughly equivalent to 15–30× subscription fee by API pricing

June 2026 GLM 5.2 update:

- reported live in GLM Coding Plan first
- 1M context window
- coding-first positioning
- high/max thinking modes, with max recommended for coding
- API and MIT weights reportedly following the plan rollout
- no solid benchmark base at launch yet, so treat early claims cautiously

Approx plan limits:

| Plan | 5-hour prompts | Weekly prompts |
|---|---:|---:|
| Lite | ~80 | ~400 |
| Pro | ~400 | ~2,000 |
| Max | ~1,600 | ~8,000 |

### Caution

There are credible user reports of fair-use/account restriction problems. Treat GLM as technically interesting but operationally risky until reliability improves. GLM 5.2 increases the upside because of long context and coding-first positioning, but it does not remove the operational-risk caveat.

### Best use in pi

- experimental comparison lane
- long-horizon agentic tasks
- GLM 5.2 trial runs on large-context coding tasks
- not yet primary default spend

---

### 6.6 OpenCode Go

OpenCode Go is not a Chinese lab, but it is relevant because it bundles Chinese/open coding models.

### Access model

- $5 first month
- $10/month after
- usage limits expressed as dollar value:
  - $12 / 5h
  - $30 / week
  - $60 / month

Models include:

- DeepSeek V4 Pro / Flash
- Kimi K2.6
- Qwen3.6 Plus
- MiniMax M2.7
- GLM-5.1
- MiMo variants

### Why it matters

This is a low-cost way to test many models without setting up each direct provider.

However, because monthly usage is capped by underlying dollar value, it is not a replacement for heavy direct API usage.

Best use:

- experimentation
- model comparison
- fallback provider
- cheap subscription bundle

---

## 7. Optimisation strategies

### 7.1 Default-cheap routing

Default routine tasks to cheap models:

| Task | Default |
|---|---|
| repo exploration | DeepSeek V4 Flash |
| simple edits | DeepSeek V4 Flash / MiniMax |
| normal implementation | DeepSeek V4 Pro / MiniMax M2.7 |
| test generation | DeepSeek Flash / Pro |
| long-context sweep | DeepSeek Pro / Kimi |
| final review | GPT / Claude / Codex |
| architecture | GPT / Claude first |

The premium models should not be the default for bulk work.

---

### 7.2 Cheap implementation, premium review

Writing code is cheap to automate. Trusted code is expensive.

Recommended flow:

1. DeepSeek/MiniMax writes the patch
2. DeepSeek/Kimi performs adversarial review
3. Tests/lint/typecheck run
4. GPT/Claude reviews only:
   - the task brief
   - the diff
   - relevant files
   - test output
5. Human reviews the final high-risk decisions

This minimizes premium-token use while preserving quality.

---

### 7.3 Escalation policy

Escalate from cheap to premium only when:

- cheap model fails twice
- tests fail in non-obvious ways
- architecture is ambiguous
- data loss/security/privacy risk exists
- model proposes broad unnecessary rewrites
- diff is large and hard to trust
- user intent is subtle

Do not escalate merely because the task is long. Long-context is exactly where DeepSeek pricing helps.

---

### 7.4 Subscription use policy

Use subscriptions while they are subsidised.

A subscription is good when:

- it handles real work inside included limits
- it avoids expensive API calls
- its front end improves success rate
- it provides non-coding value too

A subscription stops making sense when:

- it routinely spills into API-priced credits
- limits interrupt the workflow
- it covers less workload than equivalent DeepSeek spend
- the required first-party front end disrupts pi too much

For this user, against DeepSeek V4 Pro durable 1/4-price:

| Subscription price | Cutoff: must cover this much of current workload |
|---:|---:|
| $20 | >19% |
| $50 | >47% |
| $100 | >94% |
| $200 | >189% |

If a $100 or $200 subscription covers only 20–50% of the current workload before overage, it is not economically competitive with DeepSeek API for bulk work. It may still be worth buying for premium quality, first-party workflow, or non-coding value.

---

### 7.5 Cache preservation

Because cache reads dominate the current spend, pi should track and optimise:

- cache-hit ratio
- cache-write volume
- repeated repo context
- session reuse
- model/provider cache semantics
- whether Anthropic-compatible endpoints preserve cache behaviour
- whether provider routers expose cache discounts

A cheap model without cache benefits may be less cheap than expected.

---

## 8. Recommended stack for this user

## Current best practical stack

### Default API lane

- **DeepSeek V4 Pro** for serious coding
- **DeepSeek V4 Flash** for cheap/simple/background work

### Alternative cheap lane

- **MiniMax M2.7** for everyday coding comparison
- **Kimi K2.6** for hard long-context fallback
- **Qwen3.6 / Qwen Code** for open ecosystem/local experiments

### Premium lane

- **OpenAI Codex / GPT subscription** for hard tasks, first-party Codex workflow, and final review; measure real quota drain before treating it as bulk capacity. In pi, **medium** is now the default reasoning level as an experiment; fall back to low if quota pressure outweighs quality gains.
- **Claude Code / Claude subscription** for interactive first-party Claude Code, architecture, hard debugging, and review; avoid treating Claude subscription as a generic pi/programmatic backend after the Agent SDK credit split

### Experimental subscription bundles

- **MiniMax Token Plan** if request-window limits fit usage
- **OpenCode Go** for cheap multi-model testing
- **Alibaba Coding Plan** if Qwen/Kimi/GLM/MiniMax via a fixed coding quota is useful
- **Z.ai GLM Coding Plan** only with caution due to account/fair-use concerns

---

## 9. Sources refreshed on May 23 / June 14, 2026

Primary/current pages checked for this refresh:

- DeepSeek API docs — Models & Pricing: `https://api-docs.deepseek.com/quick_start/pricing`
- OpenAI Developers — Codex pricing: `https://developers.openai.com/codex/pricing`
- OpenAI Help — Codex rate card: `https://help.openai.com/en/articles/20001106-codex-rate-card`
- OpenAI Help — ChatGPT Pro tiers: `https://help.openai.com/en/articles/9793128-about-chatgpt-pro-tiers`
- OpenAI API — Using GPT-5.5: `https://developers.openai.com/api/docs/guides/latest-model`
- OpenAI API — Reasoning models: `https://developers.openai.com/api/docs/guides/reasoning`
- OpenAI Cookbook — GPT-5 new params and tools: `https://cookbook.openai.com/examples/gpt-5/gpt-5_new_params_and_tools`
- Stet — GPT-5.5 Codex reasoning curve on 26 real tasks: `https://www.stet.sh/blog/gpt-55-codex-graphql-reasoning-curve`
- Anthropic Help — Manage usage credits / Extra Usage for paid Claude plans: `https://support.claude.com/en/articles/12429409-manage-extra-usage-for-paid-claude-plans`
- Anthropic Help — Use the Claude Agent SDK with your Claude plan: `https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan`
- Claude pricing / API pricing docs: `https://claude.com/pricing`, `https://platform.claude.com/docs/en/about-claude/pricing`
- Google AI Gemini API pricing: `https://ai.google.dev/gemini-api/docs/pricing`
- MiniMax M2.7 / Token Plan docs: `https://www.minimax.io/models/text/m27`, `https://platform.minimax.io/docs/guides/pricing-token-plan`
- Kimi K2.6 pricing / membership credits: `https://platform.kimi.ai/docs/pricing/chat-k26`, `https://www.kimi.com/membership-credits`
- Alibaba Model Studio Coding Plan: `https://www.alibabacloud.com/help/en/model-studio/coding-plan`
- Z.ai GLM Coding Plan docs: `https://z.ai/subscribe`, `https://docs.z.ai/devpack/overview`
- OpenCode Go docs: `https://opencode.ai/docs/go/`

---

## 10. Final decision rule

The decision rule is:

> Use subscriptions for subsidised frontier quality; use Chinese APIs for bulk agent work; route by observed success per dollar.

For this user, the most important number is:

> **DeepSeek V4 Pro durable 1/4-price equivalent: ~$106/month for the current workload**

Every subscription should be judged against that for bulk work.

If a subscription costs $100, it must cover almost all of the current workload before overage to beat DeepSeek on raw cost.

If a subscription costs $200, it cannot beat DeepSeek on raw bulk-compute cost for this workload; it must be justified by quality, first-party workflow, reliability, or non-coding value.

If it spills into API-priced usage, Agent SDK credits, or paid usage credits early, it becomes a premium tool, not a bulk compute source.

The system to build in pi is therefore:

1. cost-aware
2. cache-aware
3. model-agnostic
4. subscription-aware
5. capable of routing cheap implementation to premium review

That system is more valuable than any single subscription or model choice.
