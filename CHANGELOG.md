# Changelog

---

## [2.0.0] - 2026-05-18

### Added

- **Five scan modes** instead of one fixed behavior:
  - `Quick` (~2 min) - stack, entry points, immediate red flags only
  - `Standard` (default, ~10 min) - full topology report
  - `Deep` (~20 min) - everything in Standard plus input-to-DB path tracing, coverage-to-endpoint matrix, dependency depth analysis, async hazard detection, and full architecture assessment against all 20 domain-wisdom seeds
  - `Change-Impact` (~5 min) - blast radius analysis for a specific file or function, rates impact as Contained / Moderate / Wide / Unknown
  - `Recon Diff` (~5 min) - compares current codebase against a previous scan and classifies changes as new, removed, modified, regression, or improvement

- **Pre-task checkpoint** - skill now triggers automatically before modifying an unfamiliar file mid-session, not only when explicitly invoked

- **Richer risk output** - Risk Summary table now includes a Seed column linking each risk to the relevant principle in `domain-wisdom.md`, plus a "Fix This First" priority list ordered by combined impact and ease

- **Zip install support** - users can now download the repo as a ZIP and copy the `code-recon` folder directly into their Claude skills directory

### Changed

- Step descriptions in each scan mode significantly expanded with more specific guidance on what to look for and why

---

## [1.0.0] - 2026-05-18

### Added

- `code-recon` skill - read-only codebase topology mapping for Claude Code
- `references/domain-wisdom.md` - 20 scalability architecture seeds bundled inside the skill
- `CLAUDE.md` - global always-on behavioral guidelines for Claude Code
  - Ambiguity detection protocol
  - Topology navigation discipline
  - The 4 invariables (state, feedback, blast radius, timing)
  - Verification gate before writing code
  - Red lines and commit decision framework
  - Execution momentum rules

