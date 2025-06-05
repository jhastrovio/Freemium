# Project Definition Report (v0)

## 1. Overview
Freenium is a **modular, APIfirst inventory platform** for ESG‑critical commodities and digital environmental assets, designed to land fast via freemium desktop and scale to SaaS + premium analytics fileciteturn1file2L7-L14.

## 2. Architecture Diagram
*(embed `/diagrams/freenium_arch_v0.png`)* – Layers: Ingest Connectors → Normalisation & ESG Enrichment → Assurance Ledger → API / UI & Dashboards.

## 3. Interfaces / Data Contracts
| Interface | Protocol | Spec |
|-----------|----------|------|
| CSV Ingest | Local file | `specs/csv_ingest.md` |
| ERP Sync | REST / GraphQL | `specs/erp_sync.md` |
| Registry API (carbon/REC) | REST | `specs/registry_api.md` |

## 4. Non‑Functional Requirements
| Area | Target | Validation |
|------|--------|------------|
| Latency (p95) | ≤ 200 ms | k6 load test |
| Offline‑first | Full desktop | QA scenario matrix |
| Security | ISO 27001 controls | Pen‑testing |

## 5. Hand‑Off Checklist
- [ ] Code reviewed & merged to `main` with >80 % test cover.
- [ ] IaC validated (Terraform plan clean).
- [ ] NFRs met (latency, security, offline).
- [ ] PDR sign‑off by Product & CTO.