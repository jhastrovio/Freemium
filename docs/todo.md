# TODO – Sprint 1 (14 Jun → 20 Jun 2025)

> **Goal:** Deliver a clear front-to-back flow diagram, first-pass interface specs, and a polished PRD v1.0.
> **🚨 STRATEGIC UPDATE:** Freemium is CorTenX's flagship ESG product - first-to-market opportunity with partner distribution ready.
> **🎯 ARCHITECTURE CLARIFIED:** Local deployment (desktop/Docker) with CorTenX cloud foundation + optional MCP add-on.

---

## 0 · CorTenX Strategic Integration 🚨 **CRITICAL PATH**
- [x] **CorTenX Integration Analysis** — Comprehensive integration specification created  
      _Owner: James · 14 Jun_ ✅ **COMPLETED**
- [x] **CorTenX Architecture Positioning** — Clarified as foundational backbone (not external)  
      _Owner: James · 14 Jun_ ✅ **COMPLETED**
- [ ] **CorTenX Engineering Alignment** — Technical review with internal CorTenX team  
      _Owner: James · Due 15 Jun_ 🚨 **URGENT: Coordinate with CorTenX Engineering**
- [ ] **First-to-Market Timeline** — Accelerated development roadmap for competitive advantage  
      _Owner: James · Due 17 Jun_

---

## 1 · Comprehensive Flow Diagram
- [x] **Choose diagram format** — Mermaid for the PR diff, Figma/Draw.io for high-res export  
      _Owner: James · Due 16 Jun_ ✅ **COMPLETED** (Mermaid chosen)
- [x] **Draft flow diagram**  
      - Local App → CorTenX Cloud API → Blockchain Ledger ✅
      - Rules Engine, Normalisation & ESG Enrichment ✅
      - Ingest Connectors (CSV, ERP Sync, Registry) ✅
      - Optional MCP Server add-on ✅
      - Annotate data contracts + p95 latency targets ✅
- [x] **Architecture alignment** — Updated to reflect local+cloud hybrid approach  
      _Owner: James · 14 Jun_ ✅ **COMPLETED** (v1.1 updated)
- [ ] **Peer review with Jon / Eng leads** + **CorTenX Engineering**  
      _Owner: James · Due 17 Jun_ 🚨 **PRIORITY: Add CorTenX technical review**
- [x] **Export to diagrams** — Created comprehensive Mermaid diagrams in `/docs/diagrams/`  
      _Owner: James · Due 18 Jun_ ✅ **COMPLETED** (flow_v1.md, system_architecture.md)

---

## 2 · Specifications Draft
- [x] Scaffold `specs/README.md` with table of contents  
      _Owner: James · 14 Jun_ ✅ **COMPLETED**
- [x] Create interface spec stubs  
      - `specs/csv_ingest.md` ✅ **COMPLETED**
      - `specs/erp_sync.md` ✅ **COMPLETED**
      - `specs/registry_api.md` ✅ **COMPLETED**
      - `specs/cortenx_integration.md` ✅ **COMPLETED**
      - `specs/mcp_server.md` ✅ **COMPLETED** (positioned as optional add-on)
        - Purpose & scope ✅
        - OpenAPI / GraphQL schema ✅
        - Auth model ✅
        - Sample payloads + error codes ✅
        - Retry / back-off strategy ✅
- [ ] **Specifications alignment review** — Update all specs to reflect local+cloud architecture  
      _Owner: James · Due 18 Jun_ 🚨 **PRIORITY: Architecture alignment needed**
- [ ] Open **“specs-draft” PR** for inline review  
      _Owner: James · 18 Jun_
- [ ] Address feedback and tag docs **v0.1-draft**  
      _Owner: James · 20 Jun_

---

## 3 · PRD Finalisation
- [x] Grep for `TODO` / `TBD` / empty tables and resolve each  
      _Owner: James · 14 Jun_ ✅ **COMPLETED** (1 TBD resolved)
- [x] **Architecture clarification** — Updated to local+cloud hybrid with CorTenX foundation  
      _Owner: James · 14 Jun_ ✅ **COMPLETED** (Major PRD updates)
- [x] **MCP Server positioning** — Clarified as optional add-on feature  
      _Owner: James · 14 Jun_ ✅ **COMPLETED** (Section 7 updated)
- [x] Link the final flow diagram ✅ **COMPLETED** (All diagrams consolidated in /docs/diagrams/)
- [ ] Complete all NFR rows (metric · owner · acceptance test)
- [ ] **Executive Brief completion** — Created comprehensive market analysis and technical feasibility  
      _Owner: James · Due 18 Jun_ ✅ **COMPLETED** (Ready for CEO/CTO presentation)
- [ ] Bump PRD to **v1.0** and request team sign-off  
      _Owner: James · 19 Jun_

---

## 4 · Quality Gates / CI
- [ ] Add CI step: fail build if PRD still contains `TODO`/`TBD` tokens  
      _Owner: DevOps · 20 Jun_
- [ ] Add integration tests: CorTenX API connectivity and blockchain verification  
      _Owner: DevOps · 20 Jun_
- [ ] Add external API tests: ERP sync and Registry API connectivity  
      _Owner: DevOps · 20 Jun_
- [ ] **Optional**: Add Cypress test for MCP add-on: request → first-token ≤ 200 ms p95  
      _Owner: DevOps · 20 Jun_ (Only if MCP add-on is enabled)

---

## 5 · Nice-to-Haves (Stretch)
- [ ] Publish diagram to Confluence for wider visibility
- [ ] Embed vector-memory schema graphic in `specs/mcp_server.md`
- [ ] Draft release-notes template for the v1.0 milestone
- [ ] Create deployment guides for both desktop and Docker options
- [ ] System architecture diagram alignment with updated flow

---

---

## 📊 **Current Status Summary**
- ✅ **Major Architecture Clarification**: Local+cloud hybrid with CorTenX foundation
- ✅ **Flow Diagram**: Updated to v1.1 with corrected architecture  
- ✅ **PRD Updates**: Significant architecture and positioning updates
- ✅ **Specifications**: All core specs created (alignment review needed)
- ✅ **Executive Brief**: Comprehensive market analysis completed
- 🚨 **Next Priority**: CorTenX Engineering alignment and specs architecture review

_Last updated: **2025-06-14 16:00 SGT**_
