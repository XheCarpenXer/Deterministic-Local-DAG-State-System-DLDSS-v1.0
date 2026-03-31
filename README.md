
# **Deterministic Local DAG State System (DLDSS) v1.0**

### *A Serverless, Multi-Context State Synchronization Architecture Using Append-Only Event DAGs*

---

## **Abstract**

This paper introduces the **Deterministic Local DAG State System (DLDSS)**, a client-native architecture for building applications with **shared, deterministic state across multiple execution contexts (e.g., browser tabs)** without requiring networking, servers, or peer-to-peer protocols.

The system models all application state as an **append-only Directed Acyclic Graph (DAG) of events**, persisted locally and synchronized across contexts using **intra-environment broadcast mechanisms**. Determinism is achieved through ordered event application and immutable state transitions, ensuring that all clients converge to the same state given identical inputs.

This document establishes prior art for:

* Local-first deterministic state replication
* DAG-based application state modeling
* BroadcastChannel-based intra-client synchronization
* Serverless multi-context consensus via shared storage

---

## **1. Introduction**

Modern distributed systems rely heavily on servers or peer-to-peer overlays to maintain shared state. However, many applications operate within a **single device across multiple execution contexts** (e.g., browser tabs, workers).

DLDSS proposes that:

> **Consensus within a device does not require networking—only deterministic event ordering and shared persistence.**

This system eliminates:

* Network transport layers
* External synchronization protocols
* Centralized coordination

Instead, it leverages:

* Local storage as a persistence layer
* Broadcast mechanisms for propagation
* Deterministic DAG replay for state reconstruction

---

## **2. Core Principles**

### **2.1 Determinism**

All application state is derived solely from an ordered sequence of events. Given the same event set, all clients compute identical state.

### **2.2 Append-Only Log (DAG)**

Events are immutable and only appended. No mutation or deletion occurs.

### **2.3 Local-First Persistence**

State is stored entirely in client-controlled storage (e.g., localStorage, IndexedDB).

### **2.4 Multi-Context Synchronization**

All execution contexts on the same device synchronize via broadcast primitives.

### **2.5 No Networking Requirement**

The system operates entirely offline and does not require TCP/IP, WebRTC, or external services.

---

## **3. System Architecture**

### **3.1 Components**

#### **Event DAG**

A linearized DAG (for MVP) representing all state transitions:

```
event_1 → event_2 → event_3 → ...
```

Each event contains:

* Type
* Payload
* Author
* Timestamp

#### **Event Creator**

Generates structured events from user actions.

#### **Persistence Layer**

Stores the full event log locally:

* MVP: localStorage
* Scalable: IndexedDB

#### **Synchronization Layer**

Uses **BroadcastChannel** to propagate new events across contexts.

#### **Renderer (UI)**

Derives view state by replaying the DAG.

---

## **4. Data Model**

### **4.1 Event Structure**

```
{
  type: string,
  payload: object,
  author: string,
  timestamp: number
}
```

### **4.2 DAG Properties**

* Directed: events reference causal order implicitly
* Acyclic: no event can depend on a future event
* Append-only: ensures immutability and replayability

---

## **5. Execution Model**

### **5.1 Event Lifecycle**

1. User action occurs
2. Event is created
3. Event is appended to DAG
4. DAG is persisted locally
5. Event is broadcast to other contexts
6. Receiving contexts append and re-render

---

### **5.2 Deterministic Replay**

State is reconstructed via:

```
state = reduce(events)
```

Because:

* Events are immutable
* Order is preserved
* No side effects exist outside event application

All clients converge to identical state.

---

## **6. Multi-Context Synchronization**

### **6.1 BroadcastChannel Mechanism**

Each client subscribes to a shared channel:

```
channel = new BroadcastChannel('dag-sync')
```

#### **On Event Creation**

* Append locally
* Persist
* Broadcast event

#### **On Event Reception**

* Append received event
* Persist updated DAG
* Trigger UI update

---

### **6.2 Convergence Guarantee**

Given:

* Reliable local broadcast delivery
* Deterministic append logic

Then:

> All active contexts will converge to identical DAG state.

---

## **7. Implementation Reference (MVP)**

### **7.1 DAG Core**

```
class DAG {
  constructor() {
    this.events = [];
  }

  append(event) {
    this.events.push(event);
  }

  getAll() {
    return this.events;
  }
}
```

---

### **7.2 Persistence + Sync**

```
const channel = new BroadcastChannel('dag-sync');

function addEvent(event) {
  dag.append(event);
  localStorage.setItem('dag', JSON.stringify(dag.getAll()));
  channel.postMessage(event);
}

channel.onmessage = (msg) => {
  dag.append(msg.data);
  localStorage.setItem('dag', JSON.stringify(dag.getAll()));
  notifyUpdate();
};
```

---

### **7.3 UI Model**

* Reads full DAG
* Renders derived state
* Updates on broadcast events

---

## **8. Comparison to Existing Systems**

| System Type         | Networking | Determinism | Local-First | DAG-Based |
| ------------------- | ---------- | ----------- | ----------- | --------- |
| Traditional Web App | Required   | No          | No          | No        |
| CRDT Systems        | Optional   | Partial     | Yes         | Sometimes |
| Blockchain          | Required   | Yes         | No          | Yes       |
| **DLDSS**           | **None**   | **Yes**     | **Yes**     | **Yes**   |

---

## **9. Novel Contributions (Prior Art Claims)**

This system establishes prior art for:

### **9.1 Serverless Deterministic State Sharing**

Using only local storage and broadcast primitives.

### **9.2 Intra-Device Consensus Model**

Consensus achieved across browser contexts without networking.

### **9.3 DAG as Primary State Engine**

Application state defined entirely by an append-only DAG.

### **9.4 BroadcastChannel as State Bus**

Using browser-native messaging as a synchronization backbone.

### **9.5 Replay-Based UI Architecture**

UI derived exclusively from deterministic event replay.

---

## **10. Limitations**

* No cross-device synchronization (by design in MVP)
* Event ordering relies on local timing (can be enhanced with logical clocks)
* Storage constraints in localStorage
* No conflict resolution beyond append order

---

## **11. Future Extensions**

### **11.1 IndexedDB Backend**

For large-scale DAG storage

### **11.2 Cryptographic Event Signing**

For verifiability and trust

### **11.3 Causal Ordering (Vector Clocks)**

To extend beyond linear append

### **11.4 P2P / IPFS Integration**

For cross-device replication

### **11.5 Access Control Layers**

Capability-based permissions

---

## **12. Conclusion**

DLDSS demonstrates that:

> **Deterministic shared state can be achieved entirely within a client environment without servers, networking, or complex consensus protocols.**

By combining:

* Append-only DAG structures
* Local-first persistence
* Broadcast-based synchronization

the system provides a minimal yet powerful foundation for building **fully deterministic, multi-context applications**.

---

## **13. Prior Art Declaration**

This document serves as a **public disclosure of system design and implementation approach**, establishing prior art for:

* Deterministic DAG-based local state systems
* BroadcastChannel-driven synchronization architectures
* Serverless multi-context consensus models

---

## 14. P2P Extension (v1.1 Update)

### 14.1 Overview

The Deterministic Local DAG State System (DLDSS) now supports **peer-to-peer synchronization across devices**, while retaining full backward compatibility with the existing DAG and intra-device BroadcastChannel mechanisms. This extension introduces a **fourth transport** alongside localStorage and BroadcastChannel, enabling cross-device state replication **without centralized servers**.

### 14.2 Key Architectural Decisions

- **Single Unified Write Path:** `dispatchEvent()` now appends events to the DAG, persists to localStorage, broadcasts to other tabs, and sends to connected peers via P2P (`p2pBroadcast()`). One call ensures **deterministic propagation across all transports**.  
- **Single Unified Read Path:** `receiveEvent()` handles all incoming events, whether from other tabs (BroadcastChannel) or remote peers (WebRTC DataChannel). Events are **deduplicated, appended, persisted, and relayed automatically**.  
- **Deduplication:** Uses a pre-seeded `seen` Set from the loaded DAG to prevent duplicates when receiving full DAG syncs from peers.  
- **Manual, Serverless Signaling:** Peer connections use a three-step manual flow: generate offer → send out-of-band → paste answer back. No TURN server is required on LAN; optional STUN servers handle NAT traversal on the open internet.  
- **Event Origin Transparency:** Each message includes a small badge indicating its origin: local, BroadcastChannel (`bc`), or P2P. This aids debugging and auditing.  
- **Deterministic Ordering:** After a full DAG sync from a peer, `dag.sort()` restores ordering by `[timestamp, authorId]` as defined in the original specification.

### 14.3 Benefits

- Extends DLDSS from **single-device, multi-tab synchronization** to **multi-device peer collaboration**.  
- Maintains **full determinism, conflict-free state, and reproducibility** across all peers.  
- Fully **serverless and decentralized**, enabling offline-first applications.  
- Supports **auditable, transparent event origins** for each cross-device message.

### 14.4 Implementation Notes

- P2P layer is **additive**, leaving all existing code paths and DAG logic unchanged.  
- Manual signaling ensures **no dependency on external servers**, protecting privacy and sovereignty.  
- Can be integrated with **higher-level networking protocols or encrypted overlays** for future extensions (e.g., IPFS, FHE-secured replication).
