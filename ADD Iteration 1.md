Include in this file the 7 steps for Iteration 1

## ADD Step 1: Review Inputs

| Category | Details |
| ----- | ----- |
| Design purpose | This is a greenfield system in a mature domain (university IT). The goal is to produce a sufficiently detailed architecture to support construction, early performance validation (≤2s), and safe integrations with institutional systems. |
| Primary functional requirements | From the use cases, the primary ones are: UC-1 (*Ask Question*): Directly supports the core value (AI conversational access). UC-2 (*Personalized Dashboard*): Core student value; validates aggregation & personalization. UC-7 (*Data Sync & Recovery*): Chosen for technical risk (resilience, recovery, and integration fault handling). |
| Quality attribute scenarios (prioritized) | QA-1 Performance (≤2s avg response) — High / High QA-2 Availability (≥99.5% uptime, graceful degradation) — High / Medium QA-3 Security & Privacy (SSO, authZ, encryption, isolation) — High / High QA-4 Scalability (5,000 concurrent users; autoscaling) — High / High QA-5 Interoperability (plug-in new LMS/calendar via REST/GraphQL) — Medium / Medium QA-6 Maintainability/Deployability (zero-downtime updates) — Medium / High → Drivers selected this iteration: QA-1, QA-2, QA-3, QA-4. |
| Constraints (drivers) | CON-1 Integrate with SSO (OIDC/SAML). CON-2 Comply with data privacy/retention policies. CON-3 Use standard REST or GraphQL APIs. CON-4 ≤2s response & ≥99.5% uptime. CON-5 Cloud-native; autoscaling, failover, zero-downtime updates (K8s). CON-6 Persist user interactions (text & voice) for personalization/auditing. |
| Architectural concerns (drivers) | CRN-1 Scalable structure for 5,000 concurrent users. CRN-2 Modular integration layer for LMS/registration/calendar/email. CRN-3 AI component for NLU \+ low latency. CRN-4 Privacy & security (SSO, RBAC/ABAC, encryption). CRN-5 Continuous deployment & monitoring (zero-downtime). CRN-6 Consistent multi-platform access (web, mobile, voice). CRN-7 Personalized history storage with isolation. CRN-8 Analytics/monitoring for usage, performance, model accuracy. CRN-9 Pluggable AI models & new system integrations. |

Iteration 1: Establishing an Overall System Structure

### Step 2: Establish Iteration Goal by Selecting Drivers

Iteration goal: Achieve CRN-1 by defining an overall system structure that honors the key drivers: QA-1 (Performance), QA-2 (Availability), QA-3 (Security/Privacy), QA-4 (Scalability), plus CON-1..6 and CRN-2/3/6.

### Step 3: Choose One or More Elements of the System to Refine

This is a greenfield system; the element to refine the entire AIDAP system (top-level context), decomposed into client channels, an API gateway/BFF layer, core services, AI services, and integration services.

### Step 4: Choose One or More Design Concepts That Satisfy the Selected Drivers

In this initial iteration, given the goal of structuring the entire system, the first iteration focuses on high-level reference architectures, deployment patterns, and major structural decisions that determine the overall shape of the system.

The following table summarizes the selection of design decisions. These decisions were chosen to satisfy the primary drivers: QA-1 (Performance), QA-2 (Availability), QA-3 (Security & Privacy), QA-4 (Scalability), along with constraints CON-1..6.

| Design Decisions and Location | Rationale |
| ----- | ----- |
| Logically structure the client part of the system using a Rich Web / Multi-Channel Client reference architecture | A Rich Web / Multi-Channel architecture supports browser, mobile, and voice interfaces needed for UC-1 (Ask Question) and UC-2 (Personalized Dashboard). It provides interactive UI capabilities without requiring installation on user devices. This supports cross-platform access (CRN-6) and contributes to meeting the ≤ 2-second response requirement (QA-1). |
| Logically structure the server part of the system using the Service Application (Microservices) reference architecture | The Service Application architecture decomposes the system into independently scalable services (conversation, dashboard, integrations, notifications). This separation supports scalability (QA-4), fault isolation (QA-2), and secure data handling (QA-3). It also enables future expansion and integration of new systems or AI models (CRN-2, CRN-9). |
| Introduce an API Gateway and Backend-for-Frontend (BFF) layer | The API Gateway centralizes SSO authentication (CON-1), request validation, and rate limiting. BFF components tailor responses for web, mobile, and voice clients, improving usability and reducing latency (QA-1). This boundary also improves security (QA-3) and ensures consistent multichannel access (CRN-6). |
| Physically structure the system using a cloud-native three-tier deployment pattern | A cloud-native deployment on an orchestration platform (e.g., Kubernetes) supports autoscaling, failover, and zero-downtime updates (CON-5). Separating client, service, and data tiers supports availability (QA-2) and ensures compliance with data retention and persistence policies (CON-2, CON-6). |
| Introduce a dedicated AI Orchestrator subsystem with model endpoints, vector store, and response cache | A dedicated AI orchestration layer supports contextual prompts, retrieval-augmented generation, and model selection. This helps achieve performance goals (QA-1), security and isolation of user context (QA-3), and future extensibility for new model versions (CRN-9). Vector storage also enables personalization (CRN-7). |
| Introduce an Integration Layer with Adapter (Connector) microservices behind an Integration Facade | Adapter microservices isolate vendor-specific protocols from LMS, registration, calendar, and email systems. This supports interoperability (QA-5) and resilience through retry, caching, and circuit-breaker patterns. The Integration Facade exposes a stable domain API, reducing coupling and supporting UC-7 (Data Sync & Recovery). |
| Introduce cross-cutting mechanisms: SSO, mTLS, encryption, centralized logging, monitoring, RBAC/ABAC | Cross-cutting services enforce institutional security and privacy requirements (QA-3, CON-1, CON-2). Centralized telemetry supports CRN-8 by enabling performance monitoring and SLA verification. Encryption and RBAC/ABAC ensure protection of student and staff data across all components. |

### Step 5: Instantiate Architectural Elements, Allocate Responsibilities, and Define Interfaces

| Design Decision and Location | Rationale |
| ----- | ----- |
| Do not store application data on client devices in any channel (web, mobile, or voice) | Because the system is cloud-hosted and always reachable, there is no need for local data persistence. All stateful information—such as conversation history, dashboard data, or personalized preferences—is retrieved from and stored in backend services. This simplifies clients, reduces security risk, and ensures consistency across devices (QA-3, CON-2). Network access is reliable in the institutional environment, and any offline mode is out of scope for this iteration. |
| Create a module dedicated to orchestrating AI interactions in the service layer of the Service Application reference architecture | A dedicated AI Orchestrator abstracts the interaction with model endpoints, vector stores, and caching mechanisms. This reduces coupling between client-facing services and AI infrastructure and supports future model replacement or extension (CRN-9). It also plays a critical role in meeting the performance requirement (QA-1) and personalized response behavior required for UC-1. |
| Create a module dedicated to managing multi-platform access in the BFF layer | The Backend-for-Frontend modules enable channel-specific transformation of data for web, mobile, and voice clients. This division isolates client concerns from core services and contributes to meeting the cross-platform access requirement (CRN-6). Because each channel shares common business logic via the gateway, BFF modules need only lightweight processing. |
| Instantiate Integration Adapter modules in the data layer of the Service Application architecture | Adapter modules encapsulate communication with external systems such as the LMS, registration, and calendar services. This abstraction supports UC-2 and UC-7 by isolating third-party protocol differences and providing a consistent domain interface. These modules will also later support mechanisms such as retry, backoff, and circuit breakers for availability (QA-2). |
| Introduce a centralized Security and Access Control module as a cross-cutting concern | Authentication (SSO), authorization (RBAC/ABAC), encryption, and audit logging must be uniformly enforced across all services (CON-1, QA-3). A centralized module simplifies the application of these policies. Although the specifics of its implementation are deferred to later iterations, its placement as a cross-cutting structure is established here. |

Step 6: Sketch Views and Record Design Decisions

Module View

<img width="841" height="411" alt="image" src="https://github.com/user-attachments/assets/4c181d96-0c12-478a-b055-09ffd166f296" />


Sketch was created using PlantUML. The tool captures each component and the organizational structure/hierarchy capturing the module view. Below is each element and it’s responsibilities:

| Element | Responsibility |
| ----- | ----- |
| Web/Mobile/Voice Clients | Render the user interface and manage user interaction for conversational queries and dashboard views. |
| BFF Modules (Web, Mobile, Voice) | Transform and tailor responses for each client channel; handle lightweight session and request shaping. |
| API Gateway | Enforce SSO authentication, API security policies, rate limiting, and route incoming requests to backend services. |
| Conversation Service | Orchestrates conversational flows and interacts with the AI Orchestrator to provide responses (supports UC-1). |
| Dashboard Service | Aggregates and composes academic, scheduling, and announcement data for personalized dashboards (supports UC-2). |
| User/Profile Service | Stores and retrieves user preferences, personalization history, and consent records. |
| Notifications Service | Sends push notifications, reminders, and alerts via email or app channels. |
| Admin/Config Service | Manages system configurations, integration settings, and policy enforcement. |
| AI Orchestrator | Coordinates prompt construction, model selection, retrieval, and safety filtering. |
| Model Endpoint | Executes ML model inference requests. |
| Vector Store Access | Retrieves user embeddings and institutional content for semantic search. |
| Response Cache | Stores recent conversational responses for latency reduction. |
| Integration Facade | Provides a unified domain API for LMS, registration, calendar, and email services. |
| Adapter Modules (LMS, Registration, Calendar, Email) | Isolate and abstract external APIs; maintain protocol mappings and error-handling mechanisms. |
| Security & Access Control Module | Enforces authentication, authorization, encryption, and token handling. |
| Logging & Monitoring Module | Provides centralized tracing, metrics, logs, and observability dashboards. |
| Audit & Compliance Module | Records critical system actions for compliance and institutional governance. |

Summarrizing the deployment view below:

<img width="756" height="711" alt="image" src="https://github.com/user-attachments/assets/8a329b5d-d9d2-48e6-a274-2dc94e68a8a5" />


And the relationships:

| Relationship | Description |
| ----- | ----- |
| Between user device and application server | Communication uses HTTPS for secure transport of conversational queries and dashboard requests. |
| Between application server and database server | Communication uses SQL over secure connections for operational data, and REST for vector retrieval and object storage operations. |
| Between application server and external systems | Integration adapters communicate using REST or GraphQL APIs exposed by LMS, registration, calendar, and email systems. |

Step 7: Perform Analysis of Current Design and Review Iteration Goal

| Requirement / Concern | Status | Design Decisions Made |
| ----- | ----- | ----- |
| UC-1 Ask Question | Partially Addressed | Defined AI Orchestrator and conversational interface. |
| UC-2 Personalized Dashboard | Partially Addressed | Defined dashboard and integration layer components. |
| UC-7 Data Sync & Recovery | Partially Addressed | Introduced synchronization and retry mechanisms in adapters. |
| QA-1 Performance | Partially Addressed | Adopted GraphQL and caching; latency validation in next iteration. |
| QA-2 Availability | Partially Addressed | Introduced autoscaling and redundancy in deployment plan. |
| QA-3 Security & Privacy | Partially Addressed | Integrated SSO and encryption; compliance rules to be finalized. |
| QA-4 Scalability | Partially Addressed | Defined microservices and containerized deployment. |
| CRN-1 Overall System Structure | Completely Addressed | Core architecture pattern established. |
| CRN-2 Integration Layer | Completely Addressed | Modular adapter layer adopted. |
| CRN-3 AI Component | Completely Addressed | AI Orchestrator and model endpoints defined. |
