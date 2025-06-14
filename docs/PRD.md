# Product Development Requirements (PDR) - Freemium

## 1. Overview

### Product Vision and Purpose
Freemium is a modular, API-first inventory platform for ESG-critical commodities and digital environmental assets. It aims to provide corporates across agri-commodities, precious metals, battery supply chains, and emerging environmental markets with a plug-and-play way to inventory physical and digital environmental assets, prove provenance, and generate compliant reports for audit-grade ESG disclosure mandates. The platform is designed for fast initial deployment via a freemium desktop/Docker image, with the capability to scale to a SaaS model with premium analytics.

### Scope and Boundaries
**Included in initial release:**
*   Desktop or Docker image installation (no cloud data egress for initial deployment).
*   CSV and API ingest for inventory and ESG metadata via modular connectors.
*   Rules engine for asset reconciliation and emission of assurance ledger entries.
*   Heat-map and variance dashboard.
*   One-click PDF report generation for auditors.

**Not included in initial release (potential future scope):**
*   Full multi-user SaaS capabilities and premium modules (activated via optional Edge-Sync).

### Assumptions
*   Target users have existing data in CSV format or accessible via APIs.
*   Users possess a baseline understanding of ESG concepts and reporting requirements.
*   The initial focus is on providing a standalone, secure solution before expanding to cloud-based services.

### Goals and Success Metrics
*   **Primary KPI:** `<30 min to audit-ready ESG inventory & variance dashboard for first dataset upload>` – beating spreadsheet baselines by >90%.
*   Mirror the “< 1 hr time-to-first-value” case study.

## 2. Stakeholder Identification

### Internal Stakeholders
*   Product Management
*   Engineering (Development, QA)
*   Design (UI/UX)
*   CTO

### External Stakeholders
*   Sustainability & ESG Leads (Scope 3, CSRD)
*   Trading & Operations Managers (inventory accuracy)
*   Finance / Audit teams (assurance & controls)
*   Potential partners for data connectors or premium modules.
*   Regulatory bodies (indirectly, by ensuring compliance).

### Responsibilities
*   **Product Management:** Define product vision, roadmap, prioritize features, and ensure alignment with user needs.
*   **Engineering:** Develop, test, and deploy the platform according to specifications.
*   **Design:** Create intuitive and effective user interfaces and experiences.
*   **CTO:** Provide technical leadership, oversee architecture, and ensure NFRs are met.
*   **External Stakeholders (Users):** Provide feedback, participate in UAT, and utilize the platform for ESG reporting.

## 3. Product Requirements

### Functional Requirements
1.  **Installation:** User can install Freemium desktop or Docker image.
2.  **Data Ingest:**
    *   System supports CSV file ingest for inventory and ESG metadata.
    *   System supports API ingest (details to be specified per connector).
    *   Ingested data is normalized via modular connectors.
3.  **Asset Reconciliation:** A rules engine reconciles assets.
4.  **Assurance Ledger:** The system emits assurance ledger entries based on reconciliation.
5.  **Reporting & Visualization:**
    *   User can view a heat-map of assets/ESG data.
    *   User can view a variance dashboard.
    *   User can generate a one-click PDF report suitable for auditors.
6.  **Modularity:** Connectors for data ingest are modular.
7.  **(Optional Future) Edge-Sync:** System allows activation for multi-user SaaS and premium modules.

### Non-Functional Requirements
| Area            | Target               | Validation         |
|-----------------|----------------------|--------------------|
| Latency (p95)   | ≤ 200 ms             | k6 load test       |
| Offline-first   | Full desktop functionality | QA scenario matrix |
| Security        | ISO 27001 controls   | Pen-testing        |
| Data Integrity  | Accurate reconciliation and ledger entries | Automated checks, Audit trail |
| Usability       | Intuitive for target users | User testing       |

### User Stories
*   As a Sustainability Lead, I want to easily upload my commodity inventory data via CSV so that I can quickly generate an ESG report.
*   As an Operations Manager, I want to view a dashboard showing variances in my environmental assets so that I can identify discrepancies.
*   As a Finance Auditor, I want to receive a standardized PDF report of ESG inventory so that I can perform my audit efficiently.

## 4. Technical Requirements

### Platform and Technology Stack
*   **Deployment:** Docker container images and standalone desktop binary; produced via Gradle as Spring Boot fat‑jars and containerised for consistent runtime.  
*   **Backend:** Kotlin 17 with Spring Boot 3.x micro‑services, aligned with the existing service stack.  
*   **CLI Utilities:** Kotlin‑based CLI tools for ingest pipelines, migrations, and local debugging.  
*   **Frontend:** React (Next.js) scaffolded via Vercel v0.  
*   **Database:** SQLite for desktop mode, PostgreSQL for Edge‑Sync/SaaS deployments.  
*   **API:** Versioned REST / GraphQL endpoints implemented with Spring controllers; OpenAPI docs generated from source.  

### System Architecture
*   **Layers:** Ingest Connectors → Normalisation & ESG Enrichment → Assurance Ledger → API / UI & Dashboards.
*   *(Embed `/diagrams/freemium_arch_v0.png` for visual representation)*

### Data Requirements
*   **Data Sources:** CSV files, ERP systems (via API), Carbon/REC Registries (via API).
*   **Data Formats:**
    *   CSV Ingest: Defined in `specs/csv_ingest.md`.
*   **Database Schema:** (To be designed based on asset types, ESG metadata, ledger entries).
*   **Data Storage:** Local storage for desktop/Docker; (cloud storage for future SaaS).

### API Specifications
| Interface                 | Protocol       | Spec Document             |
|---------------------------|----------------|---------------------------|
| CSV Ingest                | Local file     | `specs/csv_ingest.md`     |
| ERP Sync                  | REST / GraphQL | `specs/erp_sync.md`       |
| Registry API (carbon/REC) | REST           | `specs/registry_api.md`   |
| Internal APIs             | (To be defined) | (To be documented)        |

## 5. User Interface (UI) and User Experience (UX) Design

### Wireframes/Prototypes
*   (To be developed - Link to Figma/Sketch/etc. when available)
*   Key screens: Data upload, Dashboard view, Report generation, Settings.

### Design Specifications
*   (To be developed - Style guide, component library)
*   Focus on clarity, ease of use for non-technical users, and professional presentation for reports.

### Accessibility Requirements
*   Adherence to WCAG 2.1 Level AA guidelines where applicable.

## 6. Security and Compliance

### Authentication and Authorization
*   **Desktop/Docker:** Local user authentication (if applicable, or direct access).
*   **(Future SaaS):** Robust user authentication (e.g., OAuth2, MFA), role-based access control (RBAC).

### Data Privacy and Protection
*   Initial deployment ensures no cloud data egress.
*   Data at rest and in transit to be encrypted (especially for future SaaS).
*   Compliance with relevant data privacy regulations (e.g., GDPR if applicable).

### Regulatory Compliance
*   Designed to support audit-grade ESG disclosure mandates (e.g., CSRD).
*   System security to align with ISO 27001 controls.

### 7. LLM Interaction Layer – Model Context Protocol (MCP) Server

**Position in Flow**  
Sits between the Client UI ↔ Edge/API layers.  
→ Receives structured “workspace context” from the UI, adds auth + tenant metadata, and forwards the payload to the chosen LLM vendor or a local model (desktop mode).

**Core Responsibilities**  
1. **Context Packaging** – Serialise current filters, visible ledger rows, and draft queries into a single JSON object (`ContextV1`).  
2. **Authorization Guard** – Enforce JWT/mTLS checks identical to Core API before exposing any data to the LLM.  
3. **Vector Memory Cache** – Store the last N prompts + ledger snippets per tenant to enable follow-up queries without refetching full rows.  
4. **Streaming Interface** – Provide `WS /chat` and `POST /v1/context` endpoints; supports partial token streaming to meet ≤ 200 ms p95 latency.  
5. **Telemetry Hook** – Emit timing + token-usage metrics to Kafka topic `llm.telemetry` for Grafana dashboards.

**Non-Functional Requirements**  
| Metric | Target | Notes |
|--------|--------|-------|
| **Latency (p95)** | ≤ 200 ms request→first-token | Inclusive of vector lookup and auth checks. |
| **Throughput** | 50 concurrent WebSocket sessions per tenant | Aligns with current Edge limits. |
| **Offline Fallback** | Must operate with local SQLite + on-device GGUF model in Desktop mode | Feature-parity with SaaS path. |
| **Security** | Same JWT issuer + mTLS policy as Core API | No additional IdP introduced. |
| **Observability** | Traces pushed to OpenTelemetry; errors to Sentry | Alert at p95 > 250 ms for 5 min. |

**Deployment / Versioning**  
* Packaged as a stateless Node/TypeScript micro-service (`mcp-svc`) deployed alongside Edge functions.  
* One container per tenant in SaaS; desktop mode runs an embedded binary.  
* Follows Semantic Versioning; breaking schema changes require `/v2` endpoints.

**Open Items & Owners**  
| Item | Owner | Sprint |
|------|-------|--------|
| Vector schema finalised (`prompt`, `asset_id`, `embedding`) | @data-lead | SPRINT-2 |
| Latency budget test harness in CI | @devops | SPRINT-3 |
| Desktop GGUF model selection (7B vs 13B) | @ml-lead | SPRINT-3 |

---

## 7. Testing and Validation

### Test Plan
*   **Unit Tests:** For individual components and modules.
*   **Integration Tests:** For interactions between system layers (e.g., ingest to ledger).
*   **End-to-End Tests:** Simulating user flows from data upload to report generation.
*   **Performance Tests:** k6 load testing for API endpoints and dashboard responsiveness.
*   **Security Tests:** Penetration testing.
*   **User Acceptance Tests (UAT):** With target users.
*   **Offline Functionality Tests:** QA scenario matrix for desktop version.

### QA Requirements
*   Test coverage target: >80% for critical modules.
*   Automated testing integrated into CI/CD pipeline.
*   Regular regression testing.

### Acceptance Criteria
*   All defined functional requirements are met.
*   Non-functional requirements (latency, security, offline-first) are validated and met.
*   Successful completion of UAT with positive feedback from target users.
*   Code reviewed and merged to `main`.
*   IaC validated (e.g., Terraform plan clean, if applicable).

## 8. Project Timeline and Milestones
*(To be defined - Example structure below)*

*   **Phase 1: Core Desktop/Docker Product (MVP)**
    *   Milestone 1.1: Basic CSV Ingest & Normalization - [Date]
    *   Milestone 1.2: Rules Engine & Assurance Ledger v1 - [Date]
    *   Milestone 1.3: Dashboard & PDF Reporting v1 - [Date]
    *   Milestone 1.4: MVP Release - [Date]
*   **Phase 2: API Connectors & Enhancements**
    *   Milestone 2.1: ERP Sync Connector - [Date]
    *   Milestone 2.2: Registry API Connector - [Date]
*   **Phase 3: (Optional) SaaS Enablement**
    *   Milestone 3.1: Edge-Sync Implementation - [Date]

### Dependencies
*   Availability of clear specifications for `specs/csv_ingest.md`, `specs/erp_sync.md`, `specs/registry_api.md`.
*   Access to sample data for testing.

## 9. Budget and Resources
*(To be defined)*

### Cost Estimates
*   Development effort (person-months/hours).
*   Tools & Infrastructure (if any beyond standard development tools).

### Resource Allocation
*   Engineering team: X developers, Y QA
*   Product Manager: 1
*   Designer: (Fractional or dedicated)

## 10. Risk Management
*(To be defined - Example risks below)*

### Risk Identification
*   **Technical Risk:** Complexity in building robust and flexible data connectors.
*   **Market Risk:** Slow adoption if the "freemium desktop" model doesn't gain traction.
*   **Compliance Risk:** ESG regulations evolve, requiring ongoing updates.
*   **Usability Risk:** Platform may be too complex for some target users.

### Mitigation Strategies
*   **Technical:** Phased connector development, thorough testing with diverse datasets.
*   **Market:** Clear value proposition, targeted marketing, gather early user feedback.
*   **Compliance:** Modular design for easier updates, stay informed on regulatory changes.
*   **Usability:** Iterative design process with user testing.

### Contingency Plans
*   (To be defined for high-impact risks)

## 11. Post-Launch Plans

### Maintenance and Updates
*   Regular bug fixing and patch releases.
*   Scheduled updates for new features, connector enhancements, and compliance changes.
*   Version management strategy.

### Feedback Loops
*   In-app feedback mechanism (optional).
*   Direct engagement with early adopters and key users.
*   Monitoring of support channels.

## 12. Approval and Sign-Off

### Approval Process
1.  Draft PDR reviewed by internal stakeholders (Product, Engineering Lead, CTO).
2.  Feedback incorporated.
3.  Final PDR presented for sign-off.

### Signatures
*   Product Lead/Manager: _________________________ Date: _________
*   CTO: _________________________________ Date: _________
*   (Other key stakeholders as needed)

---
*This document is a living document and will be updated as the project progresses.*