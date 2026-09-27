# 1. Record architecture decisions

Date: 2026-09-27
Status: Accepted

## Context
We need a consistent way to record significant technical decisions as they're
made, so the reasoning ("why X over Y") is preserved rather than lost.

## Decision
Keep Architecture Decision Records as numbered Markdown files in `docs/adr/`.
One decision per file: context, options considered, the choice, consequences.
Written at the moment of the decision; immutable once accepted. A reversal is a
new ADR that supersedes the old one.

## Consequences
- Decisions and rationale live in Git history.
- Small writing cost at each decision point.
- Anyone (including an interviewer) can trace why the platform looks as it does.