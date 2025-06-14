# CorTenX Integration Specification

## Overview
This specification defines how to integrate CorTenX (Trovio's tokenized asset ledger) as the immutable blockchain ledger within the Freemium ESG platform.

## Strategic Rationale

### Why CorTenX?
- **Immutable Audit Trail**: Blockchain-based ledger provides tamper-proof ESG provenance
- **Cryptographic Proof**: Multi-signature transactions ensure data integrity
- **Rich Metadata**: Native support for custom attributes (perfect for ESG data)
- **Enterprise Security**: Comprehensive access controls and permission management
- **API-First**: RESTful endpoints align with Freemium's architecture

### Position in Architecture
CorTenX provides a production-ready blockchain solution for immutable ledger functionality:

```
Data Ingest → Normalization → CorTenX Integration → CorTenX Ledger → API Layer → UI
```

## Data Model Mapping

### Freemium Asset → CorTenX Product
| Freemium Field | CorTenX Field | Type | Notes |
|----------------|---------------|------|-------|
| `asset_id` | `productId` | String | Unique identifier |
| `asset_type` | `attributes.type` | String | "commodity", "carbon_credit", "rec" |
| `quantity` | `holdings.quantity` | Integer | Amount held |
| `unit` | `attributes.unit` | String | "tonnes", "mwh", "credits" |
| `origin_country` | `attributes.origin_country` | String | Country code |
| `emission_factor` | `attributes.emission_factor` | Number | CO2e per unit |
| `acquisition_date` | `holdings.issuedTime` | DateTime | When acquired |

### ESG Metadata → CorTenX Attributes
```json
{
  "attributes": {
    "type": "commodity",
    "unit": "tonnes", 
    "origin_country": "US",
    "emission_factor": 2.45,
    "certification_standard": "Organic",
    "traceability_id": "TR-CORN-001",
    "sustainability_score": 85,
    "deforestation_risk": "Low"
  },
  "metadata": {
    "freemium:created_by": "csv_ingest_v1",
    "freemium:tenant_id": "tenant-uuid",
    "freemium:source_file": "inventory_2024_06.csv"
  }
}
```

## Integration Architecture

### 1. CorTenX Client Layer
```typescript
interface CorTenXClient {
  // Product Management
  createProduct(request: CreateProductRequest): Promise<ProductData>;
  getProduct(productId: string): Promise<ProductData>;
  queryProducts(filters: ProductFilters): Promise<ProductData[]>;
  
  // Holdings Management  
  issueHolding(request: IssueHoldingRequest): Promise<HoldingData>;
  listHoldings(accountId: string, filters?: HoldingFilters): Promise<HoldingData[]>;
  transferHolding(request: TransferHoldingRequest): Promise<TransactionData>;
  
  // Metadata & Attributes
  setProductAttributes(productId: string, attributes: Record<string, any>): Promise<void>;
  setMetadata(entityRef: string, metadata: Record<string, any>): Promise<void>;
}
```

### 2. Asset Synchronization Service
```kotlin
@Service
class AssetSyncService(
    private val cortenxClient: CorTenXClient,
    private val assetRepository: AssetRepository
) {
    suspend fun syncAssetToCorTenX(asset: Asset): String {
        // 1. Create CorTenX product
        val productRequest = CreateProductRequest(
            code = asset.assetId,
            name = asset.description ?: asset.assetId,
            attributes = mapAssetToAttributes(asset)
        )
        val product = cortenxClient.createProduct(productRequest)
        
        // 2. Issue holdings for quantity
        val holdingRequest = IssueHoldingRequest(
            productId = product.productId,
            accountId = asset.tenantId,  // Map tenant to CorTenX account
            quantity = asset.quantity.toBigInteger(),
            attributes = mapESGAttributes(asset)
        )
        val holding = cortenxClient.issueHolding(holdingRequest)
        
        // 3. Update local asset with CorTenX references
        asset.cortenxProductId = product.productId
        asset.cortenxHoldingId = holding.holdingId
        assetRepository.save(asset)
        
        return product.productId
    }
    
    private fun mapAssetToAttributes(asset: Asset): Map<String, Any> {
        return mapOf(
            "type" to asset.type,
            "unit" to asset.unit,
            "origin_country" to asset.originCountry,
            "emission_factor" to asset.emissionFactor,
            "certification_standard" to asset.certificationStandard,
            "traceability_id" to asset.traceabilityId
        ).filterValues { it != null }
    }
}
```

### 3. Query Integration
```typescript
// Enhanced query service with CorTenX integration
class AssetQueryService {
  async queryAssets(filters: AssetFilters): Promise<Asset[]> {
    // 1. Query CorTenX for holdings
    const cortenxFilters = {
      accountId: filters.tenantId,
      attributes: {
        type: filters.assetType,
        origin_country: filters.originCountry
      },
      metadata: {
        [`freemium:tenant_id`]: filters.tenantId
      }
    };
    
    const holdings = await this.cortenxClient.listHoldings(filters.tenantId, cortenxFilters);
    
    // 2. Transform to Freemium asset format
    return holdings.map(holding => ({
      assetId: holding.product?.data.code || holding.productId,
      type: holding.product?.attributes.type,
      quantity: holding.quantity,
      unit: holding.product?.attributes.unit,
      originCountry: holding.product?.attributes.origin_country,
      emissionFactor: holding.product?.attributes.emission_factor,
      cortenxProductId: holding.productId,
      cortenxHoldingId: holding.holdingId,
      verificationHash: holding.issuedIn // Blockchain proof
    }));
  }
}
```

## API Integration Points

### 1. Asset Creation Flow

🔄 **[CorTenX Integration Flow Diagram](../docs/diagrams/cortenx_integration_flow.md)** - Detailed sequence diagram showing asset creation and blockchain synchronization.

### 2. Audit Trail Access
```http
GET /api/v1/assets/{assetId}/audit-trail
Authorization: Bearer {jwt_token}

Response:
{
  "asset_id": "CORN-US-2024-001",
  "cortenx_product_id": "41756:0",
  "cortenx_holding_id": "41757:0", 
  "blockchain_proof": "0x233c04beef01370ef4b0e2bb6301cda077666ed81a76dc5e93aacd830e59bb36",
  "audit_trail": [
    {
      "timestamp": "2024-06-14T10:30:00Z",
      "transaction_id": "41758:0",
      "action": "issued",
      "quantity": 1000,
      "attributes": { "emission_factor": 2.45 }
    }
  ]
}
```

### 3. ESG Reporting Enhancement
```typescript
// Enhanced ESG report with blockchain verification
interface ESGReport {
  summary: {
    total_assets: number;
    total_emissions: number;
    verified_percentage: number; // % with blockchain proof
  };
  assets: Array<{
    asset_id: string;
    quantity: number;
    emission_factor: number;
    blockchain_verified: boolean;
    verification_hash?: string;
    cortenx_proof_url?: string; // Link to blockchain explorer
  }>;
  verification: {
    total_verified_assets: number;
    blockchain_integrity_check: "PASSED" | "FAILED";
    last_verification: string;
  };
}
```

## Configuration & Deployment

### Environment Configuration
```yaml
# application.yml
cortenx:
  enabled: true
  base_url: "https://corten-demo.dev.trovio.io"
  auth:
    type: "access_token"  # or "jwt"
    access_key_id: "${CORTENX_ACCESS_KEY_ID}"
    private_key_path: "${CORTENX_PRIVATE_KEY_PATH}"
  
  default_issuer_id: "${CORTENX_ISSUER_ID}"
  tenant_account_mapping:
    default: "${CORTENX_DEFAULT_ACCOUNT_ID}"
  
  sync:
    batch_size: 100
    retry_attempts: 3
    timeout_seconds: 30
```

### Docker Integration
```dockerfile
# Add CorTenX SDK dependencies
FROM openjdk:17-jdk-alpine
COPY --from=cortenx-sdk /libs/*.jar /app/lib/
ENV CORTENX_ENABLED=true
ENV CORTENX_BASE_URL=https://localhost:8080
```

## Security Considerations

### Access Control Mapping
```typescript
// Map Freemium roles to CorTenX permissions
const PERMISSION_MAPPING = {
  "admin": ["Request", "Product", "Account"],
  "analyst": ["Request"], 
  "auditor": ["Request", "GetHoldingHistory"],
  "viewer": ["Request"]
};

async function mapFreemiumToCorTenXPermissions(
  freemiumUser: User
): Promise<CorTenXPermissions> {
  return {
    accountId: freemiumUser.tenantId,
    permissions: PERMISSION_MAPPING[freemiumUser.role] || ["Request"]
  };
}
```

### Data Protection
- **Tenant Isolation**: Each tenant maps to separate CorTenX account
- **Encryption**: All API calls use TLS 1.3
- **Key Management**: Private keys stored in secure vault
- **Audit Logging**: All CorTenX interactions logged

## Performance Considerations

### Caching Strategy
```typescript
// Redis cache for CorTenX responses
class CachedCorTenXClient {
  constructor(
    private cortenxClient: CorTenXClient,
    private redis: Redis
  ) {}
  
  async getProduct(productId: string): Promise<ProductData> {
    const cached = await this.redis.get(`cortenx:product:${productId}`);
    if (cached) return JSON.parse(cached);
    
    const product = await this.cortenxClient.getProduct(productId);
    await this.redis.setex(`cortenx:product:${productId}`, 300, JSON.stringify(product));
    return product;
  }
}
```

### Async Processing
```typescript
// Background sync for large datasets
@Queue('cortenx-sync')
class CorTenXSyncProcessor {
  @Process('sync-assets')
  async syncAssetsBatch(job: Job<{assets: Asset[]}>) {
    const { assets } = job.data;
    
    for (const asset of assets) {
      try {
        await this.assetSyncService.syncAssetToCorTenX(asset);
        await job.progress((assets.indexOf(asset) + 1) / assets.length * 100);
      } catch (error) {
        console.error(`Failed to sync asset ${asset.assetId}:`, error);
        // Continue with next asset
      }
    }
  }
}
```

## Migration Strategy

### Phase 1: Proof of Concept (2 weeks)
- Set up CorTenX development environment
- Implement basic product creation and holding issuance
- Create mapping for top 3 asset types

### Phase 2: Core Integration (4 weeks)
- Full asset synchronization service
- Enhanced query service with CorTenX data
- Audit trail API endpoints

### Phase 3: Advanced Features (3 weeks)
- Blockchain verification in reports
- Real-time sync for ERP integrations
- Performance optimization and caching

### Phase 4: Production Rollout (2 weeks)
- Security audit and penetration testing
- Performance testing at scale
- Documentation and training

## Testing Strategy

### Integration Tests
```kotlin
@SpringBootTest
class CorTenXIntegrationTest {
    @Test
    fun `should create product and issue holding`() {
        val asset = Asset(
            assetId = "TEST-CORN-001",
            type = "commodity",
            quantity = 100.0,
            unit = "tonnes"
        )
        
        val productId = cortenxService.syncAssetToCorTenX(asset)
        
        assertThat(productId).isNotNull()
        val product = cortenxClient.getProduct(productId)
        assertThat(product.data.code).isEqualTo("TEST-CORN-001")
    }
}
```

## Monitoring & Observability

### Key Metrics
```yaml
metrics:
  - name: "cortenx_api_requests_total"
    type: "counter"
    labels: ["endpoint", "status", "tenant"]
  
  - name: "cortenx_sync_duration_seconds"
    type: "histogram"
    labels: ["operation", "asset_type"]
  
  - name: "cortenx_blockchain_verification_status"
    type: "gauge"
    labels: ["tenant", "status"]
```

### Alerts
```yaml
alerts:
  - alert: "CorTenXAPIDown"
    expr: "up{job='cortenx-api'} == 0"
    for: "2m"
    severity: "critical"
  
  - alert: "CorTenXSyncLag"
    expr: "cortenx_sync_lag_seconds > 300"
    for: "5m"
    severity: "warning"
```

## Cost Estimation

### CorTenX Usage Costs (Estimated)
- **API Calls**: ~$0.01 per transaction
- **Storage**: ~$0.001 per KB of metadata
- **Monthly**: ~$50-200 for small deployment (1000 assets)
- **Enterprise**: ~$500-2000 for large deployment (100k+ assets)

## Next Steps

1. **Set up CorTenX development access** 
2. **Create POC branch** with basic integration
3. **Update PRD** to reflect CorTenX as blockchain ledger
4. **Adjust Sprint planning** to include integration milestones

## Status
- **Version:** v0.1-draft  
- **Created:** 2025-06-14  
- **Owner:** James  
- **Review Status:** Needs team evaluation 