# Claude Code Skills - Codebase Topology and Recon

Two tools for Claude Code that make it a careful engineering partner - one that maps the territory before touching anything, instead of diving straight into writing code.

---

## What's Included

### 1. `CLAUDE.md` - Always-On Behavioral Guidelines

A global instruction file that shapes how Claude approaches every coding task. Install it once and it applies automatically, no need to ask for it.

It teaches Claude to:
- Detect ambiguity before touching code and confirm understanding first
- Map the relevant codebase topology before writing anything
- Apply 4 structural checks to every change: state ownership, observability, blast radius, and timing
- Know when to stop and flag instead of proceeding blind
- Move fast but surgically - no sprawling refactors, no touching unrelated files

### 2. `code-recon/` - Codebase Reconnaissance Skill

A skill that performs a read-only, non-intrusive scan of any repository and produces a structured topology report. Use it at the start of any new project, before a risky change, or whenever you need to understand what is connected to what.

**Five scan modes:**

| Mode | What it does | Time |
|---|---|---|
| Quick | Stack, entry points, immediate red flags | ~2 min |
| Standard | Full topology report - endpoints, data flow, security, dead code, risk table | ~10 min |
| Deep | Everything in Standard plus input-to-DB tracing, coverage matrix, async hazard detection | ~20 min |
| Change-Impact | Blast radius analysis for a specific file or function | ~5 min |
| Recon Diff | Compare current codebase against a previous scan | ~5 min |

The skill also bundles `references/domain-wisdom.md` - 20 scalability and architecture principles drawn from production systems experience, used to identify structural problems like lock contention, wrong-tool usage, and scalability ceilings. Risk findings in the report link directly to the relevant seed so you know not just what the problem is, but why it matters.

---

## Install

### Option A - Download as ZIP (easiest)

1. Click the green **Code** button at the top of this page
2. Click **Download ZIP**
3. Extract the ZIP file
4. Copy the `code-recon` folder into your Claude skills directory:

**Mac/Linux:**
```bash
cp -r code-recon ~/.claude/skills/skills/engineering/
```

**Windows (PowerShell):**
```powershell
Copy-Item -Recurse code-recon "$env:USERPROFILE\.claude\skills\skills\engineering\"
```

> If the `engineering` folder does not exist yet, create it first:
> Mac/Linux: `mkdir -p ~/.claude/skills/skills/engineering`
> Windows: `New-Item -ItemType Directory "$env:USERPROFILE\.claude\skills\skills\engineering" -Force`

### Option B - Clone the repo

```bash
git clone https://github.com/thisiskishor/claude-code-recon-skill.git
cp -r claude-code-recon-skill/code-recon ~/.claude/skills/skills/engineering/
```

That is it. The skill loads automatically - no restart needed.

---

## Install the CLAUDE.md Guidelines (Optional but Recommended)

This file makes Claude more careful on every task, not just recon scans.

**If you do not have a global `~/.claude/CLAUDE.md` yet:**

Mac/Linux:
```bash
cp CLAUDE.md ~/.claude/CLAUDE.md
```

Windows:
```powershell
Copy-Item CLAUDE.md "$env:USERPROFILE\.claude\CLAUDE.md"
```

**If you already have one**, open both files and paste the contents of this `CLAUDE.md` at the bottom of yours. It will not conflict - it adds to what is already there.

---

## How to Use It

Once installed, just talk to Claude normally. The skill triggers automatically based on what you say.

**Trigger examples:**
- `"map this repo before we touch anything"` - runs Standard scan
- `"quick scan"` - runs Quick scan
- `"deep scan, I need to understand everything"` - runs Deep scan
- `"what breaks if I change this file?"` - runs Change-Impact mode
- `"what changed since the last report?"` - runs Recon Diff mode
- `"do a code audit"` - runs Standard scan
- `"blast radius of this function"` - runs Change-Impact mode

---

## What the Report Looks Like

Standard scan output structure:

```
# Topology Report: [Your Project]

## Executive Summary
## Technology Stack
## Directory Structure
## Services and Entry Points
## API Surface
## Data Flow
## Security Boundaries
## Test Coverage
## Dead and Unused Code
## Risk Summary (with architecture seed references)
## Fix This First
## Recommended Next Steps
```

See [`examples/topology-report-example.md`](examples/topology-report-example.md) for a full sample output on a real project.

---

## The 4 Invariables

Every non-trivial change filtered through `CLAUDE.md` gets checked against these four questions:

| Question | Maps To | Why It Matters |
|---|---|---|
| Where does state live? | Ownership and truth | Consistency, blast radius |
| Where does feedback live? | Observability | Debugging, monitoring |
| What breaks if I delete this? | Coupling and fragility | Safe refactoring |
| When does timing work? | Async and ordering | Race conditions, correctness |

If any are unclear, Claude stops and asks instead of guessing.

---

## Files

```
.
├── README.md
├── CLAUDE.md                          - Global behavioral guidelines
├── CHANGELOG.md
├── CONTRIBUTING.md
├── examples/
│   └── topology-report-example.md    - Sample output from the skill
└── code-recon/
    ├── SKILL.md                       - The skill definition
    └── references/
        └── domain-wisdom.md           - 20 scalability architecture seeds
```

---

## License

MIT - use freely, modify, share.

---

## Author

[thisiskishor](https://github.com/thisiskishor)
