# TODO – Sprint 1 (14 Jun → 20 Jun 2025)

> **Goal:** Deliver a clear front-to-back flow diagram, first-pass interface specs, and a polished PRD v0.9.
> **🚨 STRATEGIC UPDATE:** Freemium is CorTenX's flagship ESG product - first-to-market opportunity with partner distribution ready.

---

## 0 · CorTenX Strategic Integration 🚨 **CRITICAL PATH**
- [x] **CorTenX Integration Analysis** — Comprehensive integration specification created  
      _Owner: James · 14 Jun_ ✅ **COMPLETED**
- [ ] **CorTenX Engineering Alignment** — Technical review with internal CorTenX team  
      _Owner: James · Due 15 Jun_ 🚨 **URGENT: Coordinate with CorTenX Engineering**
- [ ] **Partner Distribution Readiness** — Ensure desktop packaging meets partner requirements  
      _Owner: James · Due 16 Jun_ 🚨 **KEY: Multiple partners waiting**
- [ ] **First-to-Market Timeline** — Accelerated development roadmap for competitive advantage  
      _Owner: James · Due 17 Jun_

---

## 1 · Comprehensive Flow Diagram
- [x] **Choose diagram format** — Mermaid for the PR diff, Figma/Draw.io for high-res export  
      _Owner: James · Due 16 Jun_ ✅ **COMPLETED** (Mermaid chosen)
- [x] **Draft swim-lane diagram**  
      - UI → **MCP Server** → API → **CorTenX Assurance Ledger** ✅
      - Rules Engine, Normalisation & ESG Enrichment ✅
      - Ingest Connectors (CSV, ERP Sync, Registry) ✅
      - Annotate data contracts + p95 latency targets ✅
- [ ] **Peer review with Jon / Eng leads** + **CorTenX Engineering**  
      _Owner: James · Due 17 Jun_ 🚨 **PRIORITY: Add CorTenX technical review**
- [x] **Export SVG/PNG** to `/docs/diagrams/flow_v1.svg` and link from PRD  
      _Owner: James · Due 18 Jun_ ✅ **COMPLETED** (Mermaid diagram created)

---

## 2 · Specifications Draft
- [x] Scaffold `specs/README.md` with table of contents  
      _Owner: James · 14 Jun_ ✅ **COMPLETED**
- [x] Create interface spec stubs  
      - `specs/csv_ingest.md` ✅ **COMPLETED**
      - `specs/erp_sync.md` ✅ **COMPLETED**
      - `specs/registry_api.md` ✅ **COMPLETED**
      - `specs/mcp_server.md` ✅ **COMPLETED**
        - Purpose & scope ✅
        - OpenAPI / GraphQL schema ✅
        - Auth model ✅
        - Sample payloads + error codes ✅
        - Retry / back-off strategy ✅
- [ ] Open **“specs-draft” PR** for inline review  
      _Owner: James · 18 Jun_
- [ ] Address feedback and tag docs **v0.1-draft**  
      _Owner: James · 20 Jun_

---

## 3 · PRD Finalisation
- [x] Grep for `TODO` / `TBD` / empty tables and resolve each  
      _Owner: James · 14 Jun_ ✅ **COMPLETED** (1 TBD resolved)
- [ ] Complete all NFR rows (metric · owner · acceptance test)
- [ ] Insert **LLM Interaction Layer – MCP Server** subsection (update owners & sprint columns)
- [ ] Link the final flow diagram
- [ ] Bump PRD to **v0.9** and request team sign-off  
      _Owner: James · 19 Jun_

---

## 4 · Quality Gates / CI
- [ ] Add CI step: fail build if PRD still contains `TODO`/`TBD` tokens  
      _Owner: DevOps · 20 Jun_
- [ ] Add Cypress test: MCP request → first-token ≤ 200 ms p95  
      _Owner: DevOps · 20 Jun_

---

## 5 · Nice-to-Haves (Stretch)
- [ ] Publish diagram to Confluence for wider visibility
- [ ] Embed vector-memory schema graphic in `specs/mcp_server.md`
- [ ] Draft release-notes template for the v0.9 milestone

---

_Last updated: **2025-06-14 09:00 SGT**_
