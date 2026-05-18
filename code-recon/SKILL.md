---
name: code-recon
description: "Maps any codebase before you touch it. Use when starting on unfamiliar code, before a risky change, or when you need to understand what's connected to what. Triggers on: 'map this repo', 'code audit', 'quick scan', 'deep scan', 'what breaks if I change X', 'blast radius of this change', 'what depends on this file', 'recon diff', 'what changed since last scan', 'analyze this project', 'give me a topology report'. Also triggers automatically before modifying an unfamiliar file mid-session."
license: MIT
metadata:
  author: thisiskishor
  version: 2.0.0
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

## Constraints (All Modes)

- No modifications to any source file, configuration, or repository metadata
- Do not execute build, test, or deployment commands
- Do not expose secrets, keys, or credentials found in the codebase
- If the repo exceeds ~10,000 files, note the limitation and ask the user whether to scan a specific subdirectory instead

---

## Architectural Reference

When assessing architecture quality in Standard, Deep, or Change-Impact mode, read `references/domain-wisdom.md`. It contains 20 production system principles for identifying structural problems like lock contention, wrong-tool usage, and scalability ceilings. Reference the relevant seed number in the Risk Summary table so the user understands not just what the problem is, but why it matters.
