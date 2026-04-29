---
title: "The Architect: Why I Built an Autonomous Dev Layer on Top of Claude Code"
description: "A full breakdown of why bare AI coding CLIs break at scale, and every mechanism inside The Architect that fixes it — from circuit breaker to ARCHITECT.md."
date: "28/04/2026"
author: "Netanel Eliav"
series: "Production AI"
tags: ["agentic systems", "autonomous development", "Claude Code", "OpenCode", "local LLM", "open source", "London"]
slug: "the-architect-autonomous-dev-layer-why-how"
featured_image: "https://raw.githubusercontent.com/iNetanel/blog/main/assets/images/the_architect.jpg"
featured: true

faqs:
  - q: "What problem does The Architect solve that Claude Code doesn't?"
    a: "Claude Code solves the coding problem. It writes good code. The Architect solves the orchestration problem — making sure the coding actually finishes, correctly, without you watching. When you use Claude Code directly on a multi-task goal, you become the orchestration layer: you catch stuck agents, verify completions aren't hallucinated, retry with adjusted context, carry state between attempts, and check whether the output actually passes tests. For a single task, that overhead is 10 minutes. For a goal that decomposes into 10+ tasks, it's 3-4 hours of active supervision. The Architect replaces that entire supervisory loop with a system that plans, executes, detects failure, retries intelligently, reviews quality, and persists memory — unattended."

  - q: "How does The Architect handle agents that claim they're done when they aren't?"
    a: "Multi-signal completion detection. No single signal is trusted in isolation. The Architect requires corroboration between four independent signals before declaring a task done: the agent outputs a promise tag (<promise>TXX_COMPLETE</promise>), the task is marked Done in PROGRESS.md, the provider CLI exits with code 0, and the agent text contains a progress signal ('all tests pass', 'task is done'). Two or more signals must agree. Exit code 0 alone is not enough — providers exit 0 on timeout. 'Task complete' in text alone is not enough — agents say this while tests are failing. And if the agent says 'I'm stuck' or 'I can't proceed' anywhere in its output, that overrides any completion claim entirely, because a stuck agent that claims completion is hallucinating."

  - q: "What is the circuit breaker and what does it actually do?"
    a: "It's a failure pattern detector inspired by electrical circuit breakers — when a physical circuit draws too much current, the breaker trips to protect the system. The Architect does the same thing across retry attempts. Three counters: no-progress (agent ran but wrote zero files, three times in a row), same-error (same bash error fingerprint across attempts — fingerprinting strips file paths and line numbers so it catches the same logical error even when the text differs), and token decline (agent using less than 40% of attempt 1's tokens by attempt 3, meaning it's giving up earlier each retry). When any counter trips, the circuit opens and chooses a recovery action: WAIT (let model rotation handle it), REPLAN (rewrite just the failing task using the architect agent), or COOLDOWN_WAIT (rate limit detected — pause 1 hour without consuming a retry slot). Circuit state is persisted to disk, so if the process is killed mid-cooldown, the remaining wait time is preserved on restart."

  - q: "How does The Architect make local GPU models like Gemma 4 or Qwen 3.6 viable for real development work?"
    a: "The context window problem is misunderstood. Gemma 4 or Qwen 3.6 on a local machine might have 40k-128k tokens on paper. But during a real coding task, that window fills fast: base instruction, files the agent reads for context, the file it's editing, test output, tool call results, and the response it's generating. For any non-trivial feature touching multiple files — especially across a monorepo — you're at 20k-40k tokens of actual working content before the agent has written a line. Cross-repo work is essentially impossible at this scale. The standard approach is to give the model the whole codebase and let it figure out what's relevant. That fills the window with noise and degrades output quality. The Architect inverts this: the planner (running on a frontier model with full context) decomposes the goal into task files scoped precisely for the execution model's reliable range. Each task contains only the specific files and instructions that task needs — typically 15k-25k tokens. The local model executes tasks one at a time in clean context windows, never overwhelmed. This is the mixed-model pattern: frontier model for planning and retrospective, local model for execution. A model with a 30k context window can do real production work this way."

  - q: "What does ARCHITECT.md contain and why does it matter across sessions?"
    a: "ARCHITECT.md is the project's institutional memory — the architectural decisions, known constraints, lessons learned, best practices, and project structure that accumulate across every session. Three agents read and update it: the planner reads it first as highest-priority context before creating task files; the build agent reads it at the start of every task and is explicitly instructed to append new constraints or lessons it discovers during execution; the reviewer reads it after execution and updates it with cross-task quality patterns. The project structure section (repo type, detected frameworks, component descriptions, test/lint commands) is rewritten fresh on every planning session. All other sections are append-only. After 50+ builds on The Architect itself, the file contained 23 architectural decisions and 11 lessons — none written manually. The agents wrote them while building. This is what prevents session 10 from repeating session 3's mistake."

  - q: "Can I run The Architect overnight safely? What prevents a runaway cost disaster?"
    a: "Several mechanisms working together. The token_budget_per_hour config option caps hourly API spend — the run pauses when the rolling hourly budget is hit, waits for the window to reset, and resumes. Cooldown detection pauses automatically when rate limits hit (HTTP 429, 'quota exceeded', 'out of extra usage', 'credit balance too low') without consuming retry slots. The circuit breaker trips before a stuck agent burns through all retries on a known-broken path. Circuit state, cooldown timers, and task progress are all persisted to disk — if the process is killed mid-run, restarting picks up from exactly where it stopped. A lock file prevents accidental concurrent runs from corrupting shared state. And the premature exit guard prevents accidental re-planning of an already-complete project if you restart the next morning."

  - q: "How does the retrospective reviewer work and what does it catch?"
    a: "After execution completes, a separate reviewer agent runs — it reads PROGRESS.md (to understand what was done and what failed), all task files (to understand what was planned), and the actual code written, then runs your test suite. If it finds quality issues — failing tests, missing edge cases, inconsistent implementations, incomplete requirements — it creates R-prefixed fix-up tasks (R01, R02, ...) that run through the same execution pipeline with the same retry and circuit breaker protection. In persistent mode, two retrospective rounds run: the second reviews the first round's fixes. The reviewer is explicitly not allowed to modify existing task files or rewrite PROGRESS.md. It writes fix-up tasks or writes nothing — silence means the work is clean. In a 12-task run, I've seen the reviewer catch a missing test edge case in T07 and an incomplete requirement in T11 that the build agent had declared complete."

  - q: "What's the difference between 'simple', 'standard', and 'complex' scope?"
    a: "Scope controls how the planner decomposes your goal into task files. At 'simple' scope, the same goal becomes 15-20 narrow task files — each one a small, focused operation (one function, one test, one file). At 'complex' scope, the same goal becomes 3-5 broad task files — each one a full subsystem or cross-cutting concern. 'Standard' sits in between. The number of tasks isn't fixed — it emerges from goal size divided by scope. The right scope depends on your model: weak or local models need 'simple' because each task must fit comfortably inside their reliable context range. Frontier models (Claude Opus, GPT-4o, Gemini 2.5 Pro) can handle 'complex' because they maintain coherence over large context. Getting scope wrong is the most common cause of failed runs — too broad for the model, and it loses track halfway through the task."
---

# The Architect: Why I Built an Autonomous Dev Layer on Top of Claude Code

I spent three months watching AI coding agents fail the same way.

Not fail catastrophically. Fail quietly. An agent that writes two files, declares success, exits code 0, and leaves you with a broken test suite. An agent that rewrites the same function three times across three retries because nobody gave it the context from attempt 1. An agent that stops making progress at 11pm and sits there, token-burning, while you're asleep.

I was building agentic systems for production — the kind where you can't babysit every run. The tools were good at writing code. They were bad at *finishing the job*.

So I built The Architect. It's an open-source autonomous development lifecycle layer that wraps Claude Code or OpenCode and adds everything they're missing: planning, completion verification, retry intelligence, quality review, and persistent memory. It's on PyPI. It shipped itself. And I'm going to explain every mechanism in it, because the problems it solves are real and the solutions aren't obvious.

**[github.com/iNetanel/the-architect](https://github.com/iNetanel/the-architect)**

---

## The Problem: You Are the Orchestration Layer

Here's what using Claude Code directly looks like on a multi-task goal.

You write a goal. You start Claude Code. The agent runs. Twenty minutes in, it's rewriting the same function for the third time — no explanation why. You kill it. You retry with adjusted context. It exits code 0. You check the tests. They fail. The completion was a hallucination. You re-prompt, manually carrying the relevant context from the failed attempt. You wait. You check again.

You repeat this for each task. By task 7, you're tired. Task 9 ships with a known edge case because it's 1am and you need to sleep.

**Active supervision: 3-4 hours for a 10-task goal.** Most of that time is watching, not thinking. You're not writing architecture decisions. You're catching stuck loops.

This is the orchestration problem. Claude Code solves the coding problem. Nobody solved the orchestration problem.

The gap shows up in four specific places:

**1. Completion isn't verified.** The agent says "task complete." It means nothing. Exit code 0 means nothing. The agent hallucinates completion routinely — writes partial output, declares done, moves on. You have to check.

**2. Retries have no memory.** When a task fails and you retry, the new attempt starts cold. It doesn't know what attempt 1 tried, what broke, what files were written. It repeats the same mistake.

**3. There's no stuck detection.** An agent that's genuinely blocked will keep trying — and keep burning tokens — until you manually intervene. There's no mechanism that says "this is the same failure three times, stop and do something different."

**4. Context resets every session.** Every time you start a new session, you re-explain the project. The constraints. The decisions already made. The lessons from last week's failed run. There's no memory.

### The Fake Solutions

Before building The Architect, I tried the obvious fixes. They don't work at scale.

**Better prompts.** You spend two hours crafting a perfect system prompt. It helps — for one session. Then you change the goal, the codebase evolves, or the model gets a new version. Your carefully tuned prompt is now partially wrong. You're back to babysitting because prompts don't adapt to the state of your project. And a better prompt still can't detect that the agent is stuck, can't carry context between retries, can't verify completion, and resets completely next session. Prompting is not an orchestration layer.

**More expensive models.** Claude Opus or GPT-4o is better than Sonnet at following complex instructions. That's real. But you're still supervising. A better model hallucinates completion less often — it doesn't eliminate it. It gets stuck less often — it doesn't detect when it is stuck and recover. It's more capable per token — but you're still paying for every retry, every stuck loop, every session restart where it re-reads your entire codebase from scratch. And for pure QA — running the retrospective review, verifying test output, catching edge cases the build agent missed — spending Opus-level pricing is expensive quality assurance that still requires your eyes on the output.

The real problem isn't model capability. **It's that you can't hand off control without losing control.** You can't walk away from an expensive model run because there's nothing to catch it when it goes wrong at 2am. You either pay attention or you pay for chaos.

The Architect is the handoff mechanism. You stay in control — you define the architecture, the goal, the scope. The Architect executes it and handles every failure mode that would otherwise require your intervention.

The Architect fixes all four root problems. Here's how each mechanism works.

---

## The Problem Nobody Talks About: Production Codebases

Everything above assumes you're building something new. Greenfield. A fresh repo with no users, no dependencies that can't change, no decisions baked into the code from 18 months of iteration.

Greenfield with AI agents is easy. You describe a goal, the agent builds it, you review the output. If it goes sideways, you delete the branch and start over. The cost of a wrong decision is low.

**Production codebases are completely different.** And this is where most AI coding tools silently fail.

In a real product — one with users, with data in production, with a team that has made hundreds of design decisions you didn't document — every agent decision compounds. The agent doesn't know that you chose that specific database schema because of a constraint that came from a compliance requirement in Q3. It doesn't know that the authentication module works the way it does because three different clients have different SSO configurations. It doesn't know that you intentionally left that abstraction loose because you're planning to replace it in the next sprint.

The agent sees the code. It doesn't see the *reasoning behind the code*.

So it optimises locally. It makes the change that looks right in isolation, and that change conflicts with something three layers deeper. It refactors a function to be cleaner, and breaks a subtle dependency you had on the previous behaviour. It adds a new pattern that's inconsistent with the existing patterns, because it didn't read enough of the codebase to know a pattern existed.

**And you can't review every agent decision anymore.** That's the whole point of autonomous execution — you're not watching every step. So by the time you look, the damage is already spread across 12 files and two retrospective rounds of fix-up tasks.

This isn't a theoretical concern. I've seen it happen. An agent adds a new API endpoint, correctly wired, correctly tested — and silently changes the error response format across the module to match the new endpoint's pattern. Technically better. But now every client that parses that error format is broken. The tests pass. The retrospective reviewer passes. The change ships.

### How The Architect Mitigates This

No tool eliminates this problem completely — that's honest. But The Architect has four specific mechanisms that reduce it substantially.

**ARCHITECT.md captures decisions before they're lost.** When the build agent discovers a constraint during execution — "this module uses a specific error format for legacy client compatibility" — it's explicitly instructed to write that to ARCHITECT.md under Known Constraints. The next planning session reads it. The next task file includes it. The next build agent sees it before touching that module. This is how institutional knowledge stops living exclusively in your head and starts living in the project.

**The planner runs on a frontier model with full context.** Your local Qwen doesn't plan. The architect agent — running on Claude Opus or Gemini 2.5 Pro — reads your ARCHITECT.md, your detected project structure, your dependency graph, and any additional context you provide via `--context`. It plans with the full picture before decomposing into tasks. This is the moment where cross-cutting concerns, existing patterns, and known constraints get baked into the task files themselves — as explicit instructions to the build agent, not as hope that the agent will figure it out.

**Scope keeps tasks isolated.** A task that touches three components in a monorepo is a task that's likely to produce unintended side effects. At `simple` scope, the planner creates tasks that are intentionally narrow — one component, one concern, one test surface. This doesn't eliminate cross-component risk, but it makes each change reviewable in isolation and reduces the blast radius of any single agent decision.

**You define the architecture, not the agent.** This is the hard constraint built into how The Architect works. You set the goal. You choose the scope. You can provide a PRD, a spec, design docs — any context that constrains what the agent should and shouldn't do. The agent plans and executes within that envelope. It doesn't invent architectural direction. If an agent decision during execution conflicts with an existing constraint in ARCHITECT.md, that constraint wins — the agent reads it before starting every task.

The real answer to "how do you use AI agents safely on a production codebase" is: **stay at architecture level, and make sure every agent decision is constrained by what you already know.** The Architect is built around that principle. You don't review every file change. You review the plan, you seed the constraints, and you let the mechanisms handle execution.

That's a different relationship with the agent than greenfield development. It requires more upfront work in ARCHITECT.md and `--context` files. It requires thinking carefully about scope. But it's the only way to use autonomous development on a codebase that has real users and real consequences.

---

## Mechanism 1: Autonomous Planning

The first job is decomposing your goal into executable task files.

You don't write the tasks. The architect agent does.

```bash
architect --plan --goal "add Stripe payment integration with webhooks and refund support"
```

The architect agent reads your goal, your project structure (auto-detected — more on this below), and any context files you've provided via `--context`. It produces numbered task files in the `tasks/` directory:

```
tasks/
├── T01_stripe_models.md
├── T02_payment_routes.md
├── T03_webhook_handler.md
├── T04_refund_logic.md
├── T05_tests_and_docs.md
└── INSTRUCTIONS.md
```

Each task file is a structured instruction: goal, context, sub-tasks. Self-contained. Not a vague directive — specific enough that the build agent can execute it without needing to ask questions.

You can also point it at a PRD, spec, or any document:

```bash
architect --plan --context PRD.md
architect --plan --context design/ --context SPEC.md
```

The architect agent extracts the goal automatically from `## Goal`, `## Objective`, or `## Requirements` sections. In CI, set `ARCHITECT_CONTEXT` as an environment variable. The planning reads the file, extracts the goal, and produces tasks without any interactive prompts.

### Scope: How Tasks Get Sized

At planning time, you choose a scope — `simple`, `standard`, or `complex`. This controls how the planner decomposes the goal.

At `simple` scope, a 10-feature goal becomes 15-20 narrow task files — each one a small, focused operation. One function, one test, one file.

At `complex` scope, the same goal becomes 3-5 broad task files — each one a full subsystem.

**Why scope matters for local GPU models.**

People talk about local model context windows as if 128k means you can work like you would with Claude Sonnet. You can't. The number is theoretical — what matters is the *reliable* range, which is much smaller once you account for what's actually in that context.

Take Gemma 4 or Qwen 3.6 running on a local machine. You might have 40k, 65k, maybe 128k tokens on paper. But think about what fills that window during a real coding task: the base instruction, the codebase files the agent reads for context, the file it's editing, the test output, previous tool call results, and the response it's generating. For any non-trivial feature touching multiple files — especially in a monorepo or cross-repo scenario — you're looking at 20k-40k tokens of actual working context before the agent has written a single line.

At that point, **128k isn't enough for cross-repo or multi-component work.** It's enough for isolated, focused, single-file tasks. That's not a limitation of the model — it's the physics of context windows on hardware that isn't a data centre.

The standard approach is to give the model the whole codebase and hope it figures out what's relevant. That's exactly wrong. It fills the window with noise and degrades output quality across the entire run.

**The Architect's approach is the opposite.** The planner, which runs on a frontier model with full context, decomposes the goal into task files scoped precisely for the execution model's reliable context range. Each task file contains only the specific files, constraints, and instructions that task needs. The execution model — your local Gemma 4 or Qwen — receives a clean, narrow instruction that fits inside 15k-25k tokens. It doesn't need to see the whole monorepo. It needs to see the three files it's modifying and the tests it has to pass.

This is the mixed-model pattern: **frontier model for planning and retrospective, local model for execution.**

```toml
[architect]
# Local GPU model for execution (via OpenCode + Ollama or LM Studio)
standalone_mode = "local/qwen3:6b"

# Frontier model for planning — set at run time via --architect-model
# architect --plan --architect-model openrouter/google/gemini-2.5-pro
```

The planner runs once, online, with full codebase context. It produces 15 narrow task files. Then your local Qwen executes them one at a time, each in a clean context window, never overwhelmed. The retrospective reviewer runs on the frontier model too — it needs to see the full picture to assess quality.

A model with a 30k context window can do real production work this way. Not synthetic benchmarks — actual feature development, actual test writing, actual refactoring. The scope system is what makes it possible.

### Project Structure Detection

Before creating task files, the architect agent needs to understand your project. The Architect detects this automatically:

- **Repo type:** single repo, monorepo, multi-repo, or untracked
- **Languages:** detected from signal files (`pyproject.toml` → Python, `package.json` → JavaScript/TypeScript, `Cargo.toml` → Rust, `go.mod` → Go, and more)
- **Frameworks:** Next.js, FastAPI, Django, Gin, Axum, React, Express, and more — detected from config files first, then dependency scanning
- **Component descriptions:** pulled from `pyproject.toml` description fields and `package.json` description fields
- **Key dependencies:** top 8 from each component's dependency list, with dev tooling filtered out
- **Test and lint commands:** auto-detected (`pytest tests/ -v --tb=short` when pytest is in deps, `npm test` when a test script exists, and so on)
- **Dependency graph:** parsed from `docker-compose.yml` `depends_on`, package workspace configs, and local path dependencies

All of this goes into the planning prompt and into `ARCHITECT.md`. The architect agent knows your codebase before it writes a single task.

---

## Mechanism 2: Execution with Live Streaming

Once tasks are planned, execution runs them one at a time:

```bash
architect
```

The runner discovers all pending task files, filters out anything already marked Done in `PROGRESS.md`, and executes them in order via the provider CLI — Claude Code or OpenCode, whichever you have configured.

Provider output streams directly to your terminal — no piping, no reformatting. With OpenCode, structured JSON events are parsed to render tool calls with filenames and commands (`→ write app/payments/models.py`, `$ pytest tests/ -v`). With Claude Code, plain text output is displayed as-is.

Between tasks, there's a configurable pause (default: 10 seconds) to let file writes flush before the next task reads them.

### The Execution Instruction

Every task gets an execution instruction composed of three parts:

1. **The execution protocol** — the rules the build agent must follow: how PROGRESS.md works, how to declare completion, what to update in ARCHITECT.md. A few hundred tokens. Minimal by design.

2. **ARCHITECT.md content** — the full accumulated project intelligence: permanent decisions, known constraints, lessons learned, best practices, and the enriched project structure. This replaces conversation history. The build agent reads architectural context from a file, not from a 40-turn chain.

3. **The task-specific instruction** — project root, pointers to PROGRESS.md and the task file, and retry context if this is a re-attempt.

The build agent is explicitly instructed to update ARCHITECT.md when it discovers new constraints or lessons during execution. Project intelligence accumulates automatically.

---

## Mechanism 3: Multi-Signal Completion Detection

This is where most AI coding tools fail. The Architect doesn't trust any single signal.

Four signals, in order of strength:

| Signal | How it works | Strength |
|--------|-------------|---------|
| **Promise tag** | Agent outputs `<promise>TXX_COMPLETE</promise>` in text | Strong |
| **PROGRESS.md** | Task marked `Done` in the progress file | Moderate |
| **Clean exit** | Provider CLI exited with code 0 | Weak |
| **Progress signal** | Text contains "all tests pass", "task is done" | Weak |

Decision rules:

- **2+ signals positive** → Done. Unambiguous.
- **Promise tag alone** → Done. Strong enough on its own.
- **PROGRESS.md alone** → Done with a warning logged.
- **Exit code alone** → Not done. Providers exit 0 on timeout.
- **Progress signal alone** → Not done. Agents say "tests pass" before running them.

And the override: **if the agent outputs any stuck signal anywhere in its text — "I'm stuck", "I can't proceed", "no clear path forward", "unable to resolve" — that overrides every other signal.** Two or more stuck patterns in one attempt triggers a stuck classification. A stuck agent claiming completion is hallucinating. This rule stops false completions dead.

---

## Mechanism 4: Retry with Context Carry

When a task fails completion detection, it retries — up to 3 attempts by default (30 in persistent mode), with a configurable pause between attempts.

The key feature is `carry_context`. On retry, the previous attempt's context is summarised and injected into the new instruction:

- Files written or edited in the previous attempt
- Files read
- Bash commands run
- Test failures extracted from pytest output

The build agent on attempt 2 knows exactly what attempt 1 tried. It doesn't repeat the same mistake. It starts from where the previous attempt stopped.

This summary is parsed from the attempt's log file on disk — not from memory. If the process is killed and restarted between attempts, the context is still available.

**Retry models** let you fall back to a different model on each attempt:

```toml
[architect]
retry_model_2 = "openrouter/google/gemini-2.5-pro"
retry_model_3 = "openrouter/anthropic/claude-opus-4.5"
```

Attempt 1 uses your default. Attempt 2 switches to the retry model. Attempt 3 escalates further. Different models have different strengths — a task that Claude Sonnet gets stuck on may complete cleanly on Gemini 2.5 Pro.

---

## Mechanism 5: The Circuit Breaker

Retry logic handles model failures. The circuit breaker handles something retries can't: **failure patterns**.

If an agent is failing the same way repeatedly, continuing to retry wastes tokens, burns budget, and sometimes makes things worse — confused context, partial writes, contradictory state. The circuit breaker detects these patterns and stops the loop before it becomes expensive.

Three counters, all persisted to disk:

### Counter 1: No-Progress

The agent ran, but wrote zero files. Three consecutive attempts with no file writes: the circuit trips.

This catches agents that are confused about what to do — they generate text, they read files, they run commands, but they don't make progress. Token burn with no output.

### Counter 2: Same-Error

The agent produced the same bash error fingerprint across multiple attempts.

Fingerprinting strips file paths (`/home/user/project/foo.py` → `<path>`) and line numbers (`:42` → `:<N>`), then normalises whitespace and lowercases. So `"Error in /a/b/c.py line 42"` and `"Error in /x/y/z.py line 99"` produce the **same fingerprint**.

This means the circuit breaker catches the same *logical* error — not just identical text. Three identical fingerprints in a row: the circuit trips. The agent is hitting the same wall with different syntax each time.

### Counter 3: Token Decline

If attempt 3 uses less than 40% of attempt 1's tokens, the agent is giving up earlier each retry. Combined with any elevated counter: circuit trips.

This catches demoralised agents — they've hit a failure, they don't know how to fix it, and each attempt they generate less and less output before halting.

### Circuit States and Recovery

```
CLOSED → (threshold hit) → OPEN → (cooldown period) → HALF_OPEN → (one test attempt) → CLOSED or OPEN
```

When the circuit opens, it chooses a recovery action based on what's available:

**WAIT** — retry models still available, or some file progress was made. Let model rotation continue.

**REPLAN** — all models exhausted, zero file progress ever made on this task. The architect agent rewrites just the failing task:

```
=== TARGETED TASK REPLAN ===

Task T03 has failed repeatedly. Circuit breaker opened.
Fix this ONE task only. Do NOT touch any other task files.

Original task content: [...]
What was tried and what failed: no_progress=3, same_error=2, last_error=[...]
Current PROGRESS.md: [...]

Either rewrite T03 with corrected assumptions,
or split T03 into two smaller tasks.
```

Replan is attempted once per task. If the replanned task also fails, the circuit moves to WAIT.

**COOLDOWN_WAIT** — rate limit detected. Pause for the required time (minimum 1 hour), no retry slot consumed, no circuit counter incremented. Resume automatically.

This is the mechanism that makes overnight runs safe. A stuck agent doesn't spin indefinitely — the circuit catches it, makes a decision, and either rotates or stops cleanly.

---

## Mechanism 6: Rate Limit and Cooldown Handling

During a long unattended run, hitting rate limits is inevitable. The Architect handles them without your intervention.

Detection patterns:

- HTTP 429 or 529 exit codes
- Text patterns: `"rate limit"`, `"too many requests"`, `"quota exceeded"`, `"out of extra usage"`, `"credit balance is too low"`, `"overloaded"`, `"temporarily unavailable"`, and more — covering both OpenCode and Claude Code output formats

When a rate limit is detected:

1. The run pauses
2. **No retry slot is consumed**
3. **No circuit breaker counter is incremented**
4. The cooldown duration is determined — minimum 1 hour, or longer if the provider's message suggests it
5. The run logs progress every 60 seconds so you can see it's still alive
6. After the wait, execution resumes from the same attempt

Cooldown state is persisted to disk. If the process is killed during a wait and restarted, it reads the remaining wait time from `circuit.json` and continues the pause from where it left off.

For OpenCode users with OpenRouter, **free mode** handles this differently: instead of waiting, it rotates to the next available free model immediately. Mid-stream. No restart needed.

---

## Mechanism 7: Free Tier Model Rotation (OpenCode + OpenRouter)

If you have OpenCode configured with OpenRouter, The Architect can run entirely on zero-cost models.

`architect --free`

On startup:

1. Fetches all models from `https://openrouter.ai/api/v1/models`
2. Filters for models where `pricing.prompt == "0"` AND `pricing.completion == "0"`
3. Excludes non-text-output models (audio generators, etc.)
4. Sorts by context length descending — larger context models first
5. Adds `openrouter/` prefix for OpenCode compatibility

During execution, when a free model hits a rate limit mid-stream, the rotator switches immediately to the next available model. No restart. No retry slot consumed. The task continues with the new model from where it left off.

When a model returns `"model not found"` or `"not available for this provider"`, it's marked as permanently unusable and skipped (not rate-limited — this is a different failure mode).

When all free models are exhausted, execution falls back to your default model from `opencode.json`.

The tradeoff is honest: free models are smaller and slower. On a 10-task goal, expect roughly 3x the wall-clock time compared to Claude Sonnet. Use it for non-urgent work or when budget is the constraint — not for a production fix you need in an hour.

*Free mode is not available with Claude Code. Claude Code routes through Anthropic's API directly, with no OpenRouter integration.*

---

## Mechanism 8: The Retrospective Reviewer

After all tasks execute, a separate reviewer agent runs. This is not the same agent that wrote the code. It's an auditor.

The reviewer reads:

- `PROGRESS.md` — full content, including failed states
- All task files — to understand what was planned
- The actual code written or modified
- The project context

Then it **runs your test suite**.

If it finds issues — failing tests, incomplete requirements, missing edge cases, code quality problems, inconsistencies — it creates R-prefixed fix-up tasks:

```
tasks/
├── T01_stripe_models.md      (done)
├── T02_payment_routes.md     (done)
├── T03_webhook_handler.md    (done)
├── R01_missing_webhook_test.md    (new — from reviewer)
├── R02_refund_edge_case.md        (new — from reviewer)
```

These R-tasks run through the full execution pipeline — with retry, circuit breaker, and completion detection — exactly like regular tasks. After R-tasks complete, if `retrospective_rounds = 2` (persistent mode), a second reviewer pass runs and reviews the fix-up work.

The reviewer is explicitly forbidden from:

- Modifying existing T-prefixed task files
- Rewriting `PROGRESS.md`
- Creating fix-up tasks when the work is clean — silence is success

In persistent mode (two retrospective rounds), the flow is:

```
Execution → Review 1 → Fix-up execution → Review 2 → Fix-up execution → Done
```

This is closer to how production engineering actually works: write, review, fix, review again.

---

## Mechanism 9: ARCHITECT.md — Persistent Project Intelligence

This is the mechanism that changes the nature of long-term autonomous work.

Every production system I've built has the same memory problem: the AI has no recollection of decisions made last session. You re-explain the same constraints. You re-discover the same failure modes. You re-make the same trade-offs.

`ARCHITECT.md` is the solution. It's a structured file that accumulates across every session and is read by every agent before they do anything.

### What it contains

**Project Structure** (rewritten fresh on every planning session):
- Repo type, detected languages, frameworks, and inferred component roles
- Component descriptions from `pyproject.toml` and `package.json`
- Key dependencies, test commands, lint commands
- Dependency graph between components

**Permanent Decisions** (append-only):
> database: SQLite — lightweight, no server needed, no ops overhead. Decided T01.

**Known Constraints** (append-only):
> Python 3.9+ required — httpx async breaks on 3.8. Discovered T04 attempt 2.

**Lessons Learned** (append-only):
> Don't skip test execution before marking PROGRESS.md Done — T07 declared complete with 3 failing tests. Always run pytest before promise tag.

**Best Practices** (append-only):
> Use typed Pydantic models for all API request/response schemas — avoids silent type coercions in production.

**Planning History** (auto-appended by The Architect after each plan):
> 2026-04-19: "add Stripe integration" — 5 tasks, standard scope

### How it flows through the system

Every planning session: the architect agent reads ARCHITECT.md first, before reading anything else. It's the highest-priority context. The planner knows every decision and constraint made in previous sessions before writing a single task.

Every task execution: the build agent receives the full ARCHITECT.md content in its instruction. It reads architectural context from a file, not from conversation history. The instruction explicitly tells it to update ARCHITECT.md when it discovers new constraints during execution.

Every retrospective: the reviewer reads ARCHITECT.md and is instructed to update it with cross-task patterns and quality findings.

All writes to ARCHITECT.md use atomic operations (temp file + `os.replace`) so readers never see partial content.

**The result:** After 50+ builds on The Architect's own codebase, ARCHITECT.md contained 23 architectural decisions and 11 lessons learned. None written manually. The agents built this memory while building the project. Session 10 doesn't repeat session 3's mistake — it reads the lesson from the file.

Commit ARCHITECT.md to git. It's the project's long-term memory and it improves with every session.

---

## Mechanism 10: PROGRESS.md — Session State

`PROGRESS.md` is how The Architect tracks which tasks are complete and what to run next. Every task reads it at the start and updates it at the end.

Format:

```markdown
## Overall Status

**Tasks completed:** 3
**Next task to run:** T04

## Task Log

| Task | Title | Status | Completed |
|------|-------|--------|-----------|
| T01 | Stripe models | Done | 2026-04-19 |
| T02 | Payment routes | Done | 2026-04-19 |
| T03 | Webhook handler | Done | 2026-04-19 |
| T04 | Refund logic | Pending | |

## Permanent Decisions

| Decision | Value | Reason | Task |
|----------|-------|--------|------|
| payment_provider | Stripe | client requirement | T01 |
```

The runner detects task state from two specific lines: `**Tasks completed:** N` and `**Next task to run:** TXX`. Task Done status is detected by grepping `TXX.*Done`.

The Architect always writes PROGRESS.md itself — never delegated to an agent. This guarantees it's always at the correct path and in the correct format.

---

## Mechanism 11: Token Budget

For overnight runs, cost control matters. The token budget prevents runaway spend.

```toml
[architect]
token_budget_per_hour = 500000   # ~5 Claude Sonnet calls per hour
```

The `HourlyTokenBudget` class tracks usage against a rolling 1-hour window. After each task completes, tokens are recorded. When the hourly budget is exceeded, execution pauses until the window resets — then resumes automatically.

Budget pauses don't consume retry slots. They don't affect circuit breaker state. A single Claude call can use 100k+ tokens, so calibrate accordingly: `500000` gives roughly 5 calls per hour, enough for steady progress without uncapped overnight cost.

Set it in `architect.toml` before you start a long run. Check `SUCCESS.md` in the morning for the actual spend.

---

## Mechanism 12: Overnight Safety — Everything Together

Here's what a safe, unattended overnight run looks like:

```toml
[architect]
persistent = true                 # 30 retries, 2 retrospective rounds
token_budget_per_hour = 500000
cooldown_detection = true         # default
circuit_enable_replan = true      # default
```

```bash
architect --plan --goal "refactor authentication layer to use JWT + refresh tokens"
architect
# Go to sleep
```

While you're gone, the system handles:

- Task fails on attempt 1 → retry with context carry, attempt 2 passes
- Rate limit hits on T06 → cooldown wait, no retry slot consumed, resumes in an hour
- T09 stuck in same-error loop → circuit trips after 3 identical fingerprints → REPLAN → architect rewrites T09 → new T09 passes on attempt 1
- All tasks complete → retrospective reviewer runs → finds missing test in T04 → creates R01 → R01 executes and passes → second reviewer pass finds nothing → done
- Process killed at 3am (power blip) → restart runs `architect` → reads circuit.json, PROGRESS.md → resumes from T11

You wake up to SUCCESS.md:

```
Result: ✓ All tasks completed
Tasks: 12/12 done
Duration: 6:43:12
Retries: 4 across 12 tasks
Rate limits hit: 1 (T06)
Retrospective: Round 1 — 1 issue found (R01). Round 2 — clean.
```

That's the whole point. You made the architecture decisions before bed. The Architect executed them overnight.

---

## The Dog-Food Story

The Architect was built using itself.

During development, it planned and executed tasks against its own codebase — the architect agent building the architect agent. The circuit breaker was tested by the circuit breaker. The retrospective reviewed the retrospective.

At task T47, the circuit breaker tripped. The build agent had failed on the same `FileExistsError` three consecutive retries — same logical pattern, different file paths, different line numbers. The fingerprinting logic normalised both away and recognised the same failure. Circuit opened. Chose REPLAN. Sent a targeted rewrite of T47 only.

T47 was the task implementing the lock file. The Architect caught a bug in its own lock file implementation using its own circuit breaker.

I found this in the build logs the next morning.

The build counter in `version.py` (`__build__`) increments on every completed agent operation and never resets. **The current version is v1.0.0 (build 10042).** That number is the honest record of how many autonomous operations it took to build and stabilise this tool — every one verified by multi-signal completion detection and at least one retrospective pass.

---

## What It Doesn't Do (Honest Limits)

**It doesn't write better code than your underlying model.** If Claude Code gets confused by your codebase, The Architect's orchestration layer won't fix that. The quality ceiling is the model. The Architect raises the floor — making sure the model actually finishes, correctly.

**A bad goal still produces bad tasks.** If your goal is underspecified, the architect agent produces vague task files. Invest time in a clear `--goal` or a well-structured `--context` file. Garbage in, garbage out — at all 12 mechanisms.

**The retrospective isn't a substitute for engineering judgment.** It catches test failures and incomplete requirements. It won't catch architectural drift, accumulating technical debt, or design decisions that are locally correct but globally wrong. It's a quality gate, not a code review.

**Local models can't do cross-repo work, even with The Architect.** Scope calibration and context minimisation let a 30k-context model execute isolated, focused tasks reliably. But cross-component work — where a task genuinely requires deep context from three different repos simultaneously — still needs a model with a large, reliable context window for that specific task. The Architect reduces how often you hit this ceiling by keeping tasks narrow. It doesn't eliminate the ceiling.

**Token counts are unavailable with Claude Code.** Claude Code outputs plain text, not structured JSON events. `SUCCESS.md` shows 0 for token totals when using Claude Code. Use OpenCode if token tracking matters.

**Free mode is slower.** Free OpenRouter models are smaller. On a 10-task goal, expect roughly 3x the wall-clock time compared to Claude Sonnet. Appropriate for non-urgent work. Not appropriate for a production incident.

---

## Get It

```bash
pip install the-architect
```

Requires Python 3.11+ and Claude Code or OpenCode:

```bash
# Claude Code
npm install -g @anthropic-ai/claude-code

# Or OpenCode
npm i -g opencode-ai
```

First run:

```bash
# Initialise (creates AGENTS.md and architect.toml)
architect init

# Plan and execute
architect --plan --goal "your goal here"
architect
```

Full documentation at **[github.com/iNetanel/the-architect](https://github.com/iNetanel/the-architect)**.

Every subcommand has a useful `--help`. The README has the full feature table, CLI reference, and configuration options.

If you're running local models and want to share what's working — open an issue. Context minimisation patterns for sub-10B models are still being refined and real production data is valuable.

If you find a failure mode I haven't covered — open a PR. The Architect can review it.

---

**About the Author**

Netanel Eliav builds production AI systems — agentic workflows, RAG pipelines, LLM infrastructure and AI evaluation frameworks. London-based.

[LinkedIn](URL) · [GitHub](https://github.com/iNetanel) · [inetanel.com](https://inetanel.com)