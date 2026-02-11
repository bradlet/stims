# breaking-upgrade-investigation

Investigates breaking changes between dependency versions and assesses their impact on your codebase. Assumes the dependency has releases published on GitHub, and parallelizes research across major version boundaries to produce a risk-scored HTML report.

## Contents

| File | Role | Model | Tools |
|------|------|-------|-------|
| `skills/investigate-upgrade/SKILL.md` | **Orchestrator** — gathers dependency info, identifies major versions via `gh`, dispatches agents, and compiles a color-coded HTML report | inherit | All |
| `agents/breaking-change-investigator.md` | **Researcher** — examines GitHub releases, changelogs, and online resources to catalog breaking changes for assigned versions | inherit | Bash, WebFetch, WebSearch, Read, Grep, Glob |
| `agents/dependency-usage-expert.md` | **Impact Analyst** — maps breaking changes against actual codebase usage and assigns risk scores (1-10) with evidence | inherit | Read, Grep, Glob, Bash |

## Interaction flow

```
User requests a dependency upgrade investigation
        │
        ▼
┌─── GATHER INFO (Skill) ──────────────┐
│  Collect dependency name, repo,       │
│  current version, tag convention,     │
│  optional target version              │
└──────────────┬────────────────────────┘
               ▼
┌─── IDENTIFY VERSIONS (Skill) ────────┐
│  Use gh api to find all major         │
│  releases (X.0.0) between current    │
│  and target versions                  │
└──────────────┬────────────────────────┘
               ▼
┌─── SCAN CODEBASE (Skill) ────────────┐
│  Map how the dependency is used,      │
│  build usage context summary          │
└──────────────┬────────────────────────┘
               ▼
┌─── RESEARCH (Breaking Change Investigators ×3) ──┐
│  Up to 3 parallel agents, each given a subset    │
│  of major versions to investigate for breaking   │
│  changes via releases, changelogs, web search    │
└──────────────┬───────────────────────────────────┘
               ▼
┌─── ASSESS (Dependency Usage Expert) ─┐
│  Score each breaking change 1-10     │
│  against actual codebase usage,      │
│  identify affected files             │
└──────────────┬────────────────────────┘
               ▼
┌─── REPORT (Skill) ───────────────────┐
│  Generate color-coded HTML report    │
│  sorted by risk: high → medium → low │
└──────────────────────────────────────┘
```

The skill orchestrates all agents and produces a single `upgrade-report-{dependency}.html` file with a color-coded table — red for high risk (7-10), orange for medium (4-6), and green for low (1-3).
