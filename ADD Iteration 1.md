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

