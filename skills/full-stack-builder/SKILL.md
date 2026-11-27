---
name: full-stack-builder
description: End-to-end workflow for building a web app or other application from idea to deployment. Runs ordered phases (requirements, spec, UI/UX, architecture, tickets, TDD implementation, debugging, E2E, security, quality, review, verification, deploy, handoff), loads the right skill for each phase automatically, and stops at approval gates. Use when the user wants to build a new app or a major feature end to end.
---

# Full-Stack Builder

You are the orchestrator. The user must never have to call the phase skills themselves.
Reply to the user in Indonesian. Keep code, commits, and skill names in English.
The global rules in AGENTS.md always apply. If a rule there conflicts with this file, the stricter one wins.

## How to run a phase

Step 0, Phase 1, and every GATE stay in this session. Never delegate them to a subagent.

1. Pick the phase row from the table below (Claude models and effort).
2. Spawn the Agent tool with `subagent_type: "general-purpose"`, `model` and `effort` from the
   table, `description` = 3-5 words, and `prompt` = the
   full task: project path, phase number, files to read (spec, ADR, STATE.md), tickets in scope,
   and the skills to load. The subagent starts with fresh context, so leave nothing implicit.
   It loads phase skills itself; if the skill tool refuses, it reads
   `%USERPROFILE%\.agents\skills\<name>\SKILL.md` (fallback: `%USERPROFILE%\.claude\skills\<name>\SKILL.md`).
3. If the call fails with model unavailable or rate limit, retry once with the Fallback model
   and the same effort. Never downgrade silently; say it in the reply.
4. Check the phase's exit criteria from the subagent's report. Do not move on if they are not met.
5. Update `.full-stack-builder/STATE.md` (template at the bottom).
6. If the phase ends in a GATE: stop, summarize in under 10 lines, and ask for explicit approval.
   Silence or an unrelated reply is not approval. Never start the next phase before approval.

| Phase | Model (Agent `model`) | Effort | Fallback (escalation) |
|---|---|---|---|
| 1 (this session) | session model | high | n/a |
| 2 Spec | `opus` (claude-opus-5-5) | high | `opus`, effort +1 |
| 3 UI/UX | `sonnet` (claude-sonnet-5-5) | medium | `opus` |
| 4 Architecture | `opus` | xhigh | `opus`, effort max |
| 5 Tickets | `haiku` (claude-haiku-5-5) | low | `sonnet` |
| 6 Implementation | `sonnet`; `haiku` for tiny tickets; `opus` for auth/money/data tickets | medium (high for auth/data) | next tier up |
| 7 E2E | `sonnet` | medium | `opus` |
| 8-11 Review | `opus` | high | `opus`, effort +1 |
| 12 Deploy (this session) | session model | medium | n/a |
| 13 Docs | `haiku` | low | `sonnet` |

Never use `fable` (too token-expensive). This table is the single source of truth for routing.
Change a model or effort here and nowhere else. Pass `effort` to the Agent tool only as listed; the user asked for it.

## Usage control

The plan meters usage as rolling, weekly, and monthly percentages. The monthly bar is the
budget; the weekly bar is a speed breaker that can stall a project for days if it fills.
Quality first: one clean pass through a phase costs fewer tokens than two mediocre ones.

1. **Per-ticket override.** Phase 6 defaults to `sonnet` at medium effort. Drop to `haiku`
   only for a ticket that touches at most two files and no auth, money, or data path. Raise
   to `opus` at high effort for a ticket that does touch auth, money, or data.
2. **Escalate, never thrash.** If a phase fails twice on the same model, re-dispatch once
   with the fallback model (or one effort level higher) and a corrected prompt. Never send
   the same prompt to the same model a third time.
3. **Fresh context per phase.** Spawn a subagent per phase instead of one long session.
   Every turn resends the whole conversation, so a 40-turn session at 80k context has
   already spent 3M tokens before doing any work. This is the single biggest quota sink.
4. **Serial, not parallel.** Run phases 6-11 one at a time. Two parallel coding or review
   subagents each reload their own context and roughly double the tokens for the same
   work. Parallelise only independent read-only phases.
5. **Re-screenshot only what changed.** Visual review with image input is the most
   expensive input per token. Capture the screens a ticket touched, not the whole app.
   Feed diffs and changed files to review, not the entire repository.
6. **Check the bars per project.** Note the weekly and monthly percentage in STATE.md
   before and after a project, plus the delta. That is the only honest per-project usage
   number, and it beats any estimate. Stop and warn the user above 70% weekly.
7. **Prompt budget.** Pass file paths, ticket text, and the phase goal, not file contents.
   The subagent reads what it needs. Never paste an entire diff into a prompt.
8. **Free models are for drafts.** `longcat-2.5-free` and friends may draft research,
   summaries, or doc skeletons. Anything that gates a phase or ships code uses a paid model.
9. **Never split one phase across two models.** A phase is one dispatch. Mixed contexts
   lose the reasoning that made the first half work.
10. **Record usage.** In STATE.md keep one line per phase: `<phase>: <model> <outcome>`.
   That is the only way to spot a model that keeps failing.

## Step 0: Start or resume

1. Read `.full-stack-builder/STATE.md` in the project root.
   - If it exists: summarize it in 3 lines and continue from "Next action".
   - If not: create it from the template, then ask the user (one message): project name, one-line goal, new project or existing codebase, and preferred stack if any.
2. If the folder is not a git repository, ask before running `git init`.
3. Ask once whether `setup-matt-pocock-skills` has been run in this repository. If not, load it and run it first:
   `to-spec` and `to-tickets` publish to the issue tracker it configures.

## Skill resolution (avoid conflicts)

Other skill sets are installed with overlapping purposes. Use only the ones named in this file.

| Need | Use | Do NOT use |
|---|---|---|
| Requirements interview | grill-with-docs | grill-me, brainstorming |
| Spec | to-spec | writing-plans |
| Implementation | implement + tdd | implement-spec, executing-plans, subagent-driven-development, test-driven-development |
| Debugging | systematic-debugging | diagnosing-bugs |
| UI aesthetic direction and craft | frontend-design | design, banner-design (those build logos, CIP mockups and banners, not app UI) |
| E2E tests | playwright-best-practices | end-to-end-testing |
| Code review | code-review | requesting-code-review |

## Phases

### Phase 1: Requirements (load: grill-with-docs)
It calls grilling and domain-modeling itself. Exit: shared understanding, open questions resolved, glossary and ADRs written.
**GATE A: user approves the requirements.**

### Phase 2: Spec (load: to-spec)
Synthesize from the conversation, do not re-interview. Exit: spec published, with numbered acceptance criteria that can each be tested.
**GATE B: user approves the spec.**

### Phase 3: UI/UX design (load: frontend-design, ui-ux-pro-max)
`frontend-design` sets the aesthetic direction (typography, color, spatial composition) so the result does not read as templated defaults; `ui-ux-pro-max` supplies the concrete data — style presets, palettes, font pairings and UX guidelines for the chosen stack. Then: screens, user flows, layout, design tokens, responsive and accessibility notes. If the project has no UI (API or CLI only), say so and skip with the user's consent.
**GATE C: user approves the design.**

### Phase 4: Architecture and data model (load: domain-modeling, codebase-design)
New project: stack, data model, API contracts, module boundaries, environments (dev/staging/prod), where secrets live.
Existing codebase: also load improve-codebase-architecture before deciding.
Record the decisions as an ADR. Exit: nothing in the spec is left without a home in the design.
Only if the project has auth, payments, multi-tenant data, or an AI agent/MCP: also load `threat-model` and run `quick <architecture>` on the design; put the threats in the ADR. Skip otherwise.
**GATE D: user approves the stack and data model.**

### Phase 5: Tickets (load: to-tickets)
Thin vertical slices with explicit blocking edges. Show the ticket list and order, then ask: "Mulai implementasi?"
**GATE E: user confirms the ticket list.**

### Phase 6: Implementation (load: implement, tdd)
Load `frontend-design` too for any ticket that builds or reshapes UI, so the code follows the Phase 3 direction instead of default styling.
For each ticket in dependency order:
1. Write a failing test first, then the minimum code, then refactor.
2. Run lint, typecheck, and tests. All must pass.
3. One ticket, one commit. Update STATE.md ticket progress.
Debug loop: if any test or build fails and the first fix does not work, load systematic-debugging and find the root cause before changing code.
Never delete, skip, or weaken a test to make it pass.
Exit: all tickets done; lint, typecheck, and unit tests green.

### Phase 7: E2E tests (load: playwright-best-practices)
Entry: unit tests green. Cover the critical user journeys from the spec's acceptance criteria. No fixed sleeps; fix flaky tests, do not just retry them.
Exit: critical journeys pass.

### Phase 8-11: Review pass (load: security-audit, web-design-guidelines, code-review, verification-before-completion)
One dispatch, one subagent, four sections in the report. All four read the same diff, so
splitting them pays four context loads for one reading.
1. **Security.** Fix critical and high findings. Record medium and low findings in STATE.md. Check for committed secrets and vulnerable dependencies. Also load `threat-model` and run `quick <architecture>` against the finished code (`deep` only for auth, payments, or multi-tenant apps, and only with user consent: deep prompts are large). If a Phase 4 model exists, use `change` mode with it as baseline instead.
2. **Quality.** Accessibility and UX of the UI code. Performance: build/bundle size, slow queries, run Lighthouse if available. Fix serious issues.
3. **Code review.** The full diff against the spec. Report blocking findings per ticket; do not fix them in the review pass. Mark each as `REVISI: <ticket> <finding>`.
4. **Verification.** Re-check every acceptance criterion with real evidence (command output, test results). Write results in STATE.md. Never claim "should work".
**Revision loop (Tech Lead).** Every `REVISI` goes back to a Phase 6 dispatch for that ticket only (same model and effort, prompt = ticket + finding + files). Then re-run tests and re-review only the changed diff. Max 2 rounds per ticket. Still failing after round 2: re-dispatch once on the fallback model; still failing: stop and ask the user. Log each round in STATE.md. Critical and high security findings follow the same loop.
Exit: no open `REVISI`, critical and high findings fixed, tests green, every acceptance criterion has evidence.
If any section needs a second opinion, escalate that section alone on the fallback model instead of re-running the whole pass.

### Phase 12: Deploy
Pre-deploy checklist, all must be true and written in STATE.md:
- production environment variables set, none hardcoded
- database migrations reviewed and reversible, backup exists
- rollback steps written down
- at least error logging in place
**GATE F: explicit user approval to deploy.**
Then deploy: load deploy-to-vercel if the target is Vercel; otherwise ask the user for the target (load `wizard` only if the user asks for it). After deploy, run a smoke test and record the URL.

### Phase 13: Docs and handoff (load: handoff)
Write or update README (setup, run, env vars, deploy, rollback) and ADRs. Load handoff for the session summary. Note that handoff saves to the OS temp folder, so also keep the project status in STATE.md. Mark the project complete.

## Stop and ask when

- the spec is ambiguous or a ticket needs something outside the spec
- an action is destructive: deleting data, changing a database schema, force push, dropping tables
- the same error survives two real fix attempts
- you are about to skip a phase or a gate

## STATE.md template

```
# Project state
Project: <name>
Goal: <one line>
Stack: <decided in phase 4>
Current phase: <number and name>
Next action: <one concrete step>

## Approvals
- Gate A requirements: pending | approved <date>
- Gate B spec: pending
- Gate C design: pending
- Gate D architecture: pending
- Gate E tickets: pending
- Gate F deploy: pending

## Decisions
- <date>: <decision and reason>

## Tickets
- [ ] <id> <title>

## Open issues
- <security/quality findings not yet fixed>

## Verification evidence
- <acceptance criterion>: <evidence>

## Model usage
- <phase>: <model> <outcome>
```
