# Kindred — Implementation Milestones

Each milestone for building the Kindred application, in build order. Kindred is the working
product name for an AI-first development workspace (describe a change → an agent makes a bounded
change → preview → inspect the diff → approve the exact revision → merge → deploy → verify).

This file is the single checklist view of the work. The authoritative detail for each stage lives
in the build pack: run `build-pack/prompts/<NN>-*.txt` one at a time, and treat the "completion
gate" below as the bar that stage must clear before advancing. See
[`build-pack/START-HERE.txt`](build-pack/START-HERE.txt) for how to use the pack and
[`build-pack/ACCEPTANCE.txt`](build-pack/ACCEPTANCE.txt) for the release gates.

**Rules that apply to every milestone**

- Run one numbered prompt at a time, in order. Do not advance past a failing gate without
  recording the reason and fixing the dependency.
- A demo/fixture build is never live acceptance evidence. Real ("live") mode must never fall back
  to fixtures.
- Keep secrets out of chat, prompts, screenshots, and commits; configure them in environment files
  excluded from version control.
- Record evidence and the next stage in `docs/BUILD-STATUS.txt` after each milestone.

---

## Stage map

### 00 — Kickoff (prepare build context)
File: [`build-pack/prompts/00-kickoff.txt`](build-pack/prompts/00-kickoff.txt)

- [ ] Inventory the repository, runtime, package manager, existing auth/data/integrations, and checks.
- [ ] Map the pack to this codebase (default stack for a new app; preserve working equivalents in an existing one).
- [ ] Reconcile `build-pack/cursor-rules/kindred.mdc` into `.cursor/rules/kindred.mdc` without overwriting unrelated rules.
- [ ] Create `docs/DECISIONS.txt`, `docs/BUILD-STATUS.txt`, `docs/INTEGRATION-SETUP.txt`.
- [ ] Verify current framework/package/provider APIs from official docs (no versions from memory).
- [ ] Produce a stage map with each stage's completion gate. No production deployments or purchases.

### 01 — Foundation
File: [`build-pack/prompts/01-foundation.txt`](build-pack/prompts/01-foundation.txt)

Scaffold the stable Next.js/TypeScript stack, app shell, typed config validation, tokens/fonts,
error/not-found/loading states, server-only credential boundaries, narrow provider interfaces, and
the tooling (format, lint, typecheck, unit tests, Playwright). Clearly labeled local demo starts
without live secrets.

**Gate:** fresh install works; shell renders; lint/typecheck/build pass (or blockers documented);
malformed config fails helpfully; no secrets bundled into the browser.

### 02 — UI and demo
File: [`build-pack/prompts/02-ui-and-demo.txt`](build-pack/prompts/02-ui-and-demo.txt)

All five screens as one coherent, responsive interactive demo (Build, Activity, Files & changes,
Review & ship, Settings) built from reusable components, behind explicit demo mode with deterministic
fixture adapters. Every visible control acts, explains its availability, or is removed.

**Gate:** full demo traversable by keyboard and mouse; every route opens directly; refresh keeps the
selected project; desktop and mobile usable; no dead primary actions; demo status persistently visible.

### 03 — Adaptive workspace
File: [`build-pack/prompts/03-adaptive-workspace.txt`](build-pack/prompts/03-adaptive-workspace.txt)

Adaptive layout (conversation-first → conversation + preview → file inspector), resizable panels on
desktop, tabs/drawers at narrow widths, focus mode, and persistent local continuity (drafts,
selection, panel sizes) per project/user. No hydration errors or leaked preferences.

**Gate:** resize, navigation, reload, back/forward, mobile tabs, and draft recovery all keep context
at ~390px / 768px / 1440px; keyboard access works; no horizontal overflow.

> **Milestone — after 03:** a usable local product demo (not live AI).

### 04 — Auth and data
File: [`build-pack/prompts/04-auth-and-data.txt`](build-pack/prompts/04-auth-and-data.txt)

Real identity (Supabase Auth), server-verified sessions, reviewed migrations for the core records,
personal workspace provisioning, owner/editor/viewer authorization, invitations with expiry, and
server-stored data in live mode with tenant-scoped reads/writes and tested RLS/storage policies.

**Gate:** sessions survive refresh and expire correctly; no cross-workspace access by changing IDs;
viewers cannot mutate and editors cannot approve production releases; fresh-DB migration works;
missing live credentials produce a setup state.

### 05 — GitLab integration
File: [`build-pack/prompts/05-gitlab-integration.txt`](build-pack/prompts/05-gitlab-integration.txt)

A real GitLab.com OAuth connection (separate from app sign-in) with encrypted credential storage,
and the `GitProvider` boundary: repository discovery, authorized import, branch/tree/file/diff/checks,
webhook validation/dedup or polling. GitLab.com only in v1.

**Gate:** connect a dev account, import a test repo, browse real files/branches, disconnect, and see
expired/revoked states; unauthorized IDs, duplicate imports, OAuth-state failure, and webhook replay
handled. Fixtures do **not** satisfy this gate.

### 06 — Durable runs
File: [`build-pack/prompts/06-durable-runs.txt`](build-pack/prompts/06-durable-runs.txt)

Durable run lifecycle with versioned state, typed persisted events, monotonic cursors, reliable
dispatch (transactional outbox + reconciler), idempotency keys, project write leases with fencing,
cancellation semantics, and authenticated reconnectable event streaming. Browser observes; it never
owns execution.

**Gate:** duplicate dispatch starts one logical run; refresh/disconnect loses no events; worker
restart resumes/reconciles; concurrent writes serialize; stale workers cannot commit; cancelled and
failed runs stay inspectable.

### 07 — Sandbox and preview
File: [`build-pack/prompts/07-sandbox-and-preview.txt`](build-pack/prompts/07-sandbox-and-preview.txt)

`SandboxProvider` (Vercel Sandbox): isolated checkout of a specific commit, bounded command
execution, dependency install/build/dev-server inside the sandbox only, real preview readiness from
an actual health response, path/traversal/egress confinement, and previews served on a separate
origin. No project script runs on the app host.

**Gate:** a real test repo runs in isolation and yields a verified preview; a failed build shows
bounded logs; timeout/cancellation clean up; expired preview offers rebuild; path escapes and secret
redaction tested.

> **Milestone — after 07:** authenticated projects and a durable manual execution/preview path.

### 08 — Agent builder
File: [`build-pack/prompts/08-agent-builder.txt`](build-pack/prompts/08-agent-builder.txt)

One real builder agent (Vercel AI SDK + configured Anthropic model) running a bounded
plan → inspect → edit → check loop on the existing durable-run and sandbox infrastructure. Typed
tools (list/read files, apply validated patches, run approved commands) enforced by the backend, not
the prompt. Repository text and tool output are untrusted. No tool may merge or deploy.

**Gate:** a small real change on a test project produces an actual diff and preview; a failing test
stays failed and an absent suite is not "passed"; reload preserves execution; disallowed tools/paths,
prompt-injection, budget exhaustion, cancellation, and repeated tool delivery are tested.

### 09 — Changes and review
File: [`build-pack/prompts/09-changes-and-review.txt`](build-pack/prompts/09-changes-and-review.txt)

Real Git diffs between base and head (unified/split, search, pagination, binary/large-file notices),
read-only file inspection with revision labels, real check outcomes bound to the SHA, revision-bound
reviewer comments that go outdated correctly, and "request changes" that creates a linked new run
without rewriting history.

**Gate:** multi-file selection and deep links survive refresh; edge-case diffs render safely; comments
persist and become outdated; a new revision visibly invalidates previous readiness; diff/file endpoints
enforce tenant authorization.

> **Milestone — after 09:** real agent-generated changes with resumable activity and file inspection.

### 10 — Approval and merge
File: [`build-pack/prompts/10-approval-and-merge.txt`](build-pack/prompts/10-approval-and-merge.txt)

Owner-only approval bound to the immutable changeset/head SHA, target, required-check evidence, and
policy version; a merge path that re-fetches provider truth immediately before merging, uses
conditional SHA constraints, and reconciles uncertain outcomes via an idempotent release coordinator.

**Gate:** owner can approve a verified revision; editor/viewer cannot; a push after approval blocks
merge; failed/missing/stale checks block; concurrent approve/merge cannot duplicate releases;
uncertain merge outcomes reconcile.

### 11 — Deploy and rollback
File: [`build-pack/prompts/11-deploy-and-rollback.txt`](build-pack/prompts/11-deploy-and-rollback.txt)

`DeploymentProvider` (Vercel): deploy the exact approved merged revision, status reconciliation,
meaningful health verification, a receipt (URL, revision, timestamps, evidence, IDs), recovery from
deploy-failure-after-merge without a second merge, and owner-only confirmed rollback to a known prior
deployment. v1 excludes automated production schema migrations.

**Gate:** a dev release deploys the intended source, passes health verification, and recovers from
timeout / failed build / failed health; unauthorized deploy/rollback and repeated actions tested.
Absent live credentials → gate unverified and live shipping disabled.

> **Milestone — after 11:** reviewed code can merge and deploy through an owner-authorized release.

### 12 — Limits and operations
File: [`build-pack/prompts/12-limits-and-operations.txt`](build-pack/prompts/12-limits-and-operations.txt)

Settings wired to membership, connection health, project config, and owner budgets; per-run and
per-workspace limits with atomic reservation; artifact retention and stale-sandbox cleanup;
structured redacted logs with correlation IDs; health endpoints; and an owner-visible audit trail.
No payment charging in this stage.

**Gate:** concurrent requests respect reservations; exhausted limits block new work helpfully; cleanup
handles abandoned runs; secrets never appear in payloads/logs/traces/artifact links; revoking access
affects future writes and subscriptions.

### 13 — Integration and recovery QA
File: [`build-pack/prompts/13-integration-and-recovery-qa.txt`](build-pack/prompts/13-integration-and-recovery-qa.txt)

Meaningful boundary tests (authorization, isolation, OAuth state, webhook auth/replay, outbox,
leases/fencing, reconnect, cancellation, path escapes, check provenance, revision-bound approval,
uncertain merge/deploy, reservations, private artifacts) exercised through injected provider errors
and real DB transitions; a disposable real-repo fixture; a separately gated live smoke suite; and a
full Playwright journey with mid-flow failures.

**Gate:** required automated checks pass; every failing or unrun scenario has an accurate explanation;
no live operation is claimed verified on fixture evidence alone. Update ACCEPTANCE evidence.

### 14 — Pilot handoff
File: [`build-pack/prompts/14-pilot-handoff.txt`](build-pack/prompts/14-pilot-handoff.txt)

Fix remaining in-scope defects, remove misleading affordances, and produce the handoff: fresh-machine
setup guide, `.env.example`, migration procedure, provider configuration, worker/webhook deployment
instructions, rollback/reconciliation runbook, backup/restore plan, secret classification, and a
go/no-go table (implemented / verified / unverified / blocked). No production deploy, purchase, merge,
or migration as part of the handoff.

**Gate:** full local/CI checks plus a configured development smoke journey run; the build outcome,
evidence, pilot prerequisites, and smallest next action are stated; `BUILD-STATUS` lets a fresh chat
continue.

> **Milestone — after 14:** pilot readiness is determined by evidence, not by the number of screens
> implemented.

---

## Recovery prompts (use when needed, not in sequence)

- [`build-pack/prompts/90-resume.txt`](build-pack/prompts/90-resume.txt) — restore context in a fresh chat.
- [`build-pack/prompts/91-repair.txt`](build-pack/prompts/91-repair.txt) — recover a failing implementation.
- [`build-pack/prompts/92-visual-review.txt`](build-pack/prompts/92-visual-review.txt) — correct design drift against `references/*.png`.

## Release acceptance gates

Milestones ship behind the acceptance gates in [`build-pack/ACCEPTANCE.txt`](build-pack/ACCEPTANCE.txt):

- **Gate A — Local demo** (covers stages 01–03)
- **Gate B — Identity and isolation** (stage 04)
- **Gate C — Execution** (stages 05–08)
- **Gate D — Review and release** (stages 09–11)
- **Gate E — Pilot operations** (stages 12–14)
