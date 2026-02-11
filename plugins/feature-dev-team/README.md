# feature-dev-team

This plugin exists as an experiment in how injecting personality traits into agent behavior can impact accuracy and performance. It is orthogonal to Anthropic's own [feature-dev plugin](https://github.com/anthropics/claude-code/blob/main/plugins/feature-dev/README.md) and was made without awareness of that core plugin's existence.

## Contents

| File | Role | Model | Tools |
|------|------|-------|-------|
| `skills/feature-spec/SKILL.md` | **Product Manager** — transforms a feature request into a structured spec | inherit | Read, Grep, Glob, AskUserQuestion |
| `agents/architect.md` | **Architect** — translates the spec into a technical design; serves as the final validation gate | Sonnet | Read, Grep, Glob, AskUserQuestion, `feature-spec` skill |
| `agents/senior-engineer.md` | **Senior Engineer** — skeptical reviewer who performs risk assessment, provides implementation guidance, and gates code quality | Sonnet | Read, Grep, Glob, Bash, AskUserQuestion |
| `agents/ic-engineer.md` | **IC Engineer** — mid-level implementer focused on simplicity, DRY code, and low cyclomatic complexity | Sonnet | Read, Write, Edit, Grep, Glob, Bash |

## Interaction flow

```
User provides a feature request
        │
        ▼
┌─── SPEC (Product Manager skill) ───┐
│  Gather requirements, define scope, │
│  produce structured spec document   │
└──────────────┬──────────────────────┘
               ▼
┌─── DESIGN (Architect) ─────────────┐
│  Clarify ambiguities with user,     │
│  review existing architecture,      │
│  produce technical spec             │
└──────────────┬──────────────────────┘
               ▼
┌─── GUIDANCE (Senior Engineer) ─────┐
│  Analyze affected systems,          │
│  assess risks, break work into      │
│  tasks with gotchas & test strategy │
└──────────────┬──────────────────────┘
               ▼
┌─── IMPLEMENT (IC Engineer) ────────┐
│  Write code & tests following       │
│  senior's guidance                  │
└──────────────┬──────────────────────┘
               ▼
┌─── CODE REVIEW (Senior Engineer) ──┐
│  PASS → forward to architect        │
│  FAIL → return to IC with feedback  │◄─── IC iterates
└──────────────┬──────────────────────┘
               ▼
┌─── FINAL REVIEW (Architect) ───────┐
│  APPROVE → done                     │
│  REVISE  → return with issues       │
└─────────────────────────────────────┘
```

Each agent only has access to the tools it needs, and hand-offs between agents use defined communication protocols. The senior engineer and architect act as sequential quality gates — code must pass both before the feature is considered complete.
