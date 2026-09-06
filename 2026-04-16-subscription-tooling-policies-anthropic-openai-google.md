# Subscription tooling policies for Anthropic, OpenAI, Google, and GitHub Copilot

_Date: 2026-04-16; web-search refresh: 2026-06-14_

> **Document role:** retained policy/entitlement analysis using the April 16 and June 14, 2026 sources below, not a present-day entitlement authority. References to “current”, “allowed” and confidence levels describe that research snapshot. Before acting, verify the provider's current official terms for the exact account, authentication route and proposed use; this memo alone grants no permission. The source claims and conclusions below have not been independently reproduced by this clarification. For the related dated routing and cost strategy, see [`2026-06-14-pi-coding-agent-model-routing-and-cost-strategy.md`](2026-06-14-pi-coding-agent-model-routing-and-cost-strategy.md).
>
> **Execution boundary:** do not switch credentials, providers, billing modes or model defaults, install integrations, or launch paid work merely because this note recommends it. Preserve explicit owner/session choices and seek the concrete approval required for those effects. tmux availability does not establish provider permission, and unavailable native delegation is not authority to cross into another OS or checkout.
>
> **June 14 refresh note:** current web-search reinforces the April conclusion, but with sharper boundaries: Anthropic is separating Agent SDK / `claude -p` / GitHub Actions-style programmatic work into a dedicated credit pool from June 15; Google is explicitly rejecting Gemini CLI OAuth piggybacking and appears to be tightening individual Code Assist / CLI serving from June 18; OpenAI remains the most permissive practical Codex subscription lane for personal developer workflow; GitHub Copilot remains the clearest officially-integrated custom-agent/ACP/SDK lane.
>
> **Direction-of-travel note:** Codex looks durable but increasingly quota/rate-card/credit-managed. Copilot is morphing from IDE assistant into a GitHub-native agent platform with CLI, cloud agents, ACP, SDKs, and third-party agents. For `pi`, OpenAI is currently the more usable personal subscription lane; Copilot may become the cleaner official integration platform if GitHub's ACP/SDK surfaces mature.

## Why this note exists

This session started from a practical developer question: _what is actually allowed under consumer/subscription plans when using coding tools, CLIs, tmux, agent SDKs, and third-party frontends?_ The immediate trigger was confusion around Claude subscription usage, but the work expanded into a cross-provider comparison because the same boundary problem appears at Anthropic, OpenAI, Google, and GitHub Copilot.

The user's intended workflow matters:

- use **pi** as the personal front end
- use providers' **official tools** as sub-agents, driven through **tmux**
- understand where subscriptions stop being acceptable and where API/commercial auth is required

This note is written for that practical developer use case.

---

## Session reference

- **Session file:** `/home/jack/.pi/agent/sessions/--home-jack-tmp--/2026-04-16T11-55-42-215Z_019d9625-a307-7699-9f5d-c40405e12af5.jsonl`
- **Session ID:** `019d9625-a307-7699-9f5d-c40405e12af5`

---

## Bottom line

The main conclusion from this session is simple:

1. **Anthropic** currently treats subscriptions as a **first-party entitlement**: use Claude in Anthropic-owned tools, especially Claude app + ordinary interactive Claude Code. Third-party harness use with subscription OAuth has been actively shut down, and June 2026 web-search shows programmatic Agent SDK / `claude -p` / Claude Code GitHub Actions / third-party Agent SDK app usage moving to a separate monthly credit pool rather than the ordinary chat/Claude Code subscription bucket.
2. **OpenAI** currently allows the broadest subscription-based coding workflow ecosystem. ChatGPT/Codex subscriptions are explicitly usable across official Codex clients, and current docs/community material remain unusually friendly to alternate developer tooling. But this still does **not** mean "use your subscription as a backend for a service or for other users"; Codex docs/help continue to point CI, shared, and clearly programmatic backend work toward API/credit economics.
3. **Google** allows subscription use in official Gemini surfaces and Gemini CLI, but explicitly forbids third-party tools from piggybacking on Gemini CLI OAuth. For third-party coding agents, Google wants API key / Vertex AI usage. June 2026 search also found Google Code Assist/Gemini CLI notices saying individual / AI Pro / AI Ultra Code Assist serving would stop from June 18, 2026, so Google subscription auth should be treated as particularly unstable for critical coding-agent workflows.
4. **GitHub Copilot** appears to be the most explicitly supportive of third-party frontends and agent orchestration. Copilot CLI exposes official automation and integration surfaces, ACP support, third-party coding agents on GitHub, and Copilot SDKs intended for custom frontends, automation systems, and multi-agent workflows.
5. **tmux is probably not the real issue** for any provider, but the confidence level differs by provider. The more defensible policy question is whether the workflow still counts as **your own use of the provider's official tool**, or whether it has become **third-party infrastructure / backend use**.
6. For the user's preferred setup — **pi as front end, official provider tools as tmux-driven sub-agents** — the policy position differs by provider:
   - **Anthropic:** subscription use through `pi` as the front end is not defensible from current docs. Using **Claude Code in tmux** may be fine, but this session did **not** find an explicit Anthropic statement blessing tmux-orchestrated use specifically.
   - **OpenAI:** much more likely to be tolerated if the work is still flowing through supported Codex auth/tooling and not powering a third-party service.
   - **Google:** not acceptable if pi is reusing Gemini CLI OAuth from outside official Gemini tooling.
   - **GitHub Copilot:** `pi` as front end looks actively supported by official integration surfaces such as ACP and the Copilot SDK.

---

## The core policy problem

The hard question in 2026 is no longer just "manual vs automated". It is:

- **official/native user tool use** vs
- **third-party harness or backend use**

Providers now ship powerful official coding agents and CLIs. That means ordinary individual use naturally includes:

- terminal use
- noninteractive runs
- background jobs
- tmux
- scripts
- agent loops
- IDE integrations

The old boundary — "human at keyboard" vs "API/backend" — no longer maps cleanly onto actual developer workflows.

The providers respond differently:

- Anthropic and Google are drawing a relatively hard line around **approved surfaces**.
- OpenAI appears more willing to let subscription usage extend across a broader coding ecosystem, as long as it is not turned into a service backend.

---

## Provider-by-provider practical rules

## GitHub Copilot

### What subscription can be used with

GitHub now supports a wide range of Copilot subscription surfaces and integration patterns:

- GitHub Copilot on GitHub
- Copilot Chat
- Copilot cloud agent
- **Copilot CLI**
- third-party coding agents inside GitHub's own agent platform
- **ACP** integration from external tools/frontends into Copilot CLI
- **Copilot SDK** embedding into applications and workflows

Key official sources found in this session:

- About third-party agents: `https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents`
- Copilot SDK repo: `https://github.com/github/copilot-sdk`
- Copilot features: `https://docs.github.com/en/copilot/get-started/features`
- Copilot CLI GA changelog: `https://github.blog/changelog/2026-02-25-github-copilot-cli-is-now-generally-available/`
- Copilot CLI automation/scripting changelog: `https://github.blog/changelog/2026-01-14-github-copilot-cli-enhanced-agents-context-management-and-new-ways-to-install/`
- ACP support in Copilot CLI: `https://github.blog/changelog/2026-01-28-acp-support-in-copilot-cli-is-now-in-public-preview/`
- Copilot SDK public preview: `https://github.blog/changelog/2026-04-02-copilot-sdk-in-public-preview/`

### What subscription seems allowed for

GitHub is unusually explicit about supporting agentic orchestration and third-party integration.

From the sources gathered here, Copilot subscription use clearly supports:

- official Copilot CLI use
- `copilot -p` and scripting/pipeline workflows
- cloud/background delegation
- custom agents, plugins, hooks, and MCP integrations
- third-party coding agents on GitHub itself
- **third-party tools, IDEs, automation systems, custom frontends, and multi-agent systems** via **ACP**
- direct embedding through the **Copilot SDK**

This is the strongest official fit in this session for the user's preferred workflow of using **pi as the personal front end**.

June 2026 web-search reinforces this: GitHub docs/search results describe third-party coding agents alongside Copilot cloud agent, ACP support in Copilot CLI, and Copilot SDKs for embedding Copilot's agentic workflows into applications. That does not remove normal product/platform limits, but it is a much more explicit integration posture than Anthropic or Google.

Direction of travel: Copilot is becoming less like a single assistant and more like GitHub's agent platform. The relevant policy signal is not just "Copilot has a CLI"; it is that GitHub is standardising integration surfaces for external clients and agents. For `pi`, this may become strategically more important than raw model quality if it offers the cleanest subscription-compatible orchestration path.

### What subscription is not clearly allowed for

This session did not find a GitHub statement forbidding personal custom frontends in the way Anthropic and Google do. The main limits found are the normal product/service boundaries and general platform terms, not a prohibition on custom orchestration.

GitHub clearly does permit a lot of development-focused automation and integration. The main caution is to stay inside normal product terms and avoid turning Copilot into a prohibited stand-alone service or abusing GitHub-hosted infrastructure in ways unrelated to development.

### Practical rule for GitHub Copilot

GitHub Copilot is the cleanest fit for:

- **pi direct** as the personal front end
- official Copilot agent backends behind it
- tmux, ACP, SDK, scripting, and local orchestration

Among the providers in this note, Copilot is the one with the strongest official support for exactly this style of workflow.


## Anthropic

### What subscription can be used with

Allowed official surfaces identified in the research:

- `claude.ai` / Claude web
- Claude desktop/mobile
- **Claude Code**

Relevant official evidence:

- Claude Code legal/compliance: `https://docs.anthropic.com/en/docs/claude-code/legal-and-compliance`
- Consumer Terms: `https://www.anthropic.com/legal/consumer-terms`
- Pro/Max + Claude Code help article: `https://support.anthropic.com/en/articles/11145838-using-claude-code-with-your-max-plan`

### What subscription seems allowed for

- personal use in official Anthropic interfaces
- Claude Code terminal use
- ordinary individual usage of Anthropic-owned tools

### What remains uncertain for Anthropic

This session did **not** find an official Anthropic statement specifically addressing:

- running Claude Code inside `tmux`
- having another local tool orchestrate Claude Code purely at the shell/session layer without reusing subscription OAuth itself

So the strongest defensible statement is narrower:

- **Claude Code itself is allowed on subscription**
- **third-party products using Claude subscription OAuth are not**
- **tmux-layer orchestration remains an inference zone, not a directly documented Anthropic allowance**

### What subscription is not allowed for

Strong current signal:

- third-party tools/harnesses using **Claude subscription OAuth**
- routing Claude subscription creds through another product/tool/service
- using consumer subscription auth as a substitute for API auth
- building products/services on top of Claude consumer OAuth

### Important current nuance: Agent SDK / programmatic credits

The April reading was already conservative: **do not rely on subscription OAuth for Agent SDK usage**. The June 14 web-search makes that boundary sharper rather than softer.

Current Anthropic help/reporting indicates that from **2026-06-15**, paid Claude plans include a separate monthly credit for programmatic usage, covering things like:

- Claude Agent SDK usage
- `claude -p` style scripted/programmatic usage
- Claude Code GitHub Actions
- third-party apps authenticating through the Agent SDK

The practical meaning is that this work is no longer safely understood as drawing from the ordinary interactive Claude / Claude Code subscription allowance. It should be treated as a **separate credit/API-style pool**, often discussed as being billed at standard API list-price economics once the included credit is exhausted.

Practical conclusion after the June refresh:

- **interactive Claude app / ordinary Claude Code** remains the subscription lane
- **Agent SDK, `claude -p`, GitHub Actions, third-party Agent SDK apps, CI, automation, and third-party harness use** should be accounted for as programmatic credit/API-style usage
- **do not build pi's core backend strategy around Claude subscription arbitrage**
- use Claude subscription primarily as an official-tool lane for interactive hard debugging/review, and use API/credits when the workflow is integrated, automated, or third-party

### Recent policy and enforcement changes

This session found strong evidence that Anthropic tightened and enforced this in early 2026.

Useful recent evidence:

- The Register (Feb 20, 2026): Anthropic clarified that using OAuth tokens from Claude Free/Pro/Max in other products/tools/services is not permitted, including the Agent SDK.
- Multiple April 2026 reports around the **April 4, 2026** cutoff for OpenClaw-style subscription usage.
- Anthropic's own current docs match the direction of that enforcement.

### Practical rule for Anthropic

**Subscription = Anthropic-owned tools only** is the strongest rule directly supported by current docs.

For Anthropic, this session supports the following conclusions with different confidence levels:

- **pi using Claude subscription OAuth:** not fine
- **third-party harness using Claude subscription OAuth:** not fine
- **API/commercial auth:** required for third-party agent tooling
- **official Claude Code in tmux:** plausible, but not explicitly documented in the sources gathered here

So the safest Anthropic reading is:

> use the official Anthropic client surface directly for subscription usage; do not assume Anthropic approves extra orchestration layers unless they say so explicitly.

---

## OpenAI

### What subscription can be used with

OpenAI's current docs show ChatGPT subscription access working across the Codex product family:

- ChatGPT / ChatGPT web/app
- **Codex app**
- **Codex CLI**
- **Codex IDE extension**
- Codex cloud tasks
- Codex MCP/tool ecosystem

Key official sources:

- Codex auth: `https://developers.openai.com/codex/auth`
- Codex CLI: `https://developers.openai.com/codex/cli`
- Codex pricing: `https://developers.openai.com/codex/pricing`
- Codex main docs: `https://developers.openai.com/codex`
- ChatGPT Pro help: `https://help.openai.com/en/articles/9793128-what-is-chatgpt-pro`

### What subscription seems allowed for

OpenAI is the broadest of the three providers researched here.

Official docs support:

- ChatGPT sign-in for Codex subscription access
- local CLI use
- IDE use
- app use
- cloud tasks
- scripting Codex
- MCP integrations
- background/parallel coding workflows within Codex
- substantial day-to-day developer automation inside OpenAI's own coding ecosystem

### Strong signal about alternate tooling

The strongest single signal found in this session was OpenAI's **Codex for Open Source** page:

- `https://developers.openai.com/community/codex-for-oss`

That page explicitly says:

> Developers should code in the tools they prefer, whether that's Codex, OpenCode, Cline, pi, OpenClaw, or something else.

That does **not** by itself prove every third-party OAuth flow is universally allowed. But it is a very strong policy/culture signal that OpenAI is actively comfortable with a broader coding-tool ecosystem than Anthropic or Google.

Additional supporting signals from current docs/search:

- Codex app-server docs explicitly support deep integrations and describe ChatGPT-managed auth
- Codex docs support scripting and rich local automation rather than trying to confine all usage to a narrow official shell
- June 2026 Codex pricing/rate-card search results still describe Codex as included in ChatGPT Plus/Pro tiers, with Pro $100 / Pro $200 positioned by usage multipliers and with extra credits/rate-limit resets used to manage overflow
- OpenAI help/search results now frame Codex cost in developer/month terms with large variance by model, number of instances, automation, and fast-mode usage — i.e. subscription use is useful capacity, not unlimited compute

### What subscription is not allowed for

OpenAI still draws lines.

From ChatGPT Pro help:

- no abusive or programmatic data extraction
- no sharing credentials
- no making the account available to others
- no reselling access
- no using ChatGPT to power third-party services

Codex auth docs also recommend:

- **API keys for programmatic Codex CLI workflows**, explicitly naming CI/CD jobs

### Practical rule for OpenAI

OpenAI appears to allow the broadest subscription-based developer use, including alternate tooling, **provided it remains your use and not service/backend use**.

For the user's preferred setup:

- **pi as front end + official Codex/Codex auth in tmux-driven subagents:** much more plausible than with Anthropic or Google
- **CI / shared service / backend-for-others:** use API keys instead

This is the closest thing found in this session to a subscription model that tolerates a sophisticated personal coding-agent setup.

June 2026 update: this conclusion still holds, but Codex should be treated as **quota/credit-managed first-party capacity**, not a loophole around API economics. Search results point toward token/rate-card-managed capacity, model-dependent message/task limits, paid/credit overflow, and banked rate-limit resets. If Codex starts behaving like CI, shared automation, service infrastructure, or paid overflow, switch that work to API accounting.

Direction of travel: OpenAI appears to be making Codex a durable first-party coding product, but one with increasingly explicit metering. For personal interactive work this is good; for bulk automation it means Codex should not be your only backend assumption.

---

## Google

### What subscription can be used with

Google currently supports subscription use in:

- Gemini app / Gemini web experience
- Gemini Connected Apps / in-product integrations
- **Gemini CLI** via **Sign in with Google**
- Gemini Code Assist / official IDE agent mode

Key official sources found:

- Gemini CLI auth docs: `https://github.com/google-gemini/gemini-cli/blob/main/docs/get-started/authentication.md`
- Gemini CLI quota/pricing: `https://github.com/google-gemini/gemini-cli/blob/main/docs/resources/quota-and-pricing.md`
- Gemini CLI FAQ: `https://geminicli.com/docs/resources/faq/`
- Gemini Connected Apps help: `https://support.google.com/gemini/answer/13695044`

### What subscription seems allowed for

- official Gemini CLI use with Sign in with Google, subject to current serving/quota rules
- official Gemini app use
- official Connected Apps / integrations inside Gemini
- higher subscription quotas for Gemini CLI and Gemini Code Assist under Google AI Pro / Ultra where Google still offers those individual tiers for the relevant product surface

June 2026 caution: web-search found Google developer/docs snippets saying **Gemini Code Assist IDE Extensions and Gemini CLI will stop serving requests for Gemini Code Assist for individuals, Google AI Pro, and Google AI Ultra tiers starting June 18, 2026**. Treat this as a major operational warning: even official-tool subscription access can change abruptly, and third-party piggybacking remains explicitly disallowed.

### What subscription is not allowed for

Google's current FAQ is unusually explicit.

It says:

> Using third-party software, tools, or services to harvest or piggyback on Gemini CLI’s OAuth authentication to access our backend services is a direct violation...

And it says the supported alternative is:

- **Vertex AI**, or
- **Google AI Studio API key**

The auth docs also distinguish:

- local users can use **Sign in with Google**
- **headless mode** should use **API key** or **Vertex AI**

That is an important practical clue: Google is clearly treating subscription OAuth as an official-tool auth path, not a general-purpose programmable credential.

### Practical rule for Google

- **official Gemini CLI in tmux:** likely fine only while the official individual/AI Pro/AI Ultra serving path remains available
- **third-party harness reusing Gemini CLI OAuth:** not fine
- **pi using Gemini CLI OAuth as a transport:** not fine
- **third-party coding agent with Gemini models:** use API key / Vertex AI
- **critical coding-agent workflows:** avoid depending on Google subscription OAuth; prefer API/Vertex or another provider lane

### Note on Google's in-product openness

Google does support third-party and external app connections **inside Gemini itself** via Connected Apps and workspace integrations. That is real openness, but it is not the same as allowing external coding harnesses to reuse Gemini CLI subscription OAuth.

---

## The tmux question

A lot of the session turned on whether `tmux` changes the policy character of use.

The strongest conclusion this session can defend is:

- **no source gathered here treats `tmux` itself as the policy boundary**
- `tmux` is just terminal/session management
- what matters is **what tool is running inside tmux**, **how it is authenticated**, and **whether another product is effectively becoming the real client**

Confidence varies by provider:

- running **official Codex** in tmux -> consistent with OpenAI's documented CLI/app scripting and automation posture
- running **official Gemini CLI** in tmux -> likely okay, because Google objects specifically to third-party piggybacking on Gemini OAuth rather than to the official CLI itself
- running **official Claude Code** in tmux -> plausible, but this session did not find an explicit Anthropic statement about tmux use, so this should be treated as an inference rather than a documented allowance

What the sources do support clearly is:

- Anthropic and Google object to **third-party products reusing subscription OAuth**
- OpenAI is much more open to alternate tooling and orchestration

So the most useful conceptual rule from the session is still:

> The real issue is not backgrounding or automation by itself; it is whether the workflow is still your use of the provider's tool, or whether your subscription has become infrastructure.

But for Anthropic specifically, tmux-based orchestration should be treated as **uncertain**, not as clearly approved.

---

## Practical matrix

| Provider | Subscription-allowed tools | Allowed purposes on subscription | Not allowed on subscription | When to switch to API/commercial auth |
|---|---|---|---|---|
| **Anthropic** | Claude web/app, ordinary interactive Claude Code | Personal use in official Anthropic tools | Third-party tools/harnesses using Claude subscription OAuth; using consumer creds in products/services; Agent SDK / `claude -p` / GitHub Actions / third-party Agent SDK apps should not be treated as ordinary subscription usage | Any third-party harness, Agent SDK work, CI/automation, product/service use, backend use; account as programmatic credits/API-style usage |
| **OpenAI** | ChatGPT, Codex app/CLI/IDE, Codex cloud, Codex MCP ecosystem | Broad personal/professional coding use; scripting Codex; cloud tasks; local automation inside Codex ecosystem | Credential sharing, resale, powering third-party services, abusive extraction; treating Codex quota as unlimited backend compute | CI/CD, shared environments, backend/service use, clear programmatic platform use, paid overflow/credits |
| **Google** | Gemini app, Gemini Connected Apps, Gemini CLI, Gemini Code Assist where currently served | Official Gemini surfaces, local CLI use, in-product integrations | Piggybacking on Gemini CLI OAuth from third-party tools; external harness reuse of subscription auth; depending on individual/AI Pro/AI Ultra Code Assist/CLI serving for critical workflows | Third-party coding agents, headless use, programmable integrations, critical coding-agent workflows; use API/Vertex |
| **GitHub Copilot** | Copilot on GitHub, Copilot CLI, cloud agent, ACP, SDK, third-party agents on GitHub | Development-focused automation, scripting, custom frontends, agent orchestration, third-party tool integration | No specific anti-custom-frontend rule found in this session; still subject to general product/platform limits | When turning it into an operational service beyond personal/dev use, or where another billing/auth model is clearly required |

---

## What this means for the user's intended setup

The user preference stated at the end of the session was:

- **use pi as the personal front end**
- **use everything else as sub-agents through tmux**
- **use only official tools**

This is not equally viable across providers.

### Anthropic

This is the worst fit.

If pi is the user-facing front end and Claude is being used behind that via subscription credentials, Anthropic's current policy direction strongly suggests that this is **not allowed**. Anthropic wants subscription use confined to Anthropic's own surfaces.

The most defensible subscription-safe Anthropic pattern found here is:

- the human uses **Anthropic's own client surface directly**
- for coding, that means **Claude Code**

Whether direct use of Claude Code can be further wrapped in `tmux` or shell-level orchestration without leaving the safe lane remains **uncertain** from the sources gathered here. This note should not be read as claiming Anthropic has explicitly approved that pattern.

### OpenAI

This is a strong fit.

OpenAI's current posture is highly compatible with:

- a personal developer using advanced local tooling
- alternate front ends and harnesses in the coding ecosystem
- Codex-based automation that is still for the individual user's work

It is still necessary to avoid crossing into:

- service/backend use
- credential sharing
- powering other users

### Google

This is a poor fit if pi is reusing Google subscription auth externally.

Google does allow official Gemini CLI and Gemini app use, but current docs explicitly reject third-party piggybacking on Gemini CLI OAuth. So pi-as-front-end is not compatible with Google subscription auth in the same way the user wants.

### GitHub Copilot

This is the best fit.

GitHub is not merely tolerant of custom frontends and orchestration; it is actively documenting and shipping them.

For the user's intended setup, GitHub Copilot is the clearest subscription-compatible option for:

- **pi as the personal front end**
- official Copilot backends underneath
- tmux and local orchestration where useful
- ACP or SDK-based integration rather than credential piggybacking

---

## Why the providers are doing this

This was a recurring theme in the session.

Rate limits alone do **not** explain provider behavior. If they did, Anthropic and Google would not care so much about the "wrong frontend."

The more convincing explanation reached here is:

- **product/channel control**
- **pricing segmentation** (subscription vs API)
- **preventing consumer subscriptions from turning into low-cost infrastructure**
- preserving telemetry, safety controls, auth boundaries, and supportability

OpenClaw mattered because it highlighted this difference clearly:

- an individual user could still be banned or blocked even without sharing or resale, simply for using the wrong access surface

That implies the actual provider rule is often:

> subscription access is not a general right to model use; it is a right to use the provider's subscription product through approved channels, for approved purposes.

OpenAI currently appears more ecosystem-friendly than Anthropic or Google, but even OpenAI still draws a line at using subscription access to power services for others.

---

## Confidence and evidence quality

### High-confidence findings

These came directly from current official docs:

- Anthropic: subscription OAuth is intended for native Anthropic tools; product/service developers should use API keys
- Google: piggybacking on Gemini CLI OAuth from third-party tools is a direct policy violation
- OpenAI: Codex supports ChatGPT sign-in for subscription access; API key is recommended for CI/programmatic workflows; ChatGPT Pro forbids powering third-party services

### Medium-confidence findings

These depend on combining official docs with recent public reporting/community evidence:

- Anthropic's April 2026 OpenClaw cutoff as a fully enforced practical boundary
- OpenAI's effective permissiveness toward alternate coding front ends using ChatGPT/Codex subscription auth
- the exact extent to which OpenAI would tolerate pi as the personal front end in every implementation detail

### Main uncertainty that remains

The hardest unresolved issue is not the official docs themselves; it is **how providers classify sophisticated personal automation that still uses official tools and official auth, but is orchestrated by another local system**.

This uncertainty matters most for OpenAI, because OpenAI is the provider most likely to permit what the user wants, but the exact edge of that permissiveness is still policy- and implementation-dependent.

---

## Practical decision rule going forward

For a personal coding-agent setup, the most useful rule from this session is:

1. If the provider says subscription auth is for **its own tools only**, believe it.
2. If the provider explicitly condemns third-party piggybacking on official OAuth, do not try to route that OAuth through pi.
3. If the provider explicitly supports a broad coding ecosystem and alternate tools, subscription use may be workable — but still avoid turning it into a backend/service.
4. `tmux` is fine; the key question is not "backgrounded or not?" but "is this still my use of the provider's tool, or has my subscription become infrastructure?"

## Low-ban-risk operating policy

After the follow-up discussion, the most useful framing is not abstract policy theory but:

> how should a developer actually use these tools to advance their own work without taking unnecessary account-risk?

The practical answer is to distinguish between:

- **tools I use directly**
- **tools I integrate deeply into pi**

That distinction is not morally perfect, but it matches the current enforcement reality well enough to be operationally useful.

### Recommended default split

#### First-class integrated providers inside pi

These are the strongest fits for `pi` as the actual front end:

- **GitHub Copilot**
- **OpenAI / Codex**

These providers currently have the best documented support for custom frontends, scripting, agent orchestration, or broad personal coding-tool use.

#### Official-tool sidecars, not core subscription backends

These are better treated as tools the human uses directly, even if they are launched and managed with `tmux`:

- **Claude Code**
- **Gemini CLI**

This means:

- keep them as official terminal tools
- use them manually or with light session management
- do **not** make them critical hidden subscription backends behind `pi`

If they need to become real programmable backends, switch to:

- **Anthropic API keys**
- **Google AI Studio API key / Vertex AI**

### Green / yellow / red guidance

#### Green zone

Patterns with the lowest practical account-risk from the evidence gathered here:

- **OpenAI**: personal local use, Codex CLI/app/IDE, `pi` as front end, tmux, scripting, subagents, moderate automation
- **GitHub Copilot**: `pi` as front end, ACP, Copilot CLI, SDK, custom frontends, local orchestration
- **Anthropic**: Claude Code used directly as an official tool; API key for integrations
- **Google**: Gemini CLI used directly as an official tool; API key or Vertex AI for integrations and automation

#### Yellow zone

Patterns that may work but carry elevated policy or enforcement risk:

- **Anthropic**: shell-driving Claude Code from another tool; `pi` supervising Claude Code sessions through tmux; heavy orchestration while still relying on subscription auth
- **Google**: shell-driving Gemini CLI from another tool; `pi` mediating Gemini CLI sessions; any workflow that starts to make Gemini CLI look like an unofficial backend

#### Red zone

Patterns that should be treated as unsafe if ban risk matters:

- **Anthropic**: Claude subscription OAuth inside `pi`; any third-party tool using Claude.ai login; Agent SDK on subscription OAuth; turning Claude subscription access into backend infrastructure
- **Google**: any third-party tool piggybacking on Gemini CLI OAuth; using Gemini subscription auth as a hidden provider backend; relying on Google subscription auth for automation you care about operationally

### Practical consequence for provider choice

For day-to-day work, the current safest operating center of gravity is:

- **OpenAI + GitHub Copilot** as the deeply integrated `pi` backends
- **Claude Code** as a direct-use official sidecar
- **Google subscription usage** treated as fragile and unsuitable for critical integrated workflows

Anthropic's position is annoying but workable because it still leaves a clear lane:

- use **Claude Code directly** on subscription
- use **API keys** for anything integrated

Google's current position is harsher in practice because once subscription/OAuth access is shut down, the fallout can affect multiple Google developer workflows at once. From a risk-management standpoint, that makes Google subscription auth a poor foundation for any important orchestrated setup.

### Single conservative rule

If a workflow matters enough that losing it would hurt, use this rule:

- **Subscription auth** only for your direct use of the provider's official tool, with at most light terminal/session management
- **API/commercial auth** for anything integrated, orchestrated, headless, repeated, or important

That rule is conservative for OpenAI and Copilot, but robust across all four providers.

---

## Links gathered in this session

### Anthropic

- Consumer Terms: `https://www.anthropic.com/legal/consumer-terms`
- Claude Code legal/compliance: `https://docs.anthropic.com/en/docs/claude-code/legal-and-compliance`
- Using Claude Code with Pro/Max: `https://support.anthropic.com/en/articles/11145838-using-claude-code-with-your-max-plan`
- Extra usage for paid Claude plans: `https://support.claude.com/en/articles/12429409-manage-extra-usage-for-paid-claude-plans`
- Use the Claude Agent SDK with your Claude plan: `https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan`
- Agent SDK overview: `https://docs.anthropic.com/en/docs/claude-code/sdk/sdk-overview`
- Usage Policy / AUP: `https://www.anthropic.com/legal/aup`

### OpenAI

- Codex auth: `https://developers.openai.com/codex/auth`
- Codex CLI: `https://developers.openai.com/codex/cli`
- Codex pricing: `https://developers.openai.com/codex/pricing`
- Codex rate card: `https://help.openai.com/en/articles/20001106-codex-rate-card`
- Codex main docs: `https://developers.openai.com/codex`
- Codex app-server: `https://developers.openai.com/codex/app-server`
- Codex MCP: `https://developers.openai.com/codex/mcp`
- Codex for Open Source: `https://developers.openai.com/community/codex-for-oss`
- ChatGPT Pro help: `https://help.openai.com/en/articles/9793128-what-is-chatgpt-pro`

### GitHub Copilot

- About third-party agents: `https://docs.github.com/en/copilot/concepts/agents/about-third-party-agents`
- Copilot SDK repo: `https://github.com/github/copilot-sdk`
- Copilot features: `https://docs.github.com/en/copilot/get-started/features`
- Copilot policies for individual subscribers: `https://docs.github.com/en/copilot/managing-copilot/managing-copilot-as-an-individual-subscriber/managing-your-copilot-plan/managing-copilot-policies-as-an-individual-subscriber`
- Copilot CLI enhanced agents/changelog: `https://github.blog/changelog/2026-01-14-github-copilot-cli-enhanced-agents-context-management-and-new-ways-to-install/`
- Copilot CLI GA: `https://github.blog/changelog/2026-02-25-github-copilot-cli-is-now-generally-available/`
- ACP support in Copilot CLI: `https://github.blog/changelog/2026-01-28-acp-support-in-copilot-cli-is-now-in-public-preview/`
- Copilot SDK public preview: `https://github.blog/changelog/2026-04-02-copilot-sdk-in-public-preview/`

### Google

- Gemini CLI auth: `https://github.com/google-gemini/gemini-cli/blob/main/docs/get-started/authentication.md`
- Gemini CLI quota/pricing: `https://github.com/google-gemini/gemini-cli/blob/main/docs/resources/quota-and-pricing.md`
- Gemini CLI FAQ: `https://geminicli.com/docs/resources/faq/`
- Gemini Code Assist FAQ: `https://developers.google.com/gemini-code-assist/resources/faqs`
- Gemini Code Assist Standard/Enterprise setup: `https://docs.cloud.google.com/gemini/docs/codeassist/set-up-gemini`
- Gemini Connected Apps help: `https://support.google.com/gemini/answer/13695044`
- Gemini Privacy Hub: `https://support.google.com/gemini/answer/13594961`

### Community / reporting used as supporting evidence

- The Register on Anthropic clarification and April enforcement
- Various web-search-discovered reports on OpenClaw / Anthropic / Google policy shifts in Feb-Apr 2026
- June 2026 web-search results from Anthropic Help, The New Stack, AgenticWire, and developer-community reporting on Agent SDK credit-pool changes
- June 2026 web-search results from Google developer docs / Cloud docs on Gemini Code Assist and Gemini CLI serving changes
- June 2026 web-search results from OpenAI Codex pricing/rate-card pages and GitHub Copilot ACP/SDK/third-party-agent pages
- OpenClaw and GitHub issue/discussion pages as secondary evidence for enforcement timing and user-visible breakage

---

## Personal rulebook for this workflow

The user's practical question is narrower than general API policy:

> Can I use **pi directly** with the provider under subscription, or must I use **tmux + the provider's official tool**?

### Pi direct

- **GitHub Copilot:** **yes** — strongest official support for custom frontends and orchestration
- **OpenAI:** **likely yes** — broad current support for personal coding workflows across a wider tooling ecosystem
- **Anthropic:** **no** — not defensible from current subscription docs
- **Google:** **no** — explicitly contrary to current Gemini CLI OAuth guidance

### tmux + official tool

- **GitHub Copilot:** yes
- **OpenAI:** yes
- **Anthropic:** ordinary interactive Claude Code is plausible, but Agent SDK / `claude -p` / GitHub Actions / automation should be treated as separate programmatic credit/API-style usage
- **Google:** official Gemini CLI only while the official subscription serving path remains available; do not rely on it for critical orchestrated workflows

## Final practical takeaway

If the goal is:

- **pi as the personal front end**
- **official tools as tmux-driven sub-agents**

then the provider fit is:

- **GitHub Copilot:** best fit for official/custom agent-platform integration; strategically improving as ACP/SDK mature
- **OpenAI:** strongest current fit for day-to-day personal Codex subscription use, but increasingly metered
- **Anthropic:** safest reading is to use the official tool directly for interactive Claude/Claude Code; use credits/API accounting for Agent SDK, `claude -p`, CI, GitHub Actions, third-party apps, or anything integrated into pi
- **Google:** use the official tool directly only where current serving still supports it; for pi/integrated workflows move to API/Vertex usage

So the durable rule from this session is:

> For subscription workflows, use pi directly only where the provider is clearly comfortable with third-party orchestration. From the sources gathered here, GitHub Copilot is the clearest yes, OpenAI is likely yes, and Anthropic/Google are no. For Anthropic, do not overclaim that tmux-layer orchestration has been explicitly approved; for programmatic Agent SDK-style workflows, assume separate credits/API economics. For Google, do not build critical pi workflows on Gemini CLI OAuth.
