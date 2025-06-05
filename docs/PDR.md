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

1.  **Web-Based Solution:**  
    *   The platform is delivered as a secure, responsive web application.

2.  **Authentication & User Management:**  
    *   Two-factor authentication (2FA) required for all logins.  
    *   Support for 1–2 users per account.  
    *   Admin Portal for user and product management.

3.  **Product Management:**  
    *   Admins can create and manage Products within the platform.
    *   Within the Admin Portal, users can define which attributes are available for Products, specify which are required, and set which are mandatory for each Product.

4.  **Trade Blotter:**  
    *   Centralized view of all trades with advanced filtering capabilities.  
    *   Ability to export filtered trade data to CSV.

5.  **Delta Ladder:**  
    *   Interactive ladder view with sorting by attribute.  
    *   Drill-down capability from monthly to daily timeframes.

6.  **Trade Booking & Issuance:**  
    *   Users can book trades and create issuances.  
    *   Flexible booking architecture supporting daily, monthly, and strip forwards, futures contracts, and physical trades.
    *   
        **Trade Amendment:**  
        *   Users have the ability to amend existing trades, including updating trade details, correcting errors, or modifying attributes as required.  
        *   All amendments are tracked with an audit trail for compliance and transparency.
    *   
        **Dynamic Attribute Management:**  
        *   Users can add attributes at runtime to Product Items during trade booking.  
        *   Attributes can be of various types, including text, strings, numbers, and documents.

7.  **Inventory Management:**  
    *   Dedicated page for inventory with advanced filtering options.

8.  **Mark Curves:**  
    *   Separate page for managing and viewing mark price curves and physical asset prices.

9.  **Profit & Loss Reporting:**  
    *   Generate daily profit and loss (P&L) reports based on mark prices.
    *   End-of-day (EOD) price marks must be retained and stored externally to CorTenX.
    *   The system must support secure, reliable storage and retrieval of historical EOD price marks.
    *   Traders must have the ability to view and analyze P&L over a customized period of time, leveraging the stored EOD price marks.

10.  **Transaction Limits:**  
    *   System enforces a limit on the number of transactions per account per month.

11.  **Automated Price Fetching:**  
    *   The system can automatically fetch real-time or historical pricing information by connecting to external APIs provided by exchanges, data vendors, or market aggregators.
    *   Fetched pricing data can be linked to the relevant Product Item during trade booking.

12.  **Helpdesk and AI Support:**  
    *   Integrated chat window for users to interact with an AI agent for troubleshooting and support.
    *   Escalation of broader or unresolved issues to a human helpdesk interface.

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
*   **Deployment:** Docker image, Desktop application (specifics TBD).
*   **Backend:** (To be defined - e.g., Python, Node.js, Go)
*   **Frontend:** (To be defined - e.g., React, Vue, Svelte, or desktop framework)
*   **Database:** (To be defined - e.g., SQLite for desktop, PostgreSQL for scalable solution)
*   **API:** REST / GraphQL for ERP Sync and Registry API.

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

#### Brand and Vision Alignment
*   The Freemium platform UI must reflect Trovio’s brand identity, including logo usage, color palette, typography, and iconography as defined in Trovio’s brand guidelines.
*   All user-facing components (desktop, Docker, and future SaaS) must maintain a consistent visual language that reinforces Trovio’s reputation for trust, innovation, and professionalism.
*   The user experience should embody Trovio’s vision of transparency, simplicity, and empowerment in ESG data management.
*   Visual design should communicate clarity, reliability, and accessibility, supporting both technical and non-technical users.
*   Reference and adhere to Trovio’s official style guide and digital asset library.
*   Collaborate with Trovio’s design team for review and approval of key UI components and screens.
*   Ensure all reports and dashboards generated by the platform are branded appropriately for external stakeholders and auditors.
*   Conduct design reviews with Trovio stakeholders at key milestones.
*   Include brand compliance checks in the UI/UX acceptance criteria and UAT process.

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