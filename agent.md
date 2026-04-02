# 📄 Distributed KV Store (Raft)

## Decision + Architecture + Execution Document

---

# 1. PROJECT NAME

## **RaftLite**

**Tagline:**

> A minimal, fault-tolerant distributed key-value store built on Raft consensus.

Alternative (if you want stronger branding):

## **ConsenStore**

> Consensus-driven distributed KV store

Pick one and stick with it. Don’t rename later.

---

# 2. PROBLEM DEFINITION

Single-node databases fail:

* data loss on crash
* no availability
* no consistency across replicas

### Real Problem:

> How do multiple nodes agree on a consistent state despite failures?

This is the **core distributed systems problem**.

---

# 3. OBJECTIVE

Build a system that:

1. Stores key-value data
2. Replicates state across nodes
3. Maintains consistency using Raft
4. Survives node failures

---

# 4. NON-GOALS (CRITICAL)

Do NOT try to:

* compete with Redis
* match etcd performance
* build full SQL/database features

Focus:

> correctness + consensus, not features

---

# 5. CORE CONCEPT (UNDERSTAND THIS OR FAIL)

## Raft guarantees:

* One leader at a time
* Log replication
* Majority agreement

If you don’t deeply understand this, stop and study first.

---

# 6. SYSTEM ARCHITECTURE

## High-Level

```id="k6f4a1"
[ Client ]
     ↓
[ Leader Node ]
     ↓
[ Follower Nodes ]
     ↓
[ Replicated Log ]
     ↓
[ State Machine (KV Store) ]
```

---

## Components

### 6.1 Node

Each node contains:

* Raft state
* Log
* KV store
* RPC server

---

### 6.2 Leader

Responsibilities:

* Accept client requests
* Append to log
* Replicate to followers

---

### 6.3 Followers

Responsibilities:

* Receive log entries
* Apply them in order

---

### 6.4 State Machine

* Applies committed log entries
* Stores:

  * key → value

---

# 7. RAFT IMPLEMENTATION (NON-NEGOTIABLE)

---

## 7.1 Leader Election

* Timeout-based election
* Randomized timers
* Majority vote wins

Failure case:

* split vote → retry

---

## 7.2 Log Replication

Leader:

* sends AppendEntries RPC
  Followers:
* accept if consistent

---

## 7.3 Safety

* Logs must not diverge
* Followers reject inconsistent entries

---

## 7.4 Commit Rule

Entry is committed when:

* replicated on majority of nodes

---

# 8. CLIENT API

Keep it minimal:

```id="a1x8kq"
PUT key value
GET key
DELETE key
```

Leader handles all writes.

---

# 9. FAILURE HANDLING (THIS DEFINES QUALITY)

You must simulate:

### Node crash

* kill leader
* ensure new leader election

### Network partition

* split cluster
* ensure no split-brain

### Slow nodes

* delayed replication

If you skip this → project is fake.

---

# 10. IMPLEMENTATION PLAN

---

## Phase 1 (Week 1–2)

* Single-node KV store
* Basic API

---

## Phase 2 (Week 3–4)

* Multi-node communication (RPC)
* Heartbeats

---

## Phase 3 (Week 5–6)

* Leader election
* Basic Raft state

---

## Phase 4 (Week 7–8)

* Log replication
* Commit logic

---

## Phase 5 (Week 9)

* Failure simulation

---

## Phase 6 (Week 10+)

* Optimization + cleanup

---

# 11. TECH STACK

Pick ONE:

### Option A (recommended)

* Go (best for concurrency + networking)

### Option B

* Python (faster dev, lower performance)

---

# 12. REPO STRUCTURE

```id="d9m2p0"
raftlite/
├── cmd/            # entrypoints
├── raft/           # core consensus logic
├── storage/        # KV store
├── rpc/            # communication layer
├── client/         # CLI client
├── tests/          # failure scenarios
├── docs/
└── DECISION_AGENT.md
```

---

# 13. OPEN SOURCE STRATEGY

## v0.1 (publish here)

* leader election works
* basic replication

## v0.2

* full log replication
* stable under node failure

## v0.3

* failure testing + docs

---

## README MUST INCLUDE

* Raft explanation (brief, not copied)
* architecture diagram
* how to run 3-node cluster
* demo steps

---

# 14. SUCCESS METRICS

Your system works if:

* cluster survives leader crash
* no data inconsistency
* logs replicate correctly

---

# 15. COMMON FAILURE PATTERNS (YOU WILL HIT THESE)

### 1. Race conditions

→ concurrency bugs everywhere

---

### 2. Broken elections

→ multiple leaders

---

### 3. Log inconsistency

→ hardest bug class

---

### 4. Debugging hell

→ distributed bugs are non-linear

---

# 16. CONTRARIAN TRUTH

Most people:

* “implement Raft”
* but don’t test failure

That’s worthless.

---

# 17. FINAL PRINCIPLES

* correctness > speed
* simulation > assumptions
* logs > intuition (debug everything)

---

# FINAL REALITY CHECK

This project is:

* harder than anything you’ve built
* slower than you expect
* more frustrating than you think

But:

> If you finish it properly, it’s one of the strongest signals you can produce.

