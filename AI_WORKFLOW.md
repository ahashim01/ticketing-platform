# AI Workflow Agreement — ticketing-platform

_Agreed at D003 (2026-11-21). Reviewed at every Weekly Review._

How AI agents (Claude Code etc.) work in this repo, and what I keep for myself.
I own every line that gets merged, whoever typed it.

---

## 1. Two modes, chosen by stage

| | Learning mode | Production-simulation mode |
|---|---|---|
| **Who goes first** | I attempt first | I define problem, constraints, architecture |
| **AI's role** | Probes, hints, reviews. Never writes my first attempt. Hints before solutions. | Implements, tests, refactors, reviews |
| **My role** | Write it, then defend it | Validate correctness, security, performance |

**Rule:** anything that is the *subject of the current stage* is Learning mode, no
exceptions. The mode is decided by the curriculum, not by how tired I am.

- **Learning mode (I write it):** whatever the current stage teaches. In Stage 1 that
  means the reservation schema, the conditional `UPDATE`, locking and isolation choices,
  the overselling fix.
- **Delegable to agents:** work that teaches me nothing new in this stage, for example
  FastAPI project scaffolding, docker-compose for Postgres/Redis, load-test harness
  boilerplate, CI configuration, fixtures and seed data.

## 2. My tests are the spec

1. I write (or explicitly approve) the tests that define "correct", and commit them
   **failing, in their own commit**, before the agent starts.
2. The agent writes the implementation until those tests pass.
3. The agent **may add** tests. It **may not change or delete** my tests without my review.
   Any diff touching my tests is reviewed line by line.

## 3. Every delegation uses this brief

```
Goal:          what outcome, in one sentence
Constraints:   stack, patterns, things it must not do
In scope:      files/directories it may touch
Out of scope:  what it must leave alone
Done when:     the exact command that must pass (e.g. `pytest tests/reservations -q`)
```

- One task = one branch.
- Keep diffs reviewable: about 300 changed lines max. Bigger → split the task.

## 4. Context lives in files, not in chat

- `CLAUDE.md` at the root points agents to `docs/architecture-v0.md` and this file.
- System invariants are written down once (in `CLAUDE.md`) and every agent works
  against them. They are the same invariants I am tested on.

## 5. Sandbox and permissions

- Agents work on **branches only**. They never push to `main` and never deploy.
- **No production credentials** and no `.env` with real secrets. Only local/test values.
- Pre-approved commands: running tests, lint and formatting only.
- Everything else needs my approval: installing packages, running migrations, network
  access, deleting files.

## 6. Validation before merge

### Automated (CI + pre-commit), so I never check these by hand
- Lint and formatting
- Unit and integration tests
- Secret scanning (no hard-coded secrets)

### Human checklist (what CI can't see)
1. I have read and understood every line. If I can't explain it, it doesn't merge.
2. **Concurrency:** what happens with two buyers and one seat? Is there a test for it?
3. **Failure paths:** retries, timeouts, partial failure. Is the invariant still safe?
4. **Authorization:** can user A act on user B's reservation or order?
5. **Migrations:** reversible, and safe to run against live data?
6. **Observability:** logs/metrics exist on the risky path.
7. Follows the project's conventions.

**Merge rule:** agent-written code merges only after CI is green **and** every item above
is checked.

## 7. What stays human-only

- **Deploying and pushing to `main`.** Nothing ships without my review, my quality check
  and my explicit approval.

## 8. Record what I validated

Every agent PR description ends with one line:

```
Validated: <what I checked> — <how> (e.g. "concurrency: ran 200 parallel reserve calls, 0 oversold")
```

These lines become entries in my Engineering Evidence Portfolio.

---

## Open challenge (revisit at D005 Weekly Review)

- Deployment approval is an operational gate. Should correctness decisions also be
  human-only: seat-state invariants, data ownership, transaction boundaries?