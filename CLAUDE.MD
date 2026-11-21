# CLAUDE.md — ticketing-platform

High-traffic ticket / commerce platform. Learning capstone: modular monolith first,
services split only on measured evidence.

## Read before any task
- `AI_WORKFLOW.md`: how agents work here (modes, test rules, permissions). Follow it.
- `docs/architecture-v0.md`: requirements, module boundaries, risks.

## System invariants (never violate)
1. A seat is never sold twice.
2. The database is the authority for seat state (available / held / sold) and hold
   expiry. Redis is a cache, never the source of truth.

## Rules
- Work on a branch. Never push to `main`. Never deploy.
- Do not modify or delete existing tests without explicit approval.
- No real secrets; local/test values only.
- If a task touches the current learning stage's core topic, stop and ask: it may be
  Learning mode (see `AI_WORKFLOW.md` §1).