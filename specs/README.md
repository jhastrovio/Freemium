# Freemium Technical Specifications

## Overview
This directory contains detailed technical specifications for the Freemium platform components and interfaces.

## Table of Contents

### Core Interfaces
- [CSV Ingest Specification](./csv_ingest.md) - File upload and parsing specifications
- [ERP Sync Specification](./erp_sync.md) - ERP system integration patterns
- [Registry API Specification](./registry_api.md) - Carbon/REC registry integrations
- [MCP Server Specification](./mcp_server.md) - Model Context Protocol server implementation
- [CorTenX Integration Specification](./cortenx_integration.md) - Blockchain assurance ledger integration

### Data Contracts
- Asset normalization schemas
- ESG metadata formats
- Assurance ledger entry specifications

### Quality Gates
- Performance targets (p95 ≤ 200ms)
- Security requirements
- Retry and back-off strategies

## Status
- **Version:** 0.1-draft
- **Last Updated:** 2025-06-14
- **Review Status:** Draft for team review

## Contributing
All specifications follow OpenAPI 3.0 standards where applicable. See individual spec files for detailed schemas and examples. 