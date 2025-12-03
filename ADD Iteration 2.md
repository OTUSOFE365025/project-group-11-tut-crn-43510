Include in this file the 7 steps for Iteration 2

In this iteration, we move from the coarse-grained, conceptual architecture to a more detailed structural design that identifies the main execution flows and defines the units of implementation. The goal is to refine the system elements established earlier and allocate responsibilities that directly realize the primary use cases.

## Step 2: Establish Iteration Goal by Selecting Drivers

The goal of this iteration is to support the core functional behavior of the AIDAP system:

- **UC-1 Ask Question:** Conversational query processing and AI-based response generation.  
- **UC-2 Personalized Dashboard:** Aggregation of academic data across LMS, registration, and calendar systems.  
- **UC-7 Data Sync & Recovery:** Reliable data synchronization with failure handling and automatic recovery.

This iteration continues to emphasize the same high-priority quality attributes:

- **QA-1 Performance:** Response ≤ 2 s  
- **QA-2 Availability:** ≥ 99.5% uptime, failover for integration errors  
- **QA-3 Security & Privacy:** SSO, access control, encryption  
- **QA-4 Scalability:** Autoscaling microservices and async processing  


---

## Step 3: Choose One or More Elements of the System to Refine

| Subsystem | Rationale for Refinement |
|----------|---------------------------|
| **Conversation Service & AI Orchestrator** | Core to UC-1; handles latency, AI model integration, and personalization. |
| **Dashboard Service & Integration Facade** | Enables UC-2; validates cross-system data composition and real-time updates. |
| **Sync & Recovery Worker** | Supports UC-7; focuses on resilience and availability attributes. |

---

## Step 4: Choose Design Concepts That Satisfy the Selected Drivers

| Design Decision & Location | Rationale / QA Addressed |
|----------------------------|---------------------------|
| **Event-driven communication between core services and integration layer** | Async queues (Kafka) reduce latency and decouple services (QA-1, QA-2). |
| **Caching layer (edge & model responses)** | Improves response time for repeated queries (QA-1). |
| **Bulk + incremental sync for adapters** | Reduces load, supports fail-safe recovery (QA-2, UC-7). |
| **Policy-based Access Control** | Fine-grained authorization (QA-3). |
| **WebSocket / GraphQL Subscriptions** | Real-time dashboard updates (QA-1, UC-2). |
| **Circuit-breaker & retry pattern in adapters** | Resilience during external failures (QA-2, UC-7). |
| **Auto-scaler policies per service** | Maintains performance at 5,000+ concurrent users (QA-4). |

**Discarded alternatives:**  
Direct synchronous HTTP calls, manual batch syncs, monolithic auth — rejected due to poor scalability and maintainability.

---

## Step 5: Instantiate Architectural Elements and Define Interfaces

---

### **UC-1 – Ask Question Flow**


1. User sends query → BFF → API Gateway  
2. Gateway authenticates via SSO and forwards to Conversation Service  
3. Conversation Service checks cache → calls AI Orchestrator if miss  
4. AI Orchestrator composes prompt using Profile Service + Vector Store  
5. Model endpoint invoked → response returned  
6. Conversation Service stores interaction + returns answer  

---

### **UC-2 – Personalized Dashboard Flow**


1. Dashboard request sent to API Gateway  
2. Dashboard Service calls Integration Facade  
3. Facade runs parallel async LMS + Registration + Calendar calls  
4. Results merged + cached  
5. Returned via GraphQL w/ live subscriptions  


### **UC-7 – Data Sync & Recovery Flow**


1. Scheduled sync triggers worker  
2. Worker fetches delta changes via cursor-based pagination  
3. Applies idempotent upserts to DB + updates Vector Store  
4. Failures routed to retry queue  
5. Circuit breaker activates on repeated failure  
6. Analytics Service logs outcomes  


## Step 6: Views for Primary Use Cases & Responsibilities

Refined Module view: 


