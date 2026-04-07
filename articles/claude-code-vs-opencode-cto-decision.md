---
title: "Claude Code vs OpenCode: The CTO Decision Nobody Talks About Honestly"
description: "A production builder's breakdown of Claude Code vs OpenCode + OpenRouter. Real cost math, GLM setup, QA trade-offs, and the Python vs MATLAB lesson for AI tooling."
date: "09/03/2026"
author: "Netanel Eliav"
series: "CTO"
tags: ["Claude Code", "OpenCode", "OpenRouter", "GLM", "AI coding tools", "production AI", "London", "CTO"]
slug: "claude-code-vs-opencode-cto-decision"
featured_image: "https://raw.githubusercontent.com/iNetanel/blog/main/assets/images/claude-code-vs-opencode-cto-decision.jpeg"
featured: true

faqs:
  - q: "Is OpenCode a real alternative to Claude Code for production development?"
    a: "Yes, but the trade-offs are non-trivial. OpenCode gives you model freedom (75+ providers via OpenRouter), zero vendor lock-in, and meaningful cost reduction, running GLM-5 via OpenRouter at $0.80/M input vs Sonnet's $3/M is a 3.75x difference on input tokens, and MiniMax M2.5 at $0.30/M pushes that to 10x. In practice, for mid-complexity tasks on a typical day you can drive total spend down 50–65% vs a Claude Max $100/month plan. The cost case is real. What you trade is orchestration quality and feature release pace: Claude Code ships weekly updates, integrates natively with Anthropic's evaluation loops, and its harness is explicitly tuned for Claude models. OpenCode requires more QA on your end because the abstraction layer introduces inconsistency when providers change APIs, tool-calling formats, or rate limits under you. My rule: OpenCode for cost-optimized long-running projects where you control the model mix. Claude Code when you need tight reliability and the fastest access to frontier capabilities."

  - q: "How do I set up OpenCode with VSCode and OpenRouter to use GLM models?"
    a: "The setup is three steps, but there's a known bug you'll hit immediately. First: install OpenCode globally (npm install -g opencode-ai or brew install opencode). Run opencode auth login, select OpenRouter, and enter your API key from openrouter.ai. Run opencode to launch the TUI and /models to confirm models appear. Second: add the VSCode extension (sst-dev.opencode-v2). Here's the bug, as of early 2026 the extension's loadOpenCodeConfig() function hardcodes only two providers (anthropic and opencode). Your OpenRouter models will be invisible in the extension even though they work fine in CLI. Workaround: set OPENROUTER_API_KEY in your shell environment and configure the provider directly in ~/.opencode/config.json. Third: select your model. For cost efficiency, use GLM-5 (via openrouter/z-ai/glm-5) for reasoning-heavy and complex agent tasks, MiniMax M2.5 (via openrouter/minimax/minimax-m2.5) for high-volume output-heavy tasks where the $1.20/M output price matters, and GLM-4.5 Air (free tier) for simple file edits and grep tasks. Route expensive Anthropic tokens only for complex multi-file refactors. The VSCode extension bug is tracked in issue #6066 on the sst/opencode repo, check if it's resolved before you waste an hour on it."

  - q: "How much cheaper is GLM via OpenRouter compared to Claude Sonnet?"
    a: "The cost difference is significant but depends on your workload composition. GLM-5 via OpenRouter prices at $0.80/M input tokens and $2.56/M output tokens. MiniMax M2.5 prices at $0.30/M input and $1.20/M output, with automatic caching included at no extra configuration. Claude Sonnet 4.6 is $3.00/M input and $15.00/M output. On input, GLM-5 is 3.75x cheaper than Sonnet. MiniMax M2.5 at $0.30/M input is 10x cheaper. On output, which dominates agentic coding costs, GLM-5 at $2.56/M is 5.9x cheaper than Sonnet. MiniMax M2.5 at $1.20/M is 12.5x cheaper. Real-world agentic sessions are output-heavy: a typical 10-step coding loop consuming 150K tokens might split 40% input / 60% output. At that ratio, GLM-5 costs roughly $1.83 per session versus $9.90 for Sonnet, about an 81% reduction. MiniMax M2.5 comes in at roughly $0.84 per session, an 91% reduction. The comparison isn't clean: both models require more iterations on ambiguous tasks, which adds output tokens back. In practice across multiple agentic projects, expect 55–70% total cost reduction versus Claude Sonnet. The sweet spot: use GLM-5 for complex reasoning and architecture-heavy tasks, use MiniMax M2.5 for high-volume greenfield work and well-specified feature implementation, use Claude for debugging and hard multi-file reasoning."

  - q: "What are the QA problems when using OpenCode with non-Anthropic models?"
    a: "Three categories of failure, all from real usage. First: tool-calling inconsistency. OpenCode standardizes how prompts are sent and how tools are invoked, but models differ dramatically in how reliably they call tools versus narrating what they'd do. GLM-5 is better than average here, but you'll still see sessions where it describes a file edit instead of making it. MiniMax M2.5 trained explicitly on agentic tool use and shows stronger tool-call compliance than earlier open models. With Claude Code, the harness is tuned specifically to Anthropic's tool-calling format, failures are rare and predictable. Second: context window behavior. Models claim context sizes but degrade differently at long contexts. In sessions involving large codebases (100K+ tokens of context), GLM-5 maintains coherence reasonably well, but you'll occasionally see it contradict instructions from earlier in the session. MiniMax M2.5 was specifically trained to preserve reasoning between turns, which helps, but the provider recommends passing back reasoning_details explicitly to avoid degradation. Claude Sonnet 4.6 with prompt caching handles large contexts more consistently. Third: provider instability. OpenRouter is a routing layer. When the underlying provider (Z.AI in GLM's case) has an outage or rate limit, your session dies mid-task. Claude Code's infrastructure is more stable for sustained agentic runs. Mitigation: configure fallback models in OpenCode's config, always keep session checkpoints, and treat OpenCode runs like async jobs, never block synchronous work on them."

  - q: "Should I use Claude Code or OpenCode for a new project in 2026?"
    a: "The question is the wrong frame, most production setups end up using both. Here's the decision rule I actually use. Start a new project on Claude Code if: you have a hard deadline in the next 4 weeks, the codebase is complex or legacy, or you need frontier model quality for architecture decisions. The time you save on QA and retries pays for the cost premium. Migrate to OpenCode + OpenRouter once: the codebase is well-understood by the AI (AGENTS.md is solid), the tasks are well-specified and repeatable, and you've profiled which task types GLM handles reliably. That inflection point typically arrives 4–8 weeks into a project. This is the Python vs MATLAB pattern: MATLAB gave you integrated tooling, faster early results, and corporate support. Python gave you community, cost freedom, and composability. Almost everyone who needed to scale eventually moved to Python, not because MATLAB was bad, but because lock-in has compounding costs. Claude Code is excellent. But betting your entire workflow on Anthropic's pricing decisions is a risk that compounds as your team grows."

  - q: "What is the actual cost of Claude Code at Max plan for a developer team?"
    a: "Anthropic's own data shows Claude Code averages $6/developer/day, with 90% of users under $12/day. That's $130–$260/developer/month at the median to 90th percentile. The Max $100/month plan covers moderate professional use. Heavy agentic workflows, multi-file refactors, parallel agent sessions, extended context, regularly hit $200–500+/month per developer on pure API billing. The Max $200/month (20x) plan exists precisely because heavy users burn through the $100 allocation. For a 5-person engineering team all running Claude Code as their primary tool, budget $500–$1,500/month conservatively. The ROI is real: even at $1,500/month across 5 developers, one day of saved debugging time per week per developer likely covers the cost. The risk is the spend is opaque until you build tracking. Use the /cost command in Claude Code and implement a usage dashboard before it becomes a surprise line item."

  - q: "Is OpenCode stable enough for production developer workflows in 2026?"
    a: "Stable enough for daily use, not reliable enough to bet mission-critical deadlines on. OpenCode has 95K+ GitHub stars and 2.5M monthly active developers, it's not a toy. The CLI is solid. The TUI is responsive. Multi-session support and LSP integration work. The reliability gaps are at the edges: the VSCode extension lags the CLI by 1–2 releases, the OpenRouter integration has known provider-filtering bugs, and when upstream model providers change APIs, OpenCode sessions can fail mid-task with cryptic errors. Claude Code's failure modes are more predictable, you hit rate limits, not silent breakages. My honest assessment: OpenCode is appropriate for professional workflows with one caveat, build a fallback into your process. For tasks longer than 30 minutes, checkpoint your work. For tasks that block your team, use Claude Code. For cost-optimized background tasks and greenfield work, OpenCode with GLM is excellent value."

  - q: "How does Claude Code's feature release pace compare to OpenCode?"
    a: "Claude Code shipped 176 updates in 2025, roughly one meaningful release every two days. Checkpoints, parallel agents, IDE extension, hooks, memory files, subagents, skills, most of the patterns OpenCode and the community built workarounds for, Anthropic eventually absorbed. OpenCode's advantage is it moves faster on model diversity: new models appear on OpenRouter within days of release and are immediately available in OpenCode. Claude Code only uses Anthropic models natively. The convergence risk for OpenCode: every feature Claude Code absorbs is one less reason to tolerate OpenCode's friction. The convergence risk for Claude Code: every cost increase is one more reason to move the workflow to an open stack. Neither tool is standing still."
---

## Claude Code vs OpenCode: The CTO Decision Nobody Talks About Honestly

Every few months I switch my primary coding setup. Not because I get bored. Because the cost math or the capability gap shifts enough to justify the friction.

I've run both Claude Code and OpenCode in production. Here's the honest breakdown, what each costs, where each breaks, and the one decision framework that determines which you should be running today.

---

## The Wrong Way to Frame This Decision

Most comparisons you'll read treat this as a features question. It's not. It's a **lock-in vs. flexibility** question with real cost consequences.

The better analogy: Python vs MATLAB for data science.

MATLAB gave you an integrated, polished environment. Good documentation, vendor support, consistent behavior. The price was a license, and the hidden price was that your entire workflow depended on one vendor's pricing and product decisions.

Python gave you fragmentation, inconsistency, and a learning curve. It also gave you composability, community models, and the ability to run your stack on $5/month in compute instead of $50k/year in licenses.

Almost everyone who needed to scale eventually moved to Python, not because MATLAB was bad, but because lock-in has compounding costs.

That's the Claude Code vs OpenCode decision. Claude Code is excellent. OpenCode is flexible. The question is which trade-off matters more at your current stage.

---

## What Claude Code Actually Costs at Scale

The headline price is $20/month (Pro). The real cost for professional use is different.

Claude Code Pro at $20/month includes access but will hit rate limits quickly in sustained coding sessions, roughly 40–80 hours per week of light use. For anyone running Claude Code as their primary development tool, that ceiling appears within minutes of an intensive session.

Anthropic's own data from the /cost command shows Claude Code averages $6 per developer per day, with 90% of users under $12/day. That's $130–$260/developer/month at the median to 90th percentile on pure API billing.

The subscription tiers exist because API billing has no ceiling:

- **Pro** ($20/month): Sonnet access, fine for occasional use
- **Max 5x** ($100/month): The professional tier, 5x usage limits, Opus access
- **Max 20x** ($200/month): Heavy users, parallel agent sessions

One developer running Claude Code as their primary tool across eight months reported their busiest single day hit 8,930 messages across 9 sessions with 2,169 tool calls, and over 90% of all tokens were cache reads, meaning Claude Code is constantly caching codebase context.

At the higher end of usage, 15 agentic tasks per day at 200K tokens per task, costs rise to $800+ per developer per month on direct API billing.

For a 5-person team all running Claude Code seriously: budget $500–$1,500/month. The ROI is real, one avoided day of debugging per week per developer covers it. But the spend is opaque until you measure it.

**Build a cost dashboard before Claude Code becomes a surprise in your P&L.**

---

## The "Claude Code With Other Models" Trap

There's a workaround that's become popular: keep Claude Code as your harness, but point it at non-Anthropic models via environment variables. Set ANTHROPIC_BASE_URL to OpenRouter's endpoint, swap in GLM-5 or MiniMax via ANTHROPIC_MODEL, and you get Claude Code's familiar interface running cheaper models underneath.

It works. Until it doesn't.

The problem is Claude Code's harness was built and tested against Claude models specifically. The tool-calling format, the context compression logic, the prompt structure that drives Plan mode and subagents, all of it is tuned for how Claude responds. When you swap in a different model, you're running an incompatible engine in a chassis designed for something else. You get degraded tool-call reliability, unexpected compaction failures, and behavior that's hard to debug because the errors look like model errors but are actually harness-model mismatch.

Community guides treat this as a neat cost hack. In practice it adds a layer of unpredictability on top of the QA overhead you already have with cheaper models.

If you've decided to go with alternative models, go all the way. Use OpenCode. It was built from the ground up as a provider-agnostic harness. The abstraction layer is the product, not an afterthought. The tool-calling interface, the session management, the config routing, all of it assumes you'll be swapping models. That's the architecture difference that matters.

Claude Code with a swapped model gives you the worst of both: a harness not designed for your model, and a model not tuned for the harness. OpenCode with the right model routing gives you a stack that's honest about what it is.

## What Your Engineers Need to Get Started

Everything here is free to adopt. OpenCode is open source with no license cost. OpenRouter has no subscription, you pay per token, starting from zero. Your engineers are almost certainly already using npm, brew, or VSCode. The installation is a single command. The config is a JSON file.

There is no procurement process. No vendor negotiation. No onboarding contract.

What you do need: an OpenRouter account and API key, and a decision on which models to route to which agents. That's the work. The rest your engineers will figure out in an afternoon.

One known friction point worth flagging to your team: the VSCode extension currently has a bug where OpenRouter models don't appear in the model selector, even when correctly configured in the CLI. The workaround is using the terminal or desktop app directly. It's tracked in the OpenCode repo and will be patched. Not a blocker, but worth setting expectations before someone wastes time on it.

---

## Why GLM-5 and MiniMax M2.5 Cut Costs 55–70% (With Caveats)

The pricing differential is real. Here's the math:

| Model | Input ($/1M tokens) | Output ($/1M tokens) |
|---|---|---|
| Claude Sonnet 4.6 | $3.00 | $15.00 |
| Claude Opus 4.6 | $5.00 | $25.00 |
| GLM-5 (via Z.AI) | $0.80 | $2.56 |
| MiniMax M2.5 | $0.30 | $1.20 |
| GLM-4.5 Air | Free (quota) | Free (quota) |

*Prices as of March 2026. GLM via OpenRouter may carry a small routing markup (~10%).*

For a typical agentic coding session, a 10-step loop consuming 150K tokens at a 40/60 input/output split:

- **Sonnet 4.6**: (60K × $3 + 90K × $15) / 1M = **$1.53**
- **GLM-5**: (60K × $0.80 + 90K × $2.56) / 1M = **$0.28**
- **MiniMax M2.5**: (60K × $0.30 + 90K × $1.20) / 1M = **$0.13**

That's an 82% reduction for GLM-5 and a 91% reduction for MiniMax M2.5 per session on paper. Both carry 200K+ context windows. MiniMax M2.5 also includes automatic caching at no extra configuration, which further cuts costs on repeated context in long sessions.

Scale that to a realistic feature development cycle. And to be clear about scope: this is a **small feature**. One endpoint, one UI component, one test file. Not a subsystem refactor. Not a multi-service integration.

Base token budget for that small feature, one pass:

- 1.5M input tokens: codebase context loaded repeatedly across iterations
- 200K output tokens: code written
- 100K output tokens: QA run
- 10K output tokens: documentation

Total base: 1.5M input / 310K output.

Now apply the real-world QA multiplier. Sonnet gets it right in one pass most of the time. GLM-5 needs roughly 3 QA cycles on average before the output is production-ready. MiniMax M2.5 needs closer to 5. Each QA cycle re-sends context and generates output, so the token totals multiply accordingly.

| Model | QA cycles | Total input | Total output | Total cost | vs Sonnet |
|---|---|---|---|---|---|
| Sonnet 4.6 | 1x | 1.5M | 310K | **$9.15** | baseline |
| GLM-5 | 3x | 4.5M | 930K | **$5.98** | 35% cheaper |
| MiniMax M2.5 | 5x | 7.5M | 1.55M | **$4.11** | 55% cheaper |

*QA cycle = re-send full context + generate fix + re-run test output. March 2026 OpenRouter pricing.*

The savings survive. But the gap is much smaller than the raw per-token math suggests. On a small feature, Sonnet costs $9.15, GLM-5 costs $5.98, MiniMax M2.5 costs $4.11.

Now imagine this is a medium feature, 3x the token budget. Or a day where a developer ships 5 features. The multipliers compound fast.

Across 10 small features a month per developer, that's $91 vs $60 vs $41. Across a team of 5, it's $455 vs $300 vs $205. The savings are real but modest at this scale. Where the math gets serious is at volume: 50+ features a month, large context windows, or multi-agent pipelines where each agent runs independent sessions. That's where the 5x QA overhead of MiniMax M2.5 starts eating into the cost advantage, and GLM-5's balance of quality and price makes more sense as the default.

In practice, the savings are lower: **55–65% overall**. Why?

GLM-5 and MiniMax M2.5 both require more iterations on ambiguous tasks. Vague specs generate extra tool calls. More output tokens close the gap. The models differ on task fit: GLM-5 excels at complex reasoning and long-horizon agent workflows. MiniMax M2.5 scored 80.2% on SWE-Bench Verified and was trained across 200,000+ real-world environments, making it strong for structured, well-specified implementation work. For debugging complex state bugs across a large codebase, Claude's reasoning quality still means fewer retries, which partially offsets the cost difference.

MiniMax M2.5 also trained to decompose tasks before writing code, thinking and planning like an architect. In practice that reduces wasted token churn on poorly-framed prompts.

**The routing strategy that works in practice:**

```json
// Route by task type
{
  "models": {
    "default": "z-ai/glm-5",              // complex reasoning, agent tasks
    "volume": "minimax/minimax-m2.5",       // high-volume greenfield, well-specified work
    "complex": "anthropic/claude-sonnet-4-6", // debugging, architecture
    "simple": "zhipuai/glm-4.5-air:free"   // grep, file reads, trivial edits
  }
}
```

Don't use one model for everything. Route by task complexity. That's where the real cost efficiency comes from.

---

## The QA Problem Nobody Mentions

Here's what OpenCode blog posts skip: **when you leave Claude's harness, your QA burden increases.**

Three failure modes I've hit repeatedly:

**1. Tool-calling inconsistency**

Models differ on how reliably they invoke tools versus narrating what they'd do. GLM-5 is better than most alternatives, but you'll still see sessions where it writes: *"I would now edit the auth handler to..."* instead of actually editing it. MiniMax M2.5 shows stronger tool-call compliance in practice, trained explicitly on agentic tool use across diverse scaffolding environments. Claude Code's harness is tuned specifically for Anthropic's tool-calling format. Failures are rare and predictable.

**2. Long-context degradation**

In sessions involving large codebases, 100K+ tokens of loaded context, GLM-5 maintains coherence reasonably well for reasoning-heavy tasks. MiniMax M2.5 was specifically trained to preserve reasoning between turns, but the provider recommends passing back reasoning_details in each call to avoid degradation in long sessions. OpenCode is building toward a Workspaces feature that would persist context even when you close your laptop, something Claude Code's simpler CLI design can't easily support, but it's not shipped yet.

Claude Sonnet 4.6 with prompt caching handles large contexts more consistently. At $3/M input, the cached reads cost $0.30/M, almost free once the cache is warm.

**3. Provider instability**

OpenRouter is a routing layer. When Z.AI has rate limits or an outage, your session dies mid-task. One developer reported burning through as much OpenRouter credit in an hour as they had in the previous 11 months when running real agentic tasks, the token consumption at coding agent scale is dramatically higher than interactive chat.

Claude Code's infrastructure handles sustained agentic runs more predictably. You hit rate limits, not silent breakages.

**The mitigation stack for production OpenCode usage:**

- Configure fallback models in config.json (GLM primary → Sonnet fallback)
- Always enable OpenCode's session checkpoint/undo feature before long runs
- Treat OpenCode sessions as async jobs, never block synchronous work on them
- Run a /cost check at the start of each session, not the end

---

## Claude Code's Feature Velocity Advantage

This is real and matters for your decision timeline.

Claude Code shipped 176 updates in 2025, with key milestones including CLAUDE.md memory files, Plan mode, Subagents, /context command, Skills, and Opus 4.5 integration. That's roughly a meaningful release every two days.

The Code 2.0 release in September 2025 introduced automatic checkpoints, an IDE extension, parallel agents, and automation hooks.

What this means in practice: features the community builds workarounds for in OpenCode tend to get absorbed into Claude Code within 3–6 months. OpenCode's AGENTS.md concept maps to Claude Code's CLAUDE.md. OpenCode's multi-session support maps to Claude Code's parallel agents. OpenCode's plan/build mode separation maps to Claude Code's Plan mode.

Claude Code also ships security patches faster. Two CVEs were found in early 2026, one allowing arbitrary code execution through untrusted project hooks (CVSS 8.7), another allowing API key exfiltration from crafted repositories (CVSS 5.3). Both were patched in current versions.

OpenCode's advantage on feature velocity: **model diversity**. New models appear on OpenRouter within days of release and are immediately usable. Claude Code's model surface is only Anthropic's lineup.

The convergence risk: every feature Claude Code ships is one less reason to tolerate OpenCode's friction. Every Anthropic price increase is one more reason to migrate.

---

## The Decision Framework

This is how I actually make the call, not how I'd explain it to an audience:

**Use Claude Code when:**
- You have a deadline in the next 4 weeks
- The codebase is complex, legacy, or poorly documented
- The task is debugging, not greenfield
- The model needs strong reasoning across multiple files simultaneously
- You need the latest features (code security scanning, 1M context window, agent teams)

**Use OpenCode + GLM when:**
- The codebase is well-understood and you've built a solid AGENTS.md
- Tasks are well-specified and repeatable (UI components, CRUD endpoints, test generation)
- You're optimizing for cost at scale across multiple projects
- You're running background/async coding tasks with no time pressure
- You want model flexibility for experimentation

**The hybrid that works:**

Start new projects on Claude Code. Build context. Write AGENTS.md properly. Once the codebase is understood and the task types are profiled, typically 4–8 weeks in, migrate routine tasks to OpenCode + GLM. Keep Claude Code for architecture sessions and debugging.

I do this on every project now. Not because either tool is better in isolation, but because the combination optimizes both cost and quality.

---

## The Python vs MATLAB Lesson, Applied

In 2010, MATLAB was the correct choice for production scientific computing. It was integrated, supported, and fast to get started. Python was the scrappy alternative with better economics and worse UX.

By 2018, the question was "why are you still using MATLAB?"

The trajectory isn't identical, Anthropic is not MathWorks, and the AI tooling space moves 10x faster. But the pattern is the same: **integrated proprietary tooling dominates early, flexible open tooling wins at scale**.

Claude Code has massive momentum, since being released in May 2025, it became the most-used AI coding tool among surveyed developers, jumping from nowhere to number one by February 2026.

That's not a reason to avoid it. It's a reason to use it deliberately, with a clear-eyed view of where OpenCode + OpenRouter provides an exit ramp when the cost math changes.

Lock-in is most expensive when you don't see it coming.

---

## OpenCode Runs Claude Models Too

One thing that gets buried in the tool comparisons: OpenCode works with Anthropic models just fine. Add your Anthropic API key, select claude-sonnet-4-6 or claude-opus-4-6, and OpenCode behaves like Claude Code, except you own the harness and pay per token with no subscription overhead.

For developers already paying $100/month for Claude Max, that math looks different. The subscription exists because it caps unpredictable API spend. If you're disciplined about session length and you've built cost tracking, direct API via OpenCode can come out cheaper, because you're not paying for headroom you don't use.

The setup:

```bash
opencode auth login
# Select "Anthropic" and enter your API key
# Then inside OpenCode:
/models
# Select anthropic/claude-sonnet-4-6 or anthropic/claude-opus-4-6
```

This is the bridge for teams not ready to switch models. Start on Anthropic models in OpenCode. Get familiar with the harness. Then introduce GLM-5 or MiniMax M2.5 for specific agents once you've built confidence in the routing. The migration is model-by-model, not a full tool switch.

---

## Advanced: Multi-Agent OpenCode With Model-Per-Role Routing

This is where OpenCode separates from everything else. Not just swapping models, but assigning different models to different agents based on what each agent actually does.

The architecture is simple in principle: a senior planner agent orchestrates the session, delegates to specialist subagents, and routes QA and debug work to models chosen for those specific tasks. Expensive, high-quality models only run where quality actually matters. Cheap, fast models run the mechanical work.

Here's a production-ready config. This is the real pattern, with two tiers of OpenRouter models replacing local GPU inference, and Claude Sonnet reserved for the tasks where it earns its cost.

```json
{
  "$schema": "https://opencode.ai/config.json",

  "enabled_providers": ["openrouter"],

  "provider": {
    "openrouter": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "OpenRouter",
      "options": {
        "baseURL": "https://openrouter.ai/api/v1",
        "apiKey": "YOUR_OPENROUTER_API_KEY"
      }
    }
  },

  "small_model": "openrouter/qwen/qwen3-30b-a3b",
  "default_agent": "plan",
  "autoupdate": true,

  "compaction": {
    "auto": true,
    "prune": true,
    "reserved": 10000
  },

  "agent": {

    "plan": {
      "mode": "primary",
      "description": "Senior architect and orchestrator. Reads requests, produces implementation plans, delegates all work to specialist subagents, drives two-tier QA loops, and drives features to completion. Never writes application code.",
      "model": "openrouter/anthropic/claude-sonnet-4-6",
      "temperature": 0.3
    },

    "build": {
      "mode": "subagent",
      "description": "API and infrastructure developer. Routes, models, middleware, auth, Docker config. Called by plan agent. Reports completion back.",
      "model": "openrouter/z-ai/glm-5",
      "temperature": 0.2
    },

    "backend": {
      "mode": "subagent",
      "description": "AI and orchestration specialist. LLM flows, pipeline logic, prompt engineering. Called by plan agent.",
      "model": "openrouter/z-ai/glm-5",
      "temperature": 0.2
    },

    "frontend": {
      "mode": "subagent",
      "description": "Frontend specialist. React, TypeScript, Tailwind, component libraries. Called by plan agent.",
      "model": "openrouter/z-ai/glm-5",
      "temperature": 0.3
    },

    "qa-fast": {
      "mode": "subagent",
      "description": "First-pass mechanical QA. Runs after every agent step. Catches missing guards, type errors, missing filters, docs not updated. Fast and cheap.",
      "model": "openrouter/minimax/minimax-m2.5",
      "temperature": 0.1
    },

    "qa-deep": {
      "mode": "subagent",
      "description": "Senior QA reviewer. Runs at feature milestones only, after all steps pass qa-fast. Catches architectural violations, security traces, cross-agent consistency, business logic gaps.",
      "model": "openrouter/anthropic/claude-opus-4-6",
      "temperature": 0.1
    },

    "debug": {
      "mode": "subagent",
      "description": "Root cause analyst. Called after 2 failed QA cycles. Finds actual cause, not symptomatic fixes. Reads full call chain before forming hypothesis.",
      "model": "openrouter/z-ai/glm-5",
      "temperature": 0.1
    },

    "explore": {
      "mode": "subagent",
      "description": "Read-only codebase navigator. Finds files, traces imports, maps data flows. Cannot edit files.",
      "model": "openrouter/qwen/qwen3-30b-a3b",
      "temperature": 0
    },

    "docs": {
      "mode": "subagent",
      "description": "Documentation specialist. READMEs, API docs, runbooks, inline docstrings. Reads actual code before writing.",
      "model": "openrouter/minimax/minimax-m2.5",
      "temperature": 0.4
    },

    "knowledge": {
      "mode": "subagent",
      "description": "Library research specialist. Version-specific API answers from official docs and changelogs. Never guesses. Cannot edit files.",
      "model": "openrouter/deepseek/deepseek-v3.2",
      "temperature": 0.1
    }

  }
}
```

### Why each agent gets its model

The config above isn't arbitrary. Each model assignment follows a specific reason.

**plan (Claude Sonnet 4.6):** The planner is the most important agent in the stack. Every other agent works from the plan it produces. A bad plan cascades into bad builds, failed QA, and debug cycles that cost more than the Sonnet tokens ever would. Sonnet 4.6 at $3/M input is the right model here: strong multi-step reasoning, consistent instruction-following, and the orchestration quality to delegate clearly. The planner's output volume is low relative to the implementation agents, so the cost is contained.

**build / backend / frontend (GLM-5):** Implementation agents write the actual code. Boilerplate, CRUD routes, UI components, infrastructure config, LLM pipeline logic. These tasks need more than mechanical generation, they need to understand context, handle edge cases, and produce code that passes QA in fewer cycles. GLM-5 at $0.80/M input and $2.56/M output is the right balance: strong enough to get complex implementation right on the first or second attempt, cheap enough to run across all three implementation roles without blowing the budget.

**qa-fast / docs (MiniMax M2.5):** QA-fast runs after every single agent step, so it accumulates the highest call volume in the stack. MiniMax M2.5 was trained on structured productivity tasks and decomposes before acting, which makes it reliable for checklist-style QA: missing auth guards, type errors, undocumented changes. At $0.30/M input and $1.20/M output it's the right cost tier for high-frequency mechanical work. Docs follow the same logic: structured, well-specified output that does not need frontier reasoning.

**qa-deep (Claude Opus 4.6):** Deep QA runs once per milestone, after every implementation step has passed qa-fast. This is the one place you want the best reasoning in the stack. Opus 4.6 catches architectural violations, full security traces, cross-agent consistency gaps, and business logic errors that GLM-5 would miss at this level of scrutiny. It runs rarely, so the $5/M input cost stays contained. Think of it as a senior engineer doing a final review before the feature ships.

**debug (GLM-5):** Debug runs only after qa-fast has failed twice. The problem is non-obvious, but not necessarily architectural. GLM-5's long-horizon reasoning handles root cause analysis well at this level without needing Opus.

**explore (Qwen3-30B-A3B):** Read-only navigation, file tracing, import mapping. No code generation, no state, just fast and cheap traversal. Qwen3-30B-A3B at around $0.10/M input is the right model for a task that produces zero output that ships.

**knowledge (DeepSeek V3.2):** Library research, version-specific API lookups, changelog reading. DeepSeek V3.2 is strong on factual technical recall and costs around $0.14/M input. No code editing, no state, just accurate lookup.

### What this actually costs

Running this full multi-agent stack on the same small feature from earlier, 1.5M input / 310K output total across all agents, with the model distribution above, blended cost lands around **$2.50 to $3.50 per feature**. The planner runs on Sonnet but its token volume is low. Build, backend, and frontend run on GLM-5, which is where most tokens go. QA-fast and docs run on MiniMax at commodity rates.

Compare that to running Sonnet across every agent: $9.15 for the same feature. This stack costs 62–73% less, and the implementation quality is higher than the Qwen-based config because GLM-5 writes better code than Qwen3-30B on complex tasks. The tradeoff is explicit: spend a little more on the agents that ship code, save everywhere else.

---

## The Practical Checklist

Before your next project starts:

- [ ] Measure your current Claude Code spend with /cost (if you haven't, do it now)
- [ ] Profile which task types in your workflow are "well-specified" vs. "require reasoning"
- [ ] Set up OpenCode with OpenRouter, add GLM-5 and MiniMax M2.5, test both against 10 real tasks
- [ ] Compare iteration count and output quality vs. Sonnet on those same tasks
- [ ] Build a config.json with model routing by task type
- [ ] Set a budget alert on OpenRouter before adding credits (it goes fast at agent scale)
- [ ] Check if VSCode extension issue #6066 is resolved before wiring your editor through it

The answer to "which tool should I use" is almost always "both, with routing logic." The answer to "which should I start with" is Claude Code. The answer to "when do I add OpenCode" is when you've measured where the cost is going and have 10 tasks that GLM handles reliably.

---

**About the Author**

Netanel Eliav builds production AI systems — agentic workflows, RAG pipelines, LLM infrastructure, and evaluation frameworks. He advises engineering teams on agentic architecture and runs projects including Data, Analytics and AI related domains. London-based, global AI leader.

[LinkedIn](https://linkedin.com/in/inetanel) · [GitHub](https://github.com/inetanel) · [inetanel.com](https://inetanel.com)