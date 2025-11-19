Include in this file the 7 steps for Iteration 1

Step 1: Review Inputs (architectural drivers)
| Category                          | Details |
|----------------------------------|---------|
| **Design purpose**               | This is a greenfield system in a mature domain (university IT). The goal is to produce a sufficiently detailed architecture to support construction, early performance validation (≤2s), and safe integrations with institutional systems. |
| **Primary functional requirements** | From the use cases, the primary ones are:<br>• **UC-1 (Ask Question):** Directly supports the core value (AI conversational access).<br>• **UC-2 (Personalized Dashboard):** Core student value; validates aggregation & personalization.<br>• **UC-7 (Data Sync & Recovery):** Chosen for technical risk (resilience, recovery, and integration fault handling). |
| **Quality attribute scenarios (prioritized)** | • **QA-1 Performance** (≤2s avg response) — High / High<br>• **QA-2 Availability** (≥99.5% uptime, graceful degradation) — High / Medium<br>• **QA-3 Security & Privacy** (SSO, authZ, encryption, isolation) — High / High<br>• **QA-4 Scalability** (5,000 concurrent users; autoscaling) — High / High<br>• **QA-5 Interoperability** (plug-in new LMS/calendar via REST/GraphQL) — Medium / Medium<br>• **QA-6 Maintainability/Deployability** (zero-downtime updates) — Medium / High<br><br>**→ Drivers selected this iteration: QA-1, QA-2, QA-3, QA-4.** |
| **Constraints (drivers)**        | • **CON-1** Integrate with SSO (OIDC/SAML).<br>• **CON-2** Comply with data privacy/retention policies.<br>• **CON-3** Use standard REST or GraphQL APIs.<br>• **CON-4** ≤2s response & ≥99.5% uptime.<br>• **CON-5** Cloud-native; autoscaling, failover, zero-downtime updates (K8s).<br>• **CON-6** Persist user interactions (text & voice) for personalization/auditing. |
| **Architectural concerns (drivers)** | • **CRN-1** Scalable structure for 5,000 concurrent users.<br>• **CRN-2** Modular integration layer for LMS/registration/calendar/email.<br>• **CRN-3** AI component for NLU + low latency.<br>• **CRN-4** Privacy & security (SSO, RBAC/ABAC, encryption).<br>• **CRN-5** Continuous deployment & monitoring (zero-downtime).<br>• **CRN-6** Consistent multi-platform access (web, mobile, voice).<br>• **CRN-7** Personalized history storage with isolation.<br>• **CRN-8** Analytics/monitoring for usage, performance, model accuracy.<br>• **CRN-9** Pluggable AI models & new system integrations. |

Step 2: Establish Iteration Goal by Selecting Drivers <br>
<br>
Iteration goal: Achieve CRN-1 by defining an overall system structure that honors the key drivers: QA-1 (Performance), QA-2 (Availability), QA-3 (Security/Privacy), QA-4 (Scalability), plus CON-1..6 and CRN-2/3/6.
<br>
<br>
## Step 3: Choose One or More Elements of the System to Refine

This is a greenfield system; therefore, the element refined in this iteration is the **entire AIDAP system at the top-level context**.

The refinement decomposes the system into:

- Client channels (web, mobile, voice)
- API Gateway / Backend-for-Frontend (BFF) layer
- Core services
- AI services
- Integration services


## Step 4: Choose One or More Design Concepts That Satisfy the Selected Drivers

This iteration focuses on **high-level structural decisions**, including reference architectures, deployment patterns, and major system organization choices.

The selected concepts satisfy the primary drivers:

- **QA-1** Performance (≤2s response)
- **QA-2** Availability
- **QA-3** Security & Privacy
- **QA-4** Scalability  
Along with constraints **CON-1 … CON-6**.


## Design Decisions Table

| **Design Decision** | **Rationale** |
|---------------------|---------------|
| **Use a Rich Web / Multi-Channel Client reference architecture** | Supports browser, mobile, and voice channels for UC-1 (Ask Question) and UC-2 (Personalized Dashboard). Enables interactive UI without installations, supports cross-platform access (CRN-6), and contributes to achieving the ≤2s response requirement (QA-1). |
| **Use a Service Application (Microservices) architecture for server-side logic** | Decomposes system into independent services (conversation, dashboard, integrations, notifications). Supports scalability (QA-4), fault isolation (QA-2), secure data handling (QA-3), and future extensibility for new systems or AI models (CRN-2, CRN-9). |
| **Introduce an API Gateway and Backend-for-Frontend (BFF)** | API Gateway centralizes SSO (CON-1), rate limiting, and validation. BFFs tailor responses for web, mobile, and voice clients, reducing latency and improving usability (QA-1). Improves security (QA-3) and ensures consistent multi-channel integration (CRN-6). |
| **Adopt a cloud-native three-tier deployment model** | Cloud orchestration (e.g., Kubernetes) ensures autoscaling, failover, and zero-downtime updates (CON-5). Separating client, service, and data tiers supports availability (QA-2) and satisfies data compliance constraints (CON-2, CON-6). |
| **Introduce a dedicated AI Orchestrator subsystem** | Provides model endpoints, vector store, and cache to support RAG, contextual prompts, and model switching. Enhances performance (QA-1), security of contextual data (QA-3), and future model expansions (CRN-9). Enables personalization features (CRN-7). |
| **Introduce an Integration Layer with Adapter microservices behind a Facade** | Adapter microservices isolate vendor-specific protocols for LMS, registration, calendar, and email systems. Increases interoperability (QA-5) and resilience through retries, caching, and circuit-breaker patterns. Facade exposes stable APIs and supports UC-7 (Data Sync & Recovery). |
| **Add cross-cutting mechanisms: SSO, mTLS, encryption, logging, monitoring, RBAC/ABAC** | Enforces institutional security and privacy (QA-3, CON-1, CON-2). Centralized telemetry supports SLA monitoring (CRN-8). Strong authZ/authN protects sensitive student and staff data across all services. |

## Step 5: Instantiate Architectural Elements, Allocate Responsibilities, and Define Interfaces

| **Design Decision** | **Rationale** |
|---------------------|---------------|
| **Do not store application data on client devices in any channel (web, mobile, or voice)** | Since the system is cloud-hosted and always reachable, no local storage is required. All stateful data—conversation history, dashboards, preferences—is stored in backend services. This simplifies clients, reduces security risks, and ensures consistent behavior across devices (QA-3, CON-2). Offline mode is out of scope for this iteration. |
| **Create a dedicated AI Orchestrator module in the service layer** | The AI Orchestrator abstracts model endpoints, vector store interactions, and caching. It reduces coupling, enables future model replacement/expansion (CRN-9), supports fast response times (QA-1), and provides personalized behavior required for UC-1. |
| **Create multi-platform access modules in the BFF layer** | BFF modules handle channel-specific formatting for web, mobile, and voice. This isolates client concerns and ensures all channels share business logic through the gateway. Supports multi-channel access requirements (CRN-6) with minimal overhead. |
| **Instantiate Integration Adapter modules in the data layer** | Adapter modules encapsulate communication with external systems (LMS, registration, calendar). This abstraction supports UC-2 and UC-7, isolates third-party protocols, and prepares for retries/backoff/circuit breakers for improved availability (QA-2). |
| **Introduce a centralized Security & Access Control module as a cross-cutting concern** | Uniform enforcement of SSO, RBAC/ABAC, encryption, and audit logging is required for compliance (CON-1, QA-3). A centralized module simplifies policy application across all services. Final implementation details will be refined in later iterations. |




