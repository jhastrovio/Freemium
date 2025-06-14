# Freemium Architecture Flow Diagram v1.0

## Overview
This diagram illustrates the comprehensive data flow through the Freemium platform, from ingestion to user interaction.

## Mermaid Source

```mermaid
graph TD
    A["CSV Upload<br/>ERP Sync<br/>Registry API"] -->|Normalize| B["Data Ingest Layer"]
    B -->|Validate & Transform| C["Normalization & ESG Enrichment"]
    C -->|Rules Engine| D["Asset Reconciliation"]
    D -->|Emit Entries| E["Assurance Ledger"]
    
    F["UI Dashboard"] -->|User Query| G["MCP Server"]
    G -->|Context + Auth| H["LLM Provider<br/>(OpenAI/Local)"]
    H -->|Stream Response| G
    G -->|Real-time Updates| F
    
    E -->|Read Access| I["API Layer"]
    I -->|GraphQL/REST| F
    I -->|Data Context| G
    
    J["Vector Memory Cache"] -->|Conversation Context| G
    G -->|Store Embeddings| J
    
    K["External Registries<br/>(Verra, GATS)"] -->|Price Feeds<br/>Verification| A
    
    L["ERP Systems<br/>(SAP, NetSuite)"] -->|Real-time Sync<br/>Batch Import| A
    
    M["Telemetry & Monitoring"] -->|Collect Metrics| N["Grafana Dashboard"]
    G -->|Emit Metrics| M
    I -->|Performance Data| M
    
    style A fill:#e1f5fe
    style F fill:#f3e5f5
    style E fill:#fff3e0
    style G fill:#e8f5e8
    style H fill:#fce4ec
```

## Data Flow Description

### 1. Data Ingestion (Left Side)
- **Multiple Sources**: CSV files, ERP systems, and external registries feed into the platform
- **Normalization**: Raw data is validated, transformed, and enriched with ESG metadata
- **Reconciliation**: Rules engine processes assets and emits entries to the assurance ledger

### 2. User Interaction (Right Side)  
- **Dashboard UI**: Users interact through React-based interface
- **MCP Server**: Handles LLM interactions with proper auth and context packaging
- **Streaming**: Real-time responses from LLM providers (OpenAI or local models)

### 3. Core Data Store
- **Assurance Ledger**: Central repository of reconciled asset data
- **API Layer**: GraphQL/REST endpoints for data access
- **Vector Cache**: Conversation context for efficient follow-up queries

### 4. External Integrations
- **Registry APIs**: Real-time price feeds and certificate verification
- **ERP Sync**: Scheduled and real-time data synchronization
- **Monitoring**: Comprehensive telemetry collection and visualization

## Performance Annotations

### Latency Targets (p95)
- **MCP Server**: ≤ 200ms request→first-token
- **API Layer**: ≤ 100ms for standard queries
- **Data Ingestion**: ≤ 5s for CSV uploads (< 100MB)
- **ERP Sync**: ≤ 30s for batch operations

### Data Contracts
- **CSV Ingest**: UTF-8, max 500k rows, structured ESG metadata
- **ERP APIs**: OAuth2/API key auth, rate-limited, incremental sync
- **Registry APIs**: Certificate verification, price feeds, compliance data

## Security Boundaries
- **Tenant Isolation**: All data access scoped by tenant_id
- **Authentication**: JWT tokens for API access, mTLS for ERP connections
- **Data Protection**: Encryption at rest/transit, no cloud egress in desktop mode

## Technology Stack Mapping
- **Ingest Layer**: Kotlin/Spring Boot microservices
- **UI Layer**: React/Next.js with Vercel deployment
- **MCP Server**: Node.js/TypeScript with WebSocket streaming
- **Storage**: SQLite (desktop) / PostgreSQL (SaaS)
- **Vector Store**: SQLite-VSS (desktop) / pgvector (SaaS)

## Status
- **Version**: v1.0
- **Created**: 2025-06-14
- **Owner**: James
- **Review Status**: Ready for team review 