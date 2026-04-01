

# **Causal Ordering & Conflict Model (DLDSS v1.x – v1.1)**

## **Overview**

DLDSS does not implement traditional conflict resolution (e.g., CRDT merges). Instead, it guarantees **global convergence** through:

> **Deterministic ordering of immutable events.**

Conflicts are not “resolved” — they are **preserved, ordered, and replayed identically across all nodes**.

---

## **1. Deterministic Total Ordering**

### **1.1 Event Identity**

Each event is defined as:

```json
{
  "type": "...",
  "payload": { ... },
  "author": "authorId",
  "timestamp": number
}
```

### **1.2 Ordering Function**

All events are sorted using a deterministic comparator:

```
order(a, b):
  if a.timestamp != b.timestamp:
    return a.timestamp - b.timestamp
  else:
    return compare(a.authorId, b.authorId)
```

### **1.3 Guarantee**

* Produces a **total order** over all events
* Ensures **identical replay sequences** across all nodes
* Eliminates nondeterminism from concurrent arrivals

---

## **2. Linearized DAG Model**

Although conceptually a DAG, the MVP operates as a:

> **Deterministically linearized append-only log**

### **Properties**

* Events are appended in arbitrary arrival order
* Replay order is **derived**, not insertion-based
* The DAG can be re-sorted at any time without loss of correctness

---

## **3. Deduplication via Seen Set**

### **3.1 Mechanism**

Each node maintains a `seen` set:

```
seen = Set(eventId)
```

Where `eventId` is typically:

```
hash(type + payload + author + timestamp)
```

### **3.2 Behavior**

* Incoming events are checked against `seen`
* Only unseen events are appended
* Prevents:

  * Duplicate propagation
  * Infinite broadcast loops
  * DAG corruption

---

## **4. Deterministic Replay**

State is always derived as:

```
state = reduce(sort(events))
```

### **Key Implications**

* Arrival order is irrelevant
* Network timing differences are irrelevant
* Replay is **pure and side-effect-free**

---

## **5. Conflict Model**

### **5.1 Definition of Conflict**

A “conflict” is:

> Two or more events that are causally independent but affect overlapping logical state.

### **5.2 Resolution Strategy**

DLDSS does **not merge state**.

Instead:

* All events are preserved
* A deterministic order is imposed
* The reducer defines final state

### **5.3 Example**

```
E1: setName("Alice") @ t=100
E2: setName("Bob")   @ t=100
```

Tie-break:

```
authorId("alice") < authorId("bob")
```

Final order:

```
E1 → E2
```

Final state:

```
name = "Bob"
```

### **Interpretation**

* No ambiguity
* No divergence
* Fully reproducible outcome

---

## **6. Append-Only Semantics**

DLDSS eliminates an entire class of conflicts by design:

* No in-place mutation
* No deletes
* No overwrites

All changes are:

> **New events describing state transitions**

---

## **7. P2P Extension: Causal Metadata (v1.1)**

With cross-device synchronization, true concurrency emerges.

### **7.1 Optional Vector Clock**

Each event may include:

```json
{
  "clock": {
    "nodeA": 5,
    "nodeB": 2
  }
}
```

### **7.2 Purpose**

* Detect causal relationships:

  * **Happened-before**
  * **Concurrent events**
* Enable advanced analysis and debugging

### **7.3 Important Constraint**

Even with vector clocks:

> **Final ordering still uses deterministic tie-breaking**

Vector clocks are **advisory**, not authoritative.

---

## **8. Relationship to Lamport Clocks**

DLDSS can optionally use:

* **Lamport clocks** → scalar causal ordering
* **Vector clocks** → full causal graph

However:

> These enhance observability, not convergence.

Convergence is already guaranteed by:

```
(timestamp, authorId)
```

---

## **9. Design Philosophy**

DLDSS takes a fundamentally different stance than CRDT systems:

| Approach  | Strategy                                      |
| --------- | --------------------------------------------- |
| CRDTs     | Merge concurrent state                        |
| OT        | Transform operations                          |
| **DLDSS** | **Preserve + deterministically order events** |

---

## **10. Key Guarantees**

* **Strong determinism**
* **Global convergence**
* **Replay reproducibility**
* **No merge ambiguity**
* **Minimal algorithmic complexity**

---

## **11. Trade-offs**

### **Advantages**

* Extremely simple model
* Fully auditable
* Easy to implement
* No merge logic required

### **Limitations**

* Last-writer-wins behavior (by ordering)
* No semantic conflict resolution
* Timestamp reliance (can be improved with logical clocks)

---

## **12. Summary**

DLDSS resolves causality not by merging or reconciling state, but by:

> **Transforming concurrency into a deterministic sequence.**

This ensures that:

* Every node sees the same history
* Every node computes the same state
* The system remains simple, verifiable, and reproducible

---
