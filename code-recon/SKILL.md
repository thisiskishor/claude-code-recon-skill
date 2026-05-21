---
name: code-recon
description: "Maps any codebase before you touch it. Use when starting on unfamiliar code, before a risky change, or when you need to understand what's connected to what. Triggers on: 'map this repo', 'code audit', 'quick scan', 'deep scan', 'what breaks if I change X', 'blast radius of this change', 'what depends on this file', 'recon diff', 'what changed since last scan', 'analyze this project', 'give me a topology report'. Also triggers automatically before modifying an unfamiliar file mid-session."
license: MIT
metadata:
  author: Kishor Gartaula (@thisiskishor)
  version: 2.1.0
---

# Code-Base Reconnaissance Analyst

> Perform a read-only, non-intrusive scan of a software repository and produce a structured topology report - before any code is touched.

The goal is to fully map the shape of the application and the geometry inside it: what relationships do components have? How does data flow? Build a mental model and let the report serve as a persistent signal you can draw from throughout the session.

**Access level:** Read-only. No writes, commits, or configuration changes at any point.

---

## Invocation Modes

Choose the mode that fits the situation. If the user does not specify, default to **Standard**.

| Mode | When to Use | Time |
|---|---|---|
| [Quick](#quick-scan) | Unfamiliar file touched mid-task, fast orientation needed | ~2 min |
| [Standard](#standard-scan) | Starting a new feature, joining an existing project | ~10 min |
| [Deep](#deep-scan) | Pre-refactor, security review, onboarding to a large codebase | ~20 min |
| [Change-Impact](#change-impact-mode) | About to modify a specific file or function | ~5 min |
| [Recon Diff](#recon-diff-mode) | Second scan on a repo you have scanned before | ~5 min |

---

## When to Use This Skill

Use Code-Recon when you need to:

- **Start on unfamiliar code** — Before touching anything, map the territory first
- **Make a risky change** — Understand blast radius before modifying a high-centrality file
- **Review security** — Pre-launch audit to find input validation gaps, auth issues, secrets in code
- **Onboard to a large codebase** — Fast topology report beats hours of reading
- **Refactor safely** — Know what depends on the code you're about to change
- **Debug emergency issues** — Understand the system before making changes
- **Track architectural drift** — Compare scans over time to see what changed

**Who benefits most:**
- Solo developers joining new projects
- Team leads onboarding engineers
- Security reviewers doing audits
- Refactoring specialists planning large changes
- On-call engineers fixing production bugs

---

## How to Use (Invocation Examples)

### Quick Scan (fast orientation)

Trigger with any of these:
```
Quick scan
Fast orientation of this project
Quick analysis
```

Claude returns stack, entry points, high-centrality files, immediate red flags in ~2 minutes.

### Standard Scan (full topology)

```
Code audit
Map this repo
Analyze this project
Give me a topology report
```

Full report: stack, endpoints, data flow, security boundaries, test coverage, dead code, risk table, fixes ordered by impact.

### Deep Scan (security + architecture)

```
Deep scan
Full security review
Comprehensive analysis
```

Everything in Standard, plus:
- Input-to-database path tracing (where does user input go?)
- Coverage cross-reference (which endpoints are untested?)
- Dependency depth analysis (what breaks if I change X?)
- Async hazard detection (race conditions, missing awaits)
- Architecture assessment against 20 domain wisdom seeds

### Change-Impact Mode (blast radius)

```
What breaks if I change this file?
Blast radius of modifying X
What depends on this function?
Is it safe to delete this?
```

Maps direct dependents, transitive dependents, side effects, test coverage, and rates the blast radius (Contained / Moderate / Wide / Unknown).

### Recon Diff (compare scans)

```
What changed since the last scan?
Compare to previous report
Recon diff
```

Diffs two scans, shows what's new, what's removed, regressions, improvements, and architectural drift.

---

## Pre-Task Checkpoint

Before modifying any file in an area that has not been mapped yet in the current session, pause and run a Quick scan on that module first. The cost of a 2-minute orientation is always lower than the cost of an unintentional side effect.

Skip the checkpoint only if:
- The change is trivial (typo, rename, comment)
- The area was already scanned earlier in this session
- The user explicitly says "just do it"

---

## Quick Scan

**Goal:** Fast orientation. Stack, entry points, obvious red flags. No deep traversal.

### Steps

1. **Identify the technology stack** - note framework names and version numbers. Look at `package.json`, `requirements.txt`, `pom.xml`, `go.mod`, or equivalent. Do not guess the stack from folder names alone.

2. **List top-level directories** - one line per directory explaining its purpose. Note any non-standard folder names that might confuse a new reader.

3. **Find the main entry point(s)** - where does the app actually start? Look for `main()`, `app.listen()`, `handler` exports, or framework-specific bootstrapping files. There may be more than one (e.g. separate API and worker entry points).

4. **Spot immediate red flags** - hardcoded credentials, missing auth middleware, `.env` files committed, obvious dead code at the top level, or any file that looks like it should not be there.

5. **Identify the 3-5 highest-centrality files** - the files most things depend on. These are the load-bearing walls. Changing them has the widest blast radius.

### Output Format

```
## Quick Scan: [Project Name]

**Stack:** [one-liner]
**Entry points:** [list]
**High-centrality files:** [list, one line each explaining why it matters]
**Immediate flags:** [list, or "none found"]
**Recommended next step:** [Quick is sufficient / run Standard for full picture]
```

---

## Standard Scan

**Goal:** Full topology report. Everything a developer needs before starting work.

### Steps

**1. Discovery and Inventory**
- List all top-level directories and their purpose
- List all configuration files (`tsconfig.json`, `webpack.config.js`, `Dockerfile`, `docker-compose.yml`, `.env.example`, etc.) and note what each one controls
- Enumerate every external dependency and note version numbers - flag anything that is pinned to an outdated or deprecated version
- Identify all services and their entry points: web server, background workers, scheduled jobs, serverless functions

**2. Interface and Endpoint Mapping**
- Scan for all API definitions: REST routes, GraphQL resolvers, gRPC services, WebSocket handlers
- For each endpoint record: HTTP method, path, auth requirement, and the file/function that handles it
- Map all integration points with external systems: third-party APIs, message queues, webhooks, storage buckets
- Note the authentication mechanism for each integration (API key, OAuth, JWT, service account)

**3. Data Flow and Call-Chain Analysis**
- Trace the path from each entry point down to the leaf functions it calls
- Highlight cross-layer calls: frontend calling backend, backend calling external services, services calling the database
- Identify where user-controlled input enters the system and trace it forward - where does it go, what touches it, where does it land in the database
- Flag any place where data crosses a trust boundary without being validated or sanitized

**4. Security Boundary Identification**
- Find all auth and authorization checks - which routes are protected, which are not, and what mechanism is used
- Check CORS configuration - is it locked to specific origins or open to all?
- Check for rate limiting - which routes have it, which do not
- Look for missing security headers
- Flag high-risk patterns: direct database queries from frontend code, unvalidated redirects, file uploads without type checking, secrets in source code, insecure deserialization

**5. Test Coverage Assessment**
- Map each module and endpoint to its test file, if one exists
- Estimate coverage level: well-tested (>80%), partial (40-80%), minimal (<40%), or untested
- Flag missing test oracles - tests that only assert an HTTP 200 status without checking the actual response payload are almost as bad as no tests at all
- Note any critical paths (auth, billing, data mutation) that have no tests

**6. Dead and Unused Code**
- Find unused imports, functions, variables, and entire modules
- Look for commented-out code blocks - these are usually either important context someone was afraid to delete, or just noise
- Identify legacy stubs, feature flags that are always-on or always-off, and code wrapped in `if false` blocks
- Flag public APIs with no documentation and no apparent callers

**7. Risk Summary Table**
For each risk, include a rating, a one-line justification, and the domain-wisdom seed from `references/domain-wisdom.md` that explains the underlying principle.

```
| Area | Rating | Justification | Seed |
|---|---|---|---|
| [area] | Critical / High / Medium / Low | [why it matters] | Seed N: "[name]" |
```

After the table, add a **Fix This First** list - the top 3-5 actions ordered by impact and ease combined, not just by severity. A Medium risk that takes 10 minutes to fix often belongs above a Critical risk that requires a full refactor.

**8. Recommended Next Steps**
Numbered, specific, and actionable. Not generic advice like "add more tests." Instead: "Add input validation to `PATCH /api/users` using the same `zod` schema pattern already used in `routes/auth.ts`."

### Output Format

```
# Topology Report: [Project Name]

## Executive Summary
## Technology Stack
## Directory Structure
## Services and Entry Points
## API Surface
## Data Flow
## Security Boundaries
## Test Coverage
## Dead and Unused Code
## Risk Summary
## Fix This First
## Recommended Next Steps
```

---

## Deep Scan

**Goal:** Everything in Standard, plus full input-to-database path tracing, coverage cross-referencing, dependency depth analysis, async hazard detection, and a complete architecture assessment against all 20 domain-wisdom seeds.

### Additional Steps (on top of Standard)

**A. Input-to-Database Path Tracing**

For every route that accepts user input, trace the complete path:

```
HTTP handler -> validation layer -> service layer -> ORM or raw query -> database
```

At each step, answer:
- Is the data validated at this step, or passed through raw?
- Is the data transformed in any way that could introduce risk?
- Could this step be bypassed?

Flag any point where the chain is broken or a step is skipped entirely.

**B. Coverage-to-Endpoint Cross-Reference**

Build a table linking every API endpoint to its test file, estimated coverage, and whether the test checks payload correctness (not just status codes).

```
| Endpoint | Test File | Coverage | Payload Assertions |
|---|---|---|---|
| POST /api/users | tests/users.test.ts | ~72% | Status only - payload assertions missing |
| POST /api/billing | None | 0% | None |
```

**C. Dependency Depth Analysis**

For the 5 highest-centrality files:
- List everything they import (their dependencies)
- List every file that imports them (their dependents)
- Rate the risk: a file that is widely depended-upon AND has no tests is the most dangerous thing in the codebase

**D. Async and Timing Hazard Detection**

Scan specifically for:
- Missing `await` on async calls
- Unhandled promise rejections (`.catch()` missing or swallowed)
- Race conditions from concurrent writes to shared state
- Background jobs that are not idempotent (running them twice produces a different result than running them once)
- Scheduled jobs that all fire at the same time (synchronized burst - see Seed 19)

**E. Architecture Assessment Against Domain Wisdom Seeds**

Go through all 20 seeds in `references/domain-wisdom.md`. For each one, check whether the codebase honors or violates it. Only report violations - skip seeds where no relevant pattern exists in this codebase. For each violation, name the specific file or pattern that triggered the finding.

### Output Format

All Standard sections, plus:

```
## Input-to-Database Path Tracing
## Coverage-to-Endpoint Matrix
## Dependency Depth (top 5 files)
## Async and Timing Hazards
## Architecture Assessment (violations only, with Seed references)
```

---

## Domain Wisdom Seeds (Production System Principles)

When analyzing architecture in Standard, Deep, or Change-Impact mode, reports reference these 20 seeds from `references/domain-wisdom.md`. These are principles for identifying structural problems like lock contention, wrong-tool usage, and scalability ceilings.

**Sample seeds (see `references/domain-wisdom.md` for all 20):**

- **Seed 1: "State is the bottleneck"** — Shared state causes lock contention. Design for isolation.
- **Seed 2: "Visibility is survival"** — Logs, metrics, and traces prevent silent failures. Instrument everything.
- **Seed 3: "Cascading failures kill systems"** — If A fails, does B fail? Does B's failure propagate to C? Design for isolation.
- **Seed 4: "Retry storms drown the system"** — Exponential backoff + jitter, not naive retries.
- **Seed 5: "Cache coherence is hard"** — Caches are a source of truth bugs. Know your invalidation strategy.
- **Seed 6: "Database is the throughput ceiling"** — Most systems are DB-limited. Understand your bottleneck.
- **Seed 7: "Input is the enemy"** — Every external input is a potential attack vector. Validate always.
- **Seed 8: "Authorization is not authentication"** — Auth checks who you are. Authorization checks what you can do. Easy to confuse.
- **Seed 9: "Secrets at rest"** — Never log, store, or transmit unencrypted secrets. Ever.
- **Seed 10: "Synchronous calls are slow calls"** — Network calls block. Design async when latency matters.
- **Seed 11: "Distributed consensus is expensive"** — If you need all nodes to agree, your design has a bottleneck.
- **Seed 12: "Minimalism"** — Fewer dependencies = smaller attack surface = easier to understand.
- **Seed 13: "Time is fake"** — Clocks drift. Rely on logical ordering, not wall-clock time.
- **Seed 14: "Partial failures exist"** — Systems don't fail cleanly. Some requests succeed, some fail. Design for that.
- **Seed 15: "Observability scales inversely with complexity"** — Complex systems need better metrics or they become opaque.
- **Seed 16: "Backpressure prevents drowning"** — If consumers can't keep up, slow down producers.
- **Seed 17: "Idempotency is your friend"** — Safe retries require operations that can run twice without side effects.
- **Seed 18: "Coupling spreads pain"** — Tight coupling between modules = changes cascade everywhere.
- **Seed 19: "Synchronized burst kills systems"** — When all background jobs fire at the same time, the system shrieks.
- **Seed 20: "Graceful degradation > clean failure"** — Better to serve most users slowly than fail everyone.

Each risk in the report links to the relevant seed so you understand not just what the problem is, but why it matters architecturally.

---

## Change-Impact Mode

**Goal:** Given a specific file or function, map everything that would be affected by changing it. Focused blast-radius analysis before making a move.

### When to Use
- "what breaks if I change X"
- "blast radius of modifying Y"
- "what depends on this file"
- "is it safe to delete this function"
- "impact of removing this module"

### Steps

1. **Confirm the target** - if the user's description is ambiguous, ask for the exact file path or function name before proceeding. Do not guess.

2. **Map direct dependents** - find every file that imports or calls the target directly. List them with file paths.

3. **Map transitive dependents** - find everything that depends on the direct dependents. This is the full blast radius. Stop at 3 levels deep unless the chain is unusually short.

4. **Map the target's own dependencies** - what does the target itself rely on? If you change the target, what upstream contracts might break?

5. **Identify side effects** - look inside the target for: database writes, external API calls, event emissions, cache invalidations, state mutations. These are the hidden blast radius items that do not show up in import analysis.

6. **Check test coverage on the impact zone** - are the direct dependents tested? Is the target itself tested? If not, any change is a blind change.

7. **Rate the blast radius:**
   - **Contained** - affects 1-3 files, all well tested
   - **Moderate** - affects 4-10 files, partial test coverage
   - **Wide** - affects 10+ files or touches untested critical paths
   - **Unknown** - the dependency chain cannot be fully traced (flag this explicitly and explain why)

### Output Format

```
## Change-Impact Report: [target]

**Blast Radius:** Contained / Moderate / Wide / Unknown
**Direct dependents:** [list with file paths]
**Transitive dependents:** [list, or "none found beyond direct"]
**Target's own dependencies:** [what this file relies on]
**Side effects inside target:** [DB calls, API calls, events, state changes]
**Test coverage on impact zone:** [summary]
**Safe to change?** Yes / Proceed with care / Requires tests first / Do not change without full audit
**Recommended approach:** [specific steps to make the change safely]
```

---

## Recon Diff Mode

**Goal:** Compare the current state of the codebase against a previous scan. Surface what changed, what regressed, and what improved.

### When to Use
- "compare this to the last scan"
- "what changed since the last report"
- "run a diff scan"
- "what is new since we last looked at this"

### Steps

1. **Locate the previous report** - ask the user where it is, or check the current session context for an earlier scan output.

2. **Run a Standard scan** on the current state of the codebase.

3. **Diff the two reports** across these dimensions:
   - New endpoints added or removed
   - New packages added or removed
   - Test coverage changes (improved or degraded)
   - New dead code introduced
   - New risks identified or old risks resolved
   - Architecture violations that appeared or disappeared

4. **Classify each change** using these markers:
   - `[+]` New addition
   - `[-]` Removed
   - `[~]` Modified
   - `[!]` Regression - something got worse
   - `[✓]` Improvement - something got better

### Output Format

```
## Recon Diff: [Project Name]

**Previous scan:** [date or session reference]
**Current scan:** [today's date]

### What Changed
| Area | Change | Classification |
|---|---|---|
| [area] | [description of change] | [+] / [-] / [~] / [!] / [✓] |

### New Risks
### Resolved Risks
### Coverage Changes
### Architectural Drift (new seed violations since last scan)
### Summary
```

---

## Worked Examples

### Example 1: Quick Scan Output

**Input:** "quick scan" on an unfamiliar Node.js project mid-task

**Output:**

```
## Quick Scan: checkout-service

**Stack:** Node.js 18 + Express 4 + PostgreSQL (Knex ORM)
**Entry points:** 
  - src/server.ts (REST API, port 8080)
  - src/worker.ts (background job processor)

**High-centrality files:**
  - src/db/client.ts — Database connection pool; 18 files import this
  - src/middleware/auth.ts — JWT validation on all protected routes
  - src/types/index.ts — Core TypeScript types; 22 files depend on this

**Immediate flags:**
  - DATABASE_URL hardcoded in src/config.ts (should be .env)
  - No input validation on /api/checkout POST (accepts raw amount)
  - Missing CORS configuration (open to all origins)

**Recommended next step:** Run Standard for full security analysis
```

---

### Example 2: Standard Scan Output (Excerpt)

**Input:** "code audit" on a mature Next.js e-commerce platform

**Output (sections shown):**

```
# Topology Report: ecommerce-platform

## Executive Summary
- 52 API routes, 12 database tables, 4 external integrations (Stripe, SendGrid, Supabase Auth, S3)
- 19 high-risk findings; 5 are critical (input validation, auth gaps, secrets in logs)
- Test coverage: 38% overall (low); checkout flow untested (critical)
- Build status: ✅ passing, 0 TypeScript errors

## Risk Summary

| Area | Rating | Justification | Seed |
|---|---|---|---|
| Missing input validation on `/api/checkout` | **Critical** | POST accepts raw amount, no bounds | Seed 7: "Input is the enemy" |
| Payment tokens logged in debug logs | **Critical** | Stripe tokens exposed in bunyan logs | Seed 9: "Secrets at rest" |
| Cart updates race condition | **High** | Concurrent PATCH requests can corrupt state | Seed 14: "Partial failures exist" |
| No test coverage on billing endpoints | **High** | 0% coverage on 12 critical routes | Seed 15: "Observability" |
| 14 unused npm dependencies | **Medium** | Bloat; increases attack surface | Seed 12: "Minimalism" |
| Database connection pool not configured | **Medium** | Using defaults; no connection limit | Seed 6: "DB is throughput ceiling" |

## Fix This First (ordered by impact + ease)

1. **10 min** — Add input validation to `/api/checkout` (Zod schema exists in `lib/schemas.ts`, just needs wiring)
2. **5 min** — Disable debug logging in production (ENV check exists, just needs activation)
3. **30 min** — Make cart updates atomic using Prisma `$transaction` (pattern already in `/api/orders`)
4. **20 min** — Configure database pool limits in `src/db/client.ts`
5. **45 min** — Remove 14 unused packages (run `npm audit` then remove each)

## Recommended Next Steps

1. Add `validateCheckout()` to line 42 of `app/api/checkout/route.ts` — pattern matches `app/api/users/route.ts`
2. Wrap cart mutations in `$transaction()` — see example in `app/api/orders/[id]/route.ts`
3. Enable Stripe webhook signature verification (code exists in `lib/stripe.ts` line 67, just commented out)
4. Add test file for checkout flow at `tests/checkout.test.ts` (template exists at `tests/users.test.ts`)
```

This shows users exactly what output to expect and how risks are prioritized.

---

## Constraints (All Modes)

- No modifications to any source file, configuration, or repository metadata
- Do not execute build, test, or deployment commands
- Do not expose secrets, keys, or credentials found in the codebase
- If the repo exceeds ~10,000 files, note the limitation and ask the user whether to scan a specific subdirectory instead

---

## Related Skills

- **code-brain** — Persistent session memory for multi-session projects
  - Pair with: Use code-recon first to understand codebase, then `/code-brain init` to set up session memory
  - Why together: Topology knowledge + session memory = context survival across sessions

---

## Architectural Reference

When assessing architecture quality in Standard, Deep, or Change-Impact mode, read `references/domain-wisdom.md`. It contains 20 production system principles for identifying structural problems like lock contention, wrong-tool usage, and scalability ceilings. Reference the relevant seed number in the Risk Summary table so the user understands not just what the problem is, but why it matters.
