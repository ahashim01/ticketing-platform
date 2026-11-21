# Building Microservices 2e — Ch 1: What Are Microservices?

_Recall note: dictated from memory, book closed (D003, 2026-11-21), then transcribed.
Read so far: Ch 1 up to, not including, "Microservice Pain Points"._

## 3 ideas (my recall)

1. **Technology heterogeneity.** Each service can pick its own technology and adopt new
   advancements without betting the whole system on them.
   _Capstone link:_ this is the argument for rewriting only the reservation/checkout hot
   path in Go (Stage 2), not the whole platform.
2. **Robustness through isolation.** If one feature/service has a problem, it can be
   isolated instead of taking down the whole application, as can happen in a monolith.
3. **Modular monolith as the default starting point.** Delivers fast, avoids the cost of
   network calls and inter-service communication, is better structured than a
   single-process monolith, and keeps a path open to extract services later.

## 1 claim I want to test

Splitting into services when two services need to change the same state (e.g. seat
hold → payment → seat sold): how do we keep that atomic once it is no longer one database
transaction, and how do we test it?

- How I'd test it: an integration test for the failure, plus logs/monitoring to watch the
  database changes and the error.
- _Mentor review:_ this names tools, not a test yet. A failure test needs three parts:
  the **invariant** that must hold (what may never happen), the **fault** injected, and the
  **assertion** that checks the invariant. Carried as Reinforcement debt.

## 1 connection (sanitized)

At work (fintech product, modular monolith), integrating a new external provider meant
adding a new provider module behind an existing, stable provider interface. Core code did
not need restructuring. Moving fast with changing business needs was cheap because the
module interface was stable, not because of how the code was deployed.

## Mentor review — corrections after recall

- **Independent deployability → no shared DB.** The causal chain: a shared database makes
  the schema a public contract. Any schema change then forces coordinated (lockstep)
  deploys of every consumer, so independent deployability is gone. That is information
  hiding, not just "loose coupling".
- **Distributed monolith** = the *disadvantages* of a single-process monolith plus the
  disadvantages of a distributed system, without enough upside of either. Typical cause:
  boundaries not aligned to the domain, so changes span services and must ship together.
- **Conway's law, direction matters:** the system ends up mirroring the organization's
  communication structure. The deliberate move is the reverse: shape teams around the
  architecture you want (business-domain teams → business-domain services).
- **Robustness is not automatic.** A distributed system adds new failure modes (network,
  timeouts, partial failure). Isolation only exists if it is designed in. A down service
  that sits on the synchronous checkout path still takes checkout down.
- **"Easy to split later" has a precondition:** each module owns its tables, and no other
  module reads or joins them directly. Without that, it is a distributed monolith waiting
  to happen.
- **Two different problems, don't mix them:**
  - *Row contention* (a scaling problem): thousands of buyers fighting over the same seat
    rows serialize on row locks. More app servers, or more services, do not help;
    Inventory still owns those rows in one place.
  - *Cross-service consistency* (a correctness problem): once hold and payment live in
    different services, there is no single transaction. This is one of Newman's own pain
    points (data consistency), so it is not a disagreement with the book.
- Not read yet: "Microservice Pain Points" → finish before / at D005.