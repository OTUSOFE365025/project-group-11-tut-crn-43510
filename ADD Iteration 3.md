Include in this file the 7 steps for Iteration 3


Iteration 3

## Iteration Goal

To refine the architectural elements to better support key quality attributes that have not yet been satisfied in previous iterations.

  

Quality attributes to refine:

1. Security & Privacy (QA-3) - SSO, data isolation, encryption, audit
    
2. Availability (QA-2) - Failure containment, fault tolerance, redundancy
    
3. Performance (QA-1) - Latency hotspots, AI call optimization
    

  

Drivers:

- CON-1: SSO integration
    
- CON-2: Data privacy/retention
    
- CON-6: Persistent interaction history
    
- CRN-4: Privacy & security
    
- CRN-8: Monitoring/analytics
    
- QA-3: Biggest gap from iteration 1-2
    
- QA-2: Requires deeper fault-handling beyond retires/circuit breakers
    
- QA-1: Needs end-to-end performance strategy, not only caching
    

## Elements to Refine

  

|   |   |
|---|---|
|Subsystem|Rationale|
|Security & Access Control Module|Enforce institutional policies, RBAC/ABAC, encryption, audit, token handling|
|Operational Data Stores|Need encryption-at-rest, partitioning, privacy & retention logic|
|AI Orchestrator|Needs guardrails, rate limiting per user, performance improvements|
|Integration layer Adapters|Need failure domains, fallback behaviors, health checks|
|Monitoring & Analytics Module|Needs SLO/SLA metrics, alerts, distributed tracing|

  

## Design Concepts

1. Zero-Trust Security Architecture
    

- Enforce mutual TLS for all internal service-to-service calls
    
- Every request carries identity via OIDC tokens
    
- Introduce short-lived service tokens with automatic rotation
    
- Supports: QA-3 Security & Privacy
    

2. Context-Aware Rate Limiting & Quotas
    

- Per-user rate limits at API Gateway
    
- Per-integration system quotas inside adapters
    
- AI interface throttling at Orchestrator
    
- Supports: QA-1 Performance, QA-2 Availability, QA-3 Security & Privacy
    

3. Data Partitioning + Attribute-Based Access Control (ABAC)
    

- Partition user data by institutional identity (student/staff)
    
- Sensitive logs stored in separate secured audit store
    
- ABAC policies defined centrally & evaluated inline
    
- Supports: QA-3 Security, CON-2
    

4. Redundancy & Failure Domain Isolation
    

- Per-adapter pool with independent health probes
    
- Orchestrator fallback models (less accuracy but increased availability
    
- Multi-AZ database replicas
    
- Supports: QA-2 Availability
    

5. “Distributed Tracing + Performance Budgets
    

- Tracing via OpenTelemetry
    
- 2-second budget decomposed:
    

- 150ms gateway
    
- 300ms orchestration
    
- 900ms model inference
    
- 300ms integration calls
    
- 150ms rendering
    

- Supports: QA-1 Performance, CRN-8 Monitoring
    

## Instantiating Elements

|   |   |
|---|---|
|Design Decision & Location|Rationale|
|Introduce Security & Access Module (Orchestrator + Gateway layer)|Selected because the Zero-Trust Security Architecture requires centralized enforcement of authentication, authorization, encryption, and audit logging. Placing this module at the Gateway and Orchestrator ensures all incoming and internal service calls follow consistent ABAC, token validation, and audit policies. Supports QA-3 (Security & Privacy).|
|Add Service Token Issuance & Rotation Component (Security subsystem)|Mutual TLS and short-lived service tokens demand automated key rotation. Isolating token issuance allows fine-grained scopes for inter-service calls and reduces blast radius. Chosen to satisfy Zero-Trust and least privilege principles. Supports QA-3.|
|Implement AI Orchestrator Enhancements (Core orchestration engine)|Context-aware rate-limiting, safety evaluation, caching, and fallback model selection are needed to support performance and availability requirements. This element directly operationalizes the Performance Budget and Rate Limiting design concepts. Supports QA-1 (Performance) & QA-2 (Availability).|
|Create Monitoring & Observability Module (Cross-cutting)|Distributed tracing (OpenTelemetry), SLO violation alerts, latency tracking, and failure recording are needed to satisfy performance monitoring and operational resilience. This supports the “Distributed Tracing + Performance Budgets” design concept. Supports QA-1 and CRN-8.|
|Introduce Integration Adapter Enhancements (Integration layer)|Independent health checks, per-system quotas, and fallback data logic reflect the Context-Aware Rate Limiting & Redundancy concepts. Each adapter becomes a failure-isolated domain, improving overall system robustness. Supports QA-2 (Availability) & QA-1 (Performance).|
|Adopt Data Partitioning + ABAC Engine (Data layer + Gateway)|Partitioning per institutional identity (student/staff) and enforcing ABAC policies at request time operationalizes the data-security design concept. Centralizing policy evaluation simplifies compliance and ensures sensitive data is isolated. Supports QA-3 (Security).|
|Deploy Multi-AZ Database Replicas (Infrastructure layer)|Supports the Redundancy & Failure Domain Isolation concept by enabling resilient storage that survives zone-level outages. Ensures high availability and aligns with failover requirements for critical workflows. Supports QA-2 (Availability).|
|Add Dedicated Audit Log Store (Security & Logging subsystem)|Sensitive logs require separate, secured storage per the Data Partitioning design concept. This decision isolates audit trails from general logs and ensures access control and compliance. Supports QA-3 (Security & Privacy).|

**Architecture Views**
<img width="782" height="420" alt="image" src="https://github.com/user-attachments/assets/2332ab81-5b46-49d4-bbf6-fdfb3e1c2be2" />


<img width="787" height="154" alt="image" src="https://github.com/user-attachments/assets/bee10324-abcf-437f-bbab-60573b102308" />

<img width="796" height="415" alt="image" src="https://github.com/user-attachments/assets/89652fcd-e992-40a2-8d26-fe6271f8c33e" />


## Analysis Review

  

|   |   |   |
|---|---|---|
|Requirement|Status|Notes|
|QA-3 Security|Completely Addressed|Zero-trust, ABAC, audit, encryption, SSO|
|QA-2 Availability|Completely Addressed|Multi-AZ, fallback models, adapter isolation, quotas|
|QA-1 Performance|Partially Addressed|E2E budgets set; profiling still pending|
|CON-2 Privacy/Retention|Partially Addressed|Retention jobs need to be added next iteration|
|CRN-8 Monitoring|Completely Addressed|Tracing, SLO/SLA metrics, dashboards|

  

## ATAM Risk Assessment Table

Scenario: Maintain ≤2-second response time during peak user load.

|   |   |   |   |   |
|---|---|---|---|---|
|Attribute(s)|Performance|   |   |   |
|Source|Student User|   |   |   |
|Stimulus|User sends a message during a time with a large amount of request|   |   |   |
|Artifact|API Gateway, BFF Layer, Conversation Service, AI Orchestrator|   |   |   |
|Environment|System under peak load (about 5,000 concurrent users)|   |   |   |
|Response|System processes the query end-to-end and returns an answer|   |   |   |
|Response Measure|Average response time ≤2-seconds (no SLA violation)|   |   |   |
|Architectural Decisions|Sensitivity|Tradeoff|Risk|Nonrisk|
|Edge + AI Response Caching|S1|T1|R1|N1|
|Auto-scaling Microservices|S2|T2|R2|N2|
|API Gateway Rate Limiting & Validation|S3|T3|R3|N3|
|AI Orchestrator with Fallback Model|S4|T4|R4|N4|
|Async Parallel Integration Calls|S5|T5|R5|N5|

  

Sensitivity Points

- S1 – Cache Hit Rate Sensitivity: Overall latency heavily depends on cache-hit percentage; misses cause high model-load delays.
    
- S2 – Auto-Scaling Threshold Sensitivity: Response time degrades if scale-out does not trigger quickly enough during spikes.
    
- S3 – Gateway Processing Sensitivity: Token validation and rate limiting add latency on the critical path; more complex policies increase delay.
    
- S4 – Model Inference Sensitivity: AI model inference time directly affects meeting the ≤2s SLA; overloaded endpoints cause slowdowns.
    
- S5 – Async Queue Delay Sensitivity: Queue saturation or slow consumers can add unpredictable delays under high load.  
      
    

Tradeoffs

- T1 – Freshness vs Speed: Caching speeds responses but may serve slightly stale information.
    
- T2 – Resource Usage vs Responsiveness: Lower scaling thresholds improve performance but increase operational cost.
    
- T3 – Throughput vs Protection: Aggressive rate limiting protects performance but reduces the number of accepted requests.
    
- T4 – Accuracy vs Latency: Using a faster fallback model improves latency but may slightly reduce answer quality.
    
- T5 – Consistency vs Parallelism: Async parallel calls minimize latency but may return partial/inconsistent integration data.  
      
    

Risks

- R1: Low cache-hit rate → high model call volume → response times spike above 2 seconds.
    
- R2: Auto-scaling reacts too slowly → temporary overload → SLA violation.
    
- R3: Gateway token validation and policy checks create noticeable queuing delays during peak times.
    
- R4: Model inference exceeds budget → entire request pipeline slows down.
    
- R5: Async queues fill up → delays in integration calls → slower dashboard and response workflows.  
      
    

Non-Risks

- N1: High cache-hit rate reliably produces sub-second responses.
    
- N2: Predictable traffic patterns allow scaling policies to maintain performance.
    
- N3: Gateway routing remains lightweight and stable under normal conditions.
    
- N4: Modular AI Orchestrator allows fast model replacement for performance tuning.
    
- N5: Async adapters prevent slow external systems from blocking critical request paths.  
      
    

  
## ATAM Utility Tree
<img width="770" height="766" alt="image" src="https://github.com/user-attachments/assets/c1e37f60-f985-4bdf-8db7-1bbdeb38bb09" />








