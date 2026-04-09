# DECISION_AGENT.md — RaftLite Project Agent (Production-Ready)

## 1. Project Overview

**Project Name:** RaftLite
**Tagline:** A fault-tolerant, Raft-based distributed key-value store with multi-node replication and AI-driven anomaly detection.

**Mission:** Build a multi-node KV store that guarantees **strong consistency**, survives **node failures and network partitions**, and provides **real-time anomaly detection** on replication logs.

**Non-Goals:**

* Competing with Redis or etcd performance
* Full SQL/database features

---

## 2. Technical Vision

### 2.1 Core System

| Component                  | Language | Responsibility                                                |
| -------------------------- | -------- | ------------------------------------------------------------- |
| Raft Node + KV Store       | Go       | Leader/follower logic, Raft log replication, commit safety    |
| Distributed Pipeline       | Rust     | High-throughput replication log collection, metrics streaming |
| AI Anomaly Detection       | Python   | Detect anomalies in replication patterns and latency          |
| CLI Client                 | Go       | Minimal `PUT/GET/DELETE` commands                             |
| Tests / Failure Simulation | Go       | Node crash, network partition, slow node, concurrency         |

**Design Principles:**

* **Correctness > performance:** All committed entries must replicate to a majority.
* **Simulation > assumptions:** All failure scenarios are tested.
* **Modular architecture:** Easy extension, debugging, and observability.
* **Persistence & snapshotting:** Raft logs persist to disk, with snapshots for memory efficiency.

---

## 3. Core Features

1. **Raft Consensus**

   * Leader election: timeout-based, randomized timers, majority vote
   * Log replication: append-only, followers reject inconsistent entries
   * Commit rule: majority quorum
   * Safety: no split-brain, logs never diverge

2. **KV Store**

   * In-memory + disk-backed persistence
   * Supports `PUT`, `GET`, `DELETE`
   * Snapshotting: periodically compacts logs to disk for memory efficiency

3. **Distributed Pipeline (Rust)**

   * Collects replication logs from all nodes
   * Streams metrics and anomalies to AI module
   * Handles backpressure and network delays

4. **AI Observability (Python)**

   * Real-time anomaly detection (latency spikes, replication inconsistencies)
   * Supports both supervised and unsupervised models
   * Alerts or logs anomalous events

5. **Failure Handling**

   * Node crash → automatic leader election
   * Network partition → quorum enforced, split-brain avoided
   * Slow nodes → delayed replication handled safely, metrics logged

---

## 4. Cross-Language Integration

* **Go ↔ Rust:** gRPC streams for replication logs from Go nodes to Rust pipeline
* **Rust ↔ Python:** REST/gRPC interface or shared Kafka topic for metrics to AI module
* **Deployment:** Docker Compose / Kubernetes for orchestrating multi-language cluster
* **Monitoring:** Central logging service to capture replication events for AI analysis

---

## 5. Edge Cases and Solutions

| Edge Case                             | Solution                                                            |
| ------------------------------------- | ------------------------------------------------------------------- |
| Node crash during partial replication | Leader retries replication; majority quorum ensures commit safety   |
| Network partition                     | Only majority cluster can commit entries; minority waits            |
| Split vote in election                | Randomized timeouts and election retries prevent deadlock           |
| Slow nodes                            | Backpressure and retries; metrics logged for AI detection           |
| Log growth                            | Periodic snapshotting and log compaction                            |
| Concurrent client writes              | Leader serializes writes; followers apply in order                  |
| Node recovery                         | Persisted Raft logs + snapshots enable resynchronization            |
| Pipeline overload                     | Rust handles backpressure; metrics queue buffered, alerts generated |

---

## 6. Implementation Roadmap

| Phase   | Timeline | Deliverables                                                                         |
| ------- | -------- | ------------------------------------------------------------------------------------ |
| Phase 1 | Week 1–2 | Single-node KV store, basic CLI API                                                  |
| Phase 2 | Week 3–4 | Multi-node RPC, heartbeats, followers                                                |
| Phase 3 | Week 5–6 | Leader election, Raft state machine                                                  |
| Phase 4 | Week 7–8 | Log replication, commit logic, snapshotting, state machine                           |
| Phase 5 | Week 9   | Failure simulations (node crash, partition, slow nodes, concurrency)                 |
| Phase 6 | Week 10+ | Rust pipeline, Python AI anomaly detection, optimizations, monitoring, documentation |

---

## 7. Success Metrics

* Cluster survives leader crash and network partitions without data loss
* Logs replicate consistently across all nodes
* AI pipeline detects anomalies in real time
* Minimal CLI fully functional (`PUT/GET/DELETE`)
* System is modular, testable, and maintainable

---

## 8. Repository Structure

```text
raftlite/
├── cmd/            # CLI entrypoints
├── raft/           # Core consensus logic (Go)
├── storage/        # KV store + snapshots (Go)
├── rpc/            # Communication layer (Go, gRPC)
├── pipeline/       # Distributed replication log processing (Rust)
├── ai/             # Anomaly detection (Python)
├── client/         # CLI client (Go)
├── tests/          # Failure simulations & concurrency tests
├── docs/           # Architecture diagrams & explanations
└── DECISION_AGENT.md
```

---

## 9. Risk Assessment & Mitigation

* **Concurrency bugs:** Automated tests for leader election, replication, multi-client writes
* **Split-brain:** Enforced majority quorum; partitions cannot commit
* **Memory / Log growth:** Snapshotting and log compaction
* **Integration complexity:** Clear gRPC interfaces, Docker/K8s orchestration
* **AI false positives:** Tuned thresholds, test datasets, metrics validation
* **Scalability:** Raft limited to ~7 nodes; pipeline designed for horizontal extension

---

## 10. Open Source / Iteration Strategy

* **v0.1:** Leader election + basic replication, CLI API
* **v0.2:** Full log replication, commit safety, snapshotting, multi-node stability
* **v0.3:** Rust pipeline + Python AI anomaly detection, failure simulation, monitoring, full documentation

**Documentation Requirements:**

* Original Raft explanation
* Architecture diagrams
* Steps to run 3-node cluster
* Demo failure scenarios

