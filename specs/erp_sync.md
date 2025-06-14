# ERP Sync Specification

## Purpose & Scope
Define the integration patterns and protocols for synchronizing inventory data from ERP systems into Freemium.

**In Scope:**
- REST and GraphQL API integration patterns
- Authentication and authorization models
- Real-time and batch synchronization
- Common ERP system adapters (SAP, Oracle, NetSuite)
- Data mapping and transformation

**Out of Scope:**
- Direct database connections to ERP systems
- Custom ERP modifications
- Legacy system integrations requiring middleware

## Supported ERP Systems

### Tier 1 (Full Support)
- **SAP S/4HANA** - REST APIs, OData services
- **Oracle ERP Cloud** - REST APIs, SOAP services
- **NetSuite** - SuiteQL, REST APIs

### Tier 2 (Planned)
- Microsoft Dynamics 365
- Workday Financials
- Infor CloudSuite

## Authentication Models

### OAuth 2.0 (Preferred)
```yaml
auth_type: oauth2
grant_type: client_credentials
scope: "inventory:read commodities:read"
token_endpoint: "https://erp.company.com/oauth/token"
```

### API Key Authentication
```yaml
auth_type: api_key
key_location: header
key_name: "X-API-Key"
```

### mTLS (Enterprise)
```yaml
auth_type: mtls
client_cert_path: "/certs/client.pem"
client_key_path: "/certs/client.key"
ca_cert_path: "/certs/ca.pem"
```

## GraphQL Schema

### Core Query Types
```graphql
type Query {
  inventory(filters: InventoryFilters): [InventoryItem!]!
  commodities(type: CommodityType): [Commodity!]!
  transactions(dateRange: DateRange): [Transaction!]!
}

type InventoryItem {
  id: ID!
  sku: String!
  quantity: Float!
  unit: String!
  location: Location
  lastUpdated: DateTime!
  esgMetadata: ESGMetadata
}

type ESGMetadata {
  certifications: [String!]
  emissionFactor: Float
  originCountry: String
  traceabilityId: String
  sustainabilityScore: Int
}

input InventoryFilters {
  skus: [String!]
  locations: [String!]
  dateRange: DateRange
  commodityTypes: [CommodityType!]
}
```

### Mutation Types
```graphql
type Mutation {
  syncInventory(input: SyncInventoryInput!): SyncResult!
  updateESGData(input: ESGDataInput!): UpdateResult!
}

type SyncResult {
  jobId: ID!
  status: SyncStatus!
  recordsProcessed: Int!
  errors: [SyncError!]!
}
```

## REST API Specification

### Base Configuration
```yaml
base_url: "https://erp.company.com/api/v1"
rate_limit: 100/minute
timeout: 30s
retry_attempts: 3
```

### Endpoints

#### Get Inventory
```http
GET /inventory
Authorization: Bearer {access_token}
Content-Type: application/json

Query Parameters:
- limit: integer (max 1000)
- offset: integer
- updated_since: ISO 8601 datetime
- location_codes: comma-separated string
```

#### Sample Response
```json
{
  "data": [
    {
      "id": "INV-001",
      "sku": "CORN-US-001",
      "quantity": 1000.5,
      "unit": "tonnes",
      "location": {
        "code": "WH-001",
        "name": "Chicago Warehouse",
        "country": "US"
      },
      "last_updated": "2024-06-14T10:30:00Z",
      "esg_metadata": {
        "certification_standard": "Organic",
        "emission_factor": 2.45,
        "origin_country": "US",
        "traceability_id": "TR-CORN-001"
      }
    }
  ],
  "pagination": {
    "total": 1500,
    "limit": 100,
    "offset": 0,
    "has_more": true
  }
}
```

## Sync Strategies

### Real-time Sync (Webhooks)
```yaml
webhook_url: "https://freemium.company.com/api/v1/webhooks/erp"
events:
  - inventory.updated
  - inventory.created
  - inventory.deleted
retry_policy:
  max_attempts: 5
  backoff: exponential
```

### Batch Sync (Scheduled)
```yaml
schedule: "0 */6 * * *"  # Every 6 hours
batch_size: 1000
incremental: true
checkpoint_field: "last_updated"
```

### Manual Sync (On-demand)
```yaml
trigger: api_call
endpoint: "POST /api/v1/sync/erp/{connection_id}"
async: true
notification: webhook
```

## Error Codes & Handling

| Code | Description | Retry Strategy |
|------|-------------|----------------|
| `ERP_001` | Authentication failed | Refresh token |
| `ERP_002` | Rate limit exceeded | Exponential backoff |
| `ERP_003` | Invalid data format | Skip record, log error |
| `ERP_004` | Connection timeout | Retry with backoff |
| `ERP_005` | Resource not found | Skip, continue processing |

## Data Mapping Templates

### SAP S/4HANA
```yaml
mapping:
  asset_id: "Material + Plant"
  quantity: "UnrestrictedStock"
  unit: "BaseUnit" 
  location: "Plant"
  last_updated: "LastChangeDate"
  emission_factor: "ZZ_CO2_FACTOR"  # Custom field
```

### NetSuite
```yaml
mapping:
  asset_id: "item.itemId"
  quantity: "quantityOnHand"
  unit: "baseUnit"
  location: "location.name"
  last_updated: "lastModifiedDate"
```

## Sample Configuration File
```yaml
name: "Production ERP Sync"
erp_system: "sap_s4hana"
connection:
  base_url: "https://sap.company.com"
  auth:
    type: oauth2
    client_id: "${ERP_CLIENT_ID}"
    client_secret: "${ERP_CLIENT_SECRET}"
    token_endpoint: "https://sap.company.com/oauth/token"
sync:
  strategy: "batch"
  schedule: "0 2 * * *"  # Daily at 2 AM
  batch_size: 500
mapping_template: "sap_s4hana_default"
filters:
  material_types: ["FERT", "HAWA"]  # Finished goods, Trading goods
  plants: ["1000", "2000"]
```

## Status
- **Version:** v0.1-draft  
- **Last Updated:** 2025-06-14  
- **Owner:** James  
- **Review Status:** Draft for team review 