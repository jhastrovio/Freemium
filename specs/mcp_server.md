# MCP Server Specification

## Purpose & Scope
Define the Model Context Protocol (MCP) server implementation for LLM interaction layer within the Freemium platform.

**In Scope:**
- Context packaging and serialization
- WebSocket streaming interface
- Vector memory cache for conversation context
- Authorization and security controls
- Telemetry and observability
- Desktop and SaaS deployment patterns

**Out of Scope:**
- LLM model training or fine-tuning
- Custom LLM inference engines
- Multi-tenant data isolation (handled by platform layer)

## Position in Architecture Flow

```
Client UI ↔ MCP Server ↔ Edge/API ↔ LLM Vendor/Local Model
          ↕                ↕
    Vector Cache    CorTenX Ledger
```

**Data Flow:**
1. UI sends workspace context + query
2. MCP Server adds auth metadata + vector lookups
3. Enriched context forwarded to LLM
4. Streaming response back through MCP to UI
5. Context cached for follow-up queries

## Core Responsibilities

### 1. Context Packaging
Serialize current application state into structured JSON for LLM consumption.

```typescript
interface ContextV1 {
  session_id: string;
  user_id: string;
  tenant_id?: string;
  timestamp: string;
  workspace: {
    active_filters: FilterSet;
    visible_rows: AssetRow[];
    current_view: "dashboard" | "inventory" | "reports";
    draft_queries: string[];
  };
  conversation_history: Message[];
  vector_context?: VectorMatch[];
}
```

### 2. Authorization Guard
Enforce identical security controls as Core API before exposing data to LLM.

```typescript
interface AuthContext {
  jwt_token: string;
  tenant_id: string;
  permissions: string[];
  data_access_scope: string[];
}

// Validation flow
async function validateRequest(context: ContextV1, auth: AuthContext): Promise<boolean> {
  // 1. Verify JWT signature and expiry
  // 2. Check tenant isolation
  // 3. Validate data access permissions
  // 4. Rate limiting per user/tenant
  return true;
}
```

### 3. Vector Memory Cache
Store conversation context and asset data for efficient follow-up queries.

```sql
-- Vector schema
CREATE TABLE vector_memory (
  id UUID PRIMARY KEY,
  tenant_id VARCHAR(255),
  session_id VARCHAR(255),
  prompt_hash VARCHAR(64),
  asset_ids TEXT[], -- Array of referenced asset IDs
  embedding VECTOR(1536), -- OpenAI ada-002 dimensions
  metadata JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  expires_at TIMESTAMP
);

-- Indexes
CREATE INDEX idx_vector_tenant_session ON vector_memory(tenant_id, session_id);
CREATE INDEX idx_vector_embedding ON vector_memory USING hnsw (embedding vector_cosine_ops);
```

### 4. Streaming Interface
Provide real-time token streaming to meet latency requirements.

#### WebSocket Endpoint
```typescript
// WS /chat
interface ChatMessage {
  type: "context" | "query" | "response" | "error";
  session_id: string;
  data: any;
  timestamp: string;
}

// Connection flow
ws.onmessage = (event) => {
  const message: ChatMessage = JSON.parse(event.data);
  switch (message.type) {
    case "context":
      // Update session context
      break;
    case "query":
      // Stream LLM response
      streamLLMResponse(message.data);
      break;
  }
};
```

#### REST Endpoint
```http
POST /v1/context
Authorization: Bearer {jwt_token}
Content-Type: application/json

{
  "session_id": "session-uuid",
  "context": { /* ContextV1 object */ },
  "query": "Show me carbon credits from Brazil",
  "stream": true
}
```

### 5. Telemetry Hook
Emit comprehensive metrics for monitoring and alerting.

```typescript
interface TelemetryEvent {
  event_type: "llm.request" | "llm.response" | "llm.error";
  session_id: string;
  tenant_id: string;
  timestamp: string;
  duration_ms: number;
  token_count: {
    input: number;
    output: number;
    total: number;
  };
  cost_usd?: number;
  model_name: string;
  error_code?: string;
}

// Kafka topic: llm.telemetry
await kafka.send("llm.telemetry", telemetryEvent);
```

## API Specification

### Authentication
All endpoints require valid JWT token with appropriate scopes.

```yaml
security:
  - bearerAuth: []
    scopes: ["llm:query", "inventory:read"]
```

### Core Endpoints

#### Health Check
```http
GET /health
```

#### Create Chat Session
```http
POST /v1/sessions
Content-Type: application/json

{
  "workspace_context": { /* ContextV1 */ }
}

Response:
{
  "session_id": "uuid-v4",
  "expires_at": "2024-06-14T16:00:00Z"
}
```

#### Query with Context
```http
POST /v1/sessions/{session_id}/query
Content-Type: application/json

{
  "query": "What are my highest emission assets?",
  "stream": true,
  "context_refresh": false
}

Response (Streaming):
data: {"type": "token", "content": "Based"}
data: {"type": "token", "content": " on your"}  
data: {"type": "token", "content": " inventory..."}
data: {"type": "complete", "total_tokens": 150}
```

#### Vector Search
```http
POST /v1/vector/search
Content-Type: application/json

{
  "query": "carbon credits Brazil",
  "limit": 10,
  "similarity_threshold": 0.8
}
```

## Non-Functional Requirements

| Metric | Target | Validation Method |
|--------|--------|-------------------|
| **Latency (p95)** | ≤ 200ms request→first-token | Load testing with k6 |
| **Throughput** | 50 concurrent WebSocket sessions/tenant | Stress testing |
| **Availability** | 99.9% uptime | Health checks + alerts |
| **Security** | Zero data leakage between tenants | Penetration testing |
| **Observability** | All requests traced | OpenTelemetry integration |

### Performance Budgets
```yaml
response_times:
  auth_validation: "< 50ms"
  vector_lookup: "< 100ms" 
  llm_first_token: "< 200ms p95"
  total_response: "< 5s p99"

resource_limits:
  memory_per_session: "256MB"
  concurrent_sessions: 50
  vector_cache_size: "1GB"
  token_rate_limit: "10k/hour/user"
```

## Deployment Patterns

### SaaS Deployment
```yaml
# Docker Compose / Kubernetes
services:
  mcp-server:
    image: freemium/mcp-server:latest
    environment:
      - NODE_ENV=production
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - POSTGRES_URL=${POSTGRES_URL}
      - REDIS_URL=${REDIS_URL}
    resources:
      limits:
        memory: 2Gi
        cpu: 1000m
      requests:
        memory: 512Mi
        cpu: 500m
    ports:
      - "8080:8080"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### Desktop Deployment
```yaml
# Embedded binary configuration
desktop:
  model_type: "gguf"
  model_path: "./models/llama-7b.gguf"
  database: "sqlite"
  vector_store: "sqlite-vss"
  memory_limit: "4GB"
  gpu_acceleration: "auto"
```

## Local Model Configuration

### GGUF Model Support
```typescript
interface LocalModelConfig {
  model_path: string;
  context_length: number;
  gpu_layers: number;
  threads: number;
  temperature: number;
  max_tokens: number;
}

// Example: Llama 7B
const llamaConfig: LocalModelConfig = {
  model_path: "./models/llama-2-7b-chat.gguf",
  context_length: 4096,
  gpu_layers: 32,
  threads: 8,
  temperature: 0.7,
  max_tokens: 512
};
```

### Model Selection Matrix
| Use Case | Model | Size | Performance | Memory |
|----------|-------|------|-------------|--------|
| Desktop Quick | Llama-7B | 4GB | Good | 8GB RAM |
| Desktop Premium | Llama-13B | 8GB | Better | 16GB RAM |
| SaaS Standard | GPT-4o-mini | API | Excellent | N/A |
| SaaS Premium | GPT-4o | API | Best | N/A |

## Error Handling

### Error Codes
| Code | Description | Recovery Action |
|------|-------------|-----------------|
| `MCP_001` | Invalid JWT token | Refresh authentication |
| `MCP_002` | Rate limit exceeded | Exponential backoff |
| `MCP_003` | Vector search failed | Fallback to non-vector query |
| `MCP_004` | LLM timeout | Retry with reduced context |
| `MCP_005` | Insufficient permissions | Request access upgrade |

### Error Response Format
```json
{
  "error": {
    "code": "MCP_002",
    "message": "Rate limit exceeded",
    "details": {
      "limit": 1000,
      "window": "1h",
      "retry_after": 300
    },
    "request_id": "req-uuid-v4"
  }
}
```

## Security Considerations

### Data Protection
- All context data encrypted at rest and in transit
- Vector embeddings anonymized (no PII in vector space)
- Session data auto-expires after 24 hours
- Audit logging for all LLM interactions

### Tenant Isolation
```typescript
// Ensure tenant-scoped queries
function buildQuery(context: ContextV1, auth: AuthContext): string {
  return `
    SELECT * FROM assets 
    WHERE tenant_id = '${auth.tenant_id}'
    AND ${buildPermissionFilters(auth.permissions)}
    ${buildContextFilters(context.workspace.active_filters)}
  `;
}
```

## Monitoring & Alerting

### Key Metrics
```yaml
metrics:
  - name: "mcp_request_duration"
    type: "histogram"
    labels: ["endpoint", "tenant", "status"]
  
  - name: "mcp_active_sessions"
    type: "gauge"
    labels: ["tenant"]
  
  - name: "mcp_token_usage"
    type: "counter" 
    labels: ["model", "tenant", "type"]
  
  - name: "mcp_vector_cache_hits"
    type: "counter"
    labels: ["tenant"]
```

### Alerts
```yaml
alerts:
  - alert: "MCPHighLatency"
    expr: "histogram_quantile(0.95, mcp_request_duration) > 0.25"
    for: "5m"
    severity: "warning"
  
  - alert: "MCPErrorRate"
    expr: "rate(mcp_requests_total{status=~\"5..\"}[5m]) > 0.05"
    for: "2m"
    severity: "critical"
```

## Sample Implementation

### TypeScript Server Structure
```
src/
├── server.ts              # Express app + WebSocket server
├── auth/
│   ├── middleware.ts      # JWT validation
│   └── permissions.ts     # RBAC logic
├── context/
│   ├── serializer.ts      # ContextV1 builder
│   └── validator.ts       # Input validation
├── vector/
│   ├── store.ts           # Vector database client
│   ├── embeddings.ts      # OpenAI embeddings
│   └── search.ts          # Similarity search
├── llm/
│   ├── client.ts          # LLM provider abstraction
│   ├── openai.ts          # OpenAI integration
│   └── local.ts           # GGUF model integration
├── streaming/
│   ├── websocket.ts       # WS message handling
│   └── sse.ts             # Server-sent events
└── telemetry/
    ├── metrics.ts         # Prometheus metrics
    └── tracing.ts         # OpenTelemetry spans
```

## Testing Strategy

### Unit Tests
- Context serialization/deserialization
- Authorization middleware
- Vector similarity calculations
- LLM response streaming

### Integration Tests
- End-to-end WebSocket flows
- Database connectivity
- External LLM API calls
- Authentication integration

### Performance Tests
```yaml
load_tests:
  - name: "concurrent_sessions"
    users: 50
    duration: "5m"
    endpoints: ["POST /v1/sessions/{id}/query"]
  
  - name: "vector_search_load"
    rps: 100
    duration: "2m"
    endpoint: "POST /v1/vector/search"
```

## Open Items & Owners

| Item | Owner | Sprint | Priority |
|------|-------|--------|----------|
| Vector schema finalized | @data-lead | SPRINT-2 | High |
| Latency budget test harness | @devops | SPRINT-3 | High |
| Desktop GGUF model selection | @ml-lead | SPRINT-3 | Medium |
| Multi-tenant vector isolation | @security | SPRINT-4 | High |
| Cost monitoring dashboard | @ops | SPRINT-4 | Medium |

## Status
- **Version:** v0.1-draft  
- **Last Updated:** 2025-06-14  
- **Owner:** James  
- **Review Status:** Draft for team review 