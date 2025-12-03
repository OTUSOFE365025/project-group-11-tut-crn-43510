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
<img width="660" height="631" alt="Screenshot 2025-12-02 at 7 53 31 PM" src="https://github.com/user-attachments/assets/c445508e-66b7-492f-9074-67e3c0961caf" />

 DomainObjects to associated use cases:

<img width="328" height="438" alt="Screenshot 2025-12-02 at 7 58 06 PM" src="https://github.com/user-attachments/assets/01ad6281-cc46-4ebd-814a-e91d96ae9701" />


Module view to support Use Cases:
<img width="840" height="700" alt="image" src="https://github.com/user-attachments/assets/d08d6f8d-8ffe-41d8-819d-2f8865b66d78" />



---

### **Module Responsibilities Table**

| Element | Responsibility |
|--------|----------------|
| **QueryInputView** | Displays input UI, updates when responses arrive (UC-1). |
| **DashboardView** | Shows personalized dashboard (UC-2). |
| **QueryController** | Coordinates query handling on client side. |
| **DashboardController** | Retrieves and renders dashboard info. |
| **ClientRequestManager** | Manages outbound communication + auth tokens. |
| **RequestService (Facade)** | Unified client entry point (UC-1, UC-2, UC-7). |
| **ConversationController** | Server-side logic for UC-1. |
| **DashboardComposer** | Aggregates multi-system data (UC-2). |
| **SyncCoordinator** | Sync + recovery business logic (UC-7). |
| **AIOrchestrator** | Prompt building, vector retrieval, model invocation. |
| **DomainEntities** | Query, AIResponse, Course, Event. |
| **UserDataMapper** | Profile CRUD + personalization data. |
| **CourseDataMapper** | Course persistence. |
| **EventDataMapper** | Event persistence. |
| **ExternalSystemConnector** | LMS / Registration / Calendar integrations. |
| **VectorStoreConnector** | Embeddings + semantic search operations. |

---

## **UC-1 – Ask Question: Methods**

| Element | Method | Description |
|--------|--------|-------------|
| **ConversationView** | `initialize()` | Prepares UI. |
| | `submitQuery(text)` | Sends query to BFF. |
| **ConversationController** | `getResponse(query)` | Forwards request server-side. |
| **RequestManager** | `requestAnswer(query)` | Sends request to Conversation Service. |
| **APIGateway** | `routeRequest(query)` | Auth + routing. |
| **ConversationService** | `handleQuery()` | Orchestrates AI flow; cache → orchestrator. |
| | `storeInteraction()` | Saves history. |
| **AIOrchestrator** | `assemblePrompt()` | Builds contextualized prompt. |
| | `invokeModel()` | Calls LLM endpoint. |
| **VectorStore** | `retrieveContext()` | Retrieves embeddings. |
| **ProfileService** | `getUserProfile()` | Returns personalization data. |
| **ModelEndpoint** | `generate()` | Produces final model output. |

---

## **UC-2 – Personalized Dashboard: Methods**

| Element | Method | Description |
|--------|--------|-------------|
| **DashboardView** | `openDashboard()` | Loads UI. |
| | `renderDashboard(data)` | Displays combined course/event info. |
| **DashboardBFF** | `requestDashboard()` | Sends request. |
| | `returnDashboard()` | Formats response. |
| **APIGateway** | `forwardRequest()` | Auth + routing. |
| **DashboardService** | `getDashboardData()` | Orchestrates aggregation. |
| | `mergeData()` | Produces final dashboard object. |
| **IntegrationFacade** | `fetchAggregatedData()` | Parallel adapter calls. |
| **LMSAdapter** | `fetchCourses()` | Gets courses. |
| **RegistrationAdapter** | `fetchEnrollments()` | Enrollment data. |
| **CalendarAdapter** | `fetchEvents()` | Event data. |

---

## **UC-7 – Data Sync & Recovery: Methods**

| Element | Method | Description |
|--------|--------|-------------|
| **SyncScheduler** | `triggerSync()` | Initiates sync job. |
| **SyncWorker** | `runIncrementalSync()` | Delta sync. |
| | `runFullSync()` | Full dataset sync. |
| | `processItems(batch)` | Writes with idempotency. |
| | `enqueueRetry(item)` | Pushes failures. |
| **IntegrationFacade** | `fetchDelta(cursor)` | Merged delta results. |
| **Adapters** | `syncCourses()` / `syncEnrollment()` / `syncEvents()` | External updates. |
| **OperationalDB** | `upsert()` | Idempotent writes. |
| **VectorStore** | `updateEmbedding()` | Embedding updates. |
| **CircuitBreaker** | `check() / open() / close()` | Failure isolation. |
| **AnalyticsService** | `recordSyncOutcome()` | Logs metrics. |

---

## Step 7: Perform Analysis of Current Design and Review Iteration Goal

| Driver / Requirement | Status | Design Decisions Made During Iteration |
|----------------------|--------|----------------------------------------|
| **UC-1 Ask Question** | ✔ Completely Addressed | AI Orchestrator, caching, flow fully designed. |
| **UC-2 Personalized Dashboard** | ◑ Partially Addressed | Async aggregation works; needs more validation. |
| **UC-7 Data Sync & Recovery** | ✔ Completely Addressed | Retry queues, sync worker, circuit breaker defined. |
| **QA-1 Performance** | ◑ Partially Addressed | Caching + parallelism added; profiling next. |
| **QA-2 Availability** | ◑ Partially Addressed | Failover + async retries; more refinement needed. |
| **QA-3 Security & Privacy** | ◑ Partially Addressed | PBAC added; retention next. |
| **QA-4 Scalability** | ✔ Completely Addressed | Autoscaling + partitioned queues defined. |

