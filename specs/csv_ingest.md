# CSV Ingest Specification

## Purpose & Scope
Define the format and processing requirements for CSV file ingestion of commodity inventory and ESG metadata.

**In Scope:**
- File format validation
- Column mapping and normalization
- Data type validation and coercion
- Error handling and reporting
- Batch processing capabilities

**Out of Scope:**
- Real-time streaming ingestion
- Binary file formats
- Direct database imports

## File Format Requirements

### Supported Formats
- UTF-8 encoded CSV files
- Maximum file size: 100MB
- Maximum rows: 500,000 per file

### Required Columns
| Column Name | Type | Required | Description | Example |
|-------------|------|----------|-------------|---------|
| `asset_id` | String | Yes | Unique asset identifier | `CORN-US-2024-001` |
| `asset_type` | String | Yes | Asset category | `commodity`, `carbon_credit`, `rec` |
| `quantity` | Decimal | Yes | Asset quantity | `1000.50` |
| `unit` | String | Yes | Measurement unit | `tonnes`, `mwh`, `credits` |
| `acquisition_date` | Date | Yes | When asset was acquired | `2024-06-01` |
| `origin_country` | String | No | Country of origin | `US`, `BR`, `AU` |
| `emission_factor` | Decimal | No | CO2e per unit | `2.45` |

### Optional ESG Metadata Columns
- `certification_standard` (String): e.g., "FSC", "Fairtrade"
- `traceability_id` (String): Supply chain tracking ID
- `sustainability_score` (Integer): 1-100 rating
- `deforestation_risk` (String): "Low", "Medium", "High"

## API Specification

### Upload Endpoint
```
POST /api/v1/ingest/csv
Content-Type: multipart/form-data

Parameters:
- file: CSV file (required)
- mapping_profile: String (optional) - predefined column mapping
- validate_only: Boolean (optional) - dry-run validation
```

### Sample Request
```bash
curl -X POST "http://localhost:8080/api/v1/ingest/csv" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@inventory.csv" \
  -F "mapping_profile=agriculture_v1"
```

### Response Format
```json
{
  "upload_id": "uuid-v4",
  "status": "processing|completed|failed",
  "total_rows": 1500,
  "processed_rows": 1500,
  "failed_rows": 0,
  "errors": [],
  "validation_summary": {
    "schema_valid": true,
    "data_quality_score": 95
  }
}
```

## Error Codes
| Code | Description | Action |
|------|-------------|--------|
| `CSV_001` | Invalid file format | Use UTF-8 CSV |
| `CSV_002` | Missing required column | Add column to file |
| `CSV_003` | Invalid data type | Fix data format |
| `CSV_004` | File too large | Split into smaller files |

## Retry Strategy
- Failed rows are logged with specific error reasons
- Partial success supported - valid rows are processed
- Retry endpoint available for failed rows only

## Sample Payload
```csv
asset_id,asset_type,quantity,unit,acquisition_date,origin_country,emission_factor
CORN-US-2024-001,commodity,1000.5,tonnes,2024-06-01,US,2.45
CARBON-VER-001,carbon_credit,100,credits,2024-05-15,BR,
REC-WIND-TX-001,rec,500,mwh,2024-06-10,US,0.02
```

## Status
- **Version:** v0.1-draft  
- **Last Updated:** 2025-06-14  
- **Owner:** James  
- **Review Status:** Draft for team review 