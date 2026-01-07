---
name: feature-spec
description: Product manager skill for creating feature specifications from high-level requirements. Analyzes user needs, defines scope, and creates clear requirements docs. Use when starting new feature work.
allowed-tools: Read, Grep, Glob, AskUserQuestion
model: inherit
---

# Feature Specification Skill

## Purpose
Transform high-level feature requests into structured specifications.

## Process

**1. Gather Requirements**
- Extract user intent and goals
- Identify success criteria
- Note constraints and dependencies
- Ask clarifying questions if unclear

**2. Define Scope**
- List in-scope functionality
- Explicitly state out-of-scope items
- Identify MVP vs future enhancements
- Flag assumptions needing validation

**3. Create Spec Document**
- User stories or use cases
- Acceptance criteria (testable)
- Non-functional requirements (performance, security, etc.)
- Dependencies and integration points
- Edge cases and error scenarios

**4. Handoff**
- Pass spec to architect agent for technical design
- Request user clarification if requirements ambiguous
- Document open questions clearly

## Output Format

```
# Feature: [Name]

## Goal
[One sentence]

## User Stories
- As a [user], I want [action] so that [benefit]

## Acceptance Criteria
- [ ] Testable criteria 1
- [ ] Testable criteria 2

## Constraints
- Technical limitations
- Business rules
- Security requirements

## Out of Scope
- Explicitly excluded items

## Open Questions
- Items needing clarification
```
