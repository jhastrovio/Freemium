# Registry API Specification

## Purpose & Scope
Define the integration patterns for connecting to environmental commodity registries (Carbon Credits, RECs, etc.).

**In Scope:**
- Carbon credit registry APIs (Verra, Gold Standard, CAR, ACR)
- Renewable Energy Certificate (REC) registries
- Certificate verification and validation
- Real-time price feeds and market data
- Registry transaction history

**Out of Scope:**
- Trading and transaction execution
- Registry account management
- Direct blockchain integrations

## Supported Registries

### Carbon Credit Registries
- **Verra (VCS)** - Verified Carbon Standard
- **Gold Standard** - Gold Standard Certified
- **Climate Action Reserve (CAR)** - California compliance
- **American Carbon Registry (ACR)** - Voluntary carbon market

### REC Registries  
- **PJM-GATS** - Generation Attribute Tracking System
- **NEPOOL-GIS** - Generation Information System
- **WREGIS** - Western Renewable Energy Generation Information System
- **M-RETS** - Midwest Renewable Energy Tracking System

## Authentication Models

### API Key (Most Common)
```yaml
auth_type: api_key
key_location: header
key_name: "X-Registry-API-Key"
rate_limit: 1000/hour
```

### OAuth 2.0 (Institutional)
```yaml
auth_type: oauth2
grant_type: client_credentials
scope: "registry:read certificates:verify"
token_endpoint: "https://registry.api.com/oauth/token"
```

### Certificate-based (Enterprise)
```yaml
auth_type: client_certificate
cert_path: "/certs/registry_client.pem"
key_path: "/certs/registry_client.key"
```

## REST API Specifications

### Verra VCS Registry

#### Base Configuration
```yaml
base_url: "https://registry.verra.org/mymodule/rpt/myrpt.asp"
rate_limit: 500/hour
timeout: 30s
```

#### Get Carbon Credits
```http
GET /api/v1/credits
Authorization: Bearer {access_token}
Content-Type: application/json

Query Parameters:
- project_id: string
- vintage_year: integer (YYYY)
- credit_type: string (VCU, VER)
- status: string (active, retired, cancelled)
- limit: integer (max 1000)
```

#### Sample Response
```json
{
  "data": [
    {
      "credit_id": "VCS-001-2024-12345",
      "project_id": "2948",
      "project_name": "Amazon Forest Conservation",
      "methodology": "VM0015",
      "vintage_year": 2024,
      "quantity": 1000,
      "unit": "tCO2e",
      "status": "active",
      "issuance_date": "2024-06-01T00:00:00Z",
      "retirement_date": null,
      "price_usd": 15.50,
      "registry": "Verra",
      "verification_standard": "VCS",
      "additional_certifications": ["CCBS", "SD-VISta"],
      "location": {
        "country": "BR",
        "region": "Amazonas",
        "coordinates": [-3.4653, -62.2159]
      }
    }
  ],
  "pagination": {
    "total": 2500,
    "page": 1,
    "limit": 100,
    "has_more": true
  }
}
```

### PJM-GATS (REC Registry)

#### Base Configuration  
```yaml
base_url: "https://gats.pjm.com/api/v2"
rate_limit: 200/hour
timeout: 45s
```

#### Get RECs
```http
GET /certificates
Authorization: X-API-Key {api_key}
Content-Type: application/json

Query Parameters:
- fuel_type: string (Solar, Wind, Hydro, Biomass)
- generation_period: string (YYYY-MM)
- state: string (state abbreviation)
- eligibility: string (Class1, Class2)
```

#### Sample Response
```json
{
  "certificates": [
    {
      "certificate_id": "PJM-REC-2024-987654",
      "generator_id": "GEN-12345",
      "generator_name": "Sunrise Solar Farm",
      "fuel_type": "Solar",
      "generation_period": "2024-05",
      "mwh": 1000,
      "state": "PA",
      "eligibility": "Class1",
      "vintage": "2024",
      "status": "active",
      "issuance_date": "2024-06-01T10:00:00Z",
      "market_value_usd": 45.00
    }
  ]
}
```

## Data Synchronization

### Real-time Updates (Webhooks)
```yaml
webhook_endpoints:
  - url: "https://freemium.app/api/v1/webhooks/registry"
    events: ["credit.issued", "credit.retired", "price.updated"]
    auth: 
      type: hmac
      secret: "${WEBHOOK_SECRET}"
```

### Batch Sync (Scheduled)
```yaml
sync_schedule:
  carbon_credits: "0 */4 * * *"  # Every 4 hours
  recs: "0 1 * * *"              # Daily at 1 AM
  prices: "*/15 * * * *"         # Every 15 minutes
batch_size: 500
incremental: true
```

## Certificate Verification

### Verify Carbon Credit
```http
POST /api/v1/verify/carbon-credit
Content-Type: application/json

{
  "credit_id": "VCS-001-2024-12345",
  "registry": "verra",
  "verification_level": "full"
}
```

### Verification Response
```json
{
  "verification_id": "VERIFY-001",
  "status": "valid",
  "verified_at": "2024-06-14T15:30:00Z",
  "details": {
    "credit_exists": true,
    "current_status": "active",
    "chain_of_custody": "verified",
    "methodology_valid": true,
    "additionality_confirmed": true
  },
  "warnings": [],
  "errors": []
}
```

## Price Feed Integration

### Market Data Sources
- **CBL Markets** - Carbon credit prices
- **EEX** - European carbon markets  
- **ICE** - Intercontinental Exchange
- **Regional REC Markets** - State-specific pricing

### Price Feed Schema
```json
{
  "asset_type": "carbon_credit",
  "registry": "verra",
  "vintage": "2024",
  "price_usd": 15.50,
  "currency": "USD",
  "unit": "tCO2e",
  "timestamp": "2024-06-14T15:45:00Z",
  "volume_24h": 10000,
  "source": "CBL_Markets"
}
```

## Error Codes & Handling

| Code | Description | Retry Strategy |
|------|-------------|----------------|
| `REG_001` | Invalid API credentials | Check credentials |
| `REG_002` | Registry temporarily unavailable | Exponential backoff |
| `REG_003` | Certificate not found | No retry, log warning |
| `REG_004` | Rate limit exceeded | Wait and retry |
| `REG_005` | Data format error | Skip record, alert |

## Registry Mapping Configuration

### Verra Configuration
```yaml
registry: "verra"
connection:
  base_url: "https://registry.verra.org/api"
  auth:
    type: api_key
    key: "${VERRA_API_KEY}"
mapping:
  asset_id: "credit_id"
  quantity: "quantity"
  unit: "unit"
  vintage: "vintage_year"
  status: "status"
  verification_date: "issuance_date"
custom_fields:
  methodology: "methodology"
  additional_certifications: "additional_certifications"
```

### PJM-GATS Configuration
```yaml
registry: "pjm_gats"
connection:
  base_url: "https://gats.pjm.com/api/v2"
  auth:
    type: api_key
    header: "X-API-Key"
    key: "${GATS_API_KEY}"
mapping:
  asset_id: "certificate_id"
  quantity: "mwh"
  unit: "MWh"
  fuel_type: "fuel_type"
  generation_period: "generation_period"
  state: "state"
```

## Sample Integration
```python
# Example Python integration
import requests
from datetime import datetime

class RegistryClient:
    def __init__(self, registry_config):
        self.config = registry_config
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'Bearer {self.config["api_key"]}'
        })
    
    def get_carbon_credits(self, filters=None):
        url = f"{self.config['base_url']}/credits"
        response = self.session.get(url, params=filters)
        response.raise_for_status()
        return response.json()
    
    def verify_certificate(self, certificate_id):
        url = f"{self.config['base_url']}/verify"
        payload = {"certificate_id": certificate_id}
        response = self.session.post(url, json=payload)
        return response.json()
```

## Status
- **Version:** v0.1-draft  
- **Last Updated:** 2025-06-14  
- **Owner:** James  
- **Review Status:** Draft for team review 