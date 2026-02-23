# Sample Inventory Data

This directory contains sample datasets for testing, demonstration, and development purposes.

## Available Datasets

### Product Catalog (`products.csv`)
Sample product master data including:
- SKU identifiers
- Product descriptions  
- Categories and classifications
- Physical attributes (dimensions, weight)
- Cost and pricing information

### Inventory Transactions (`transactions.csv`)  
Historical inventory movement data including:
- Stock receipts and shipments
- Inter-location transfers
- Cycle count adjustments
- Time-stamped transaction records

### Sales Orders (`orders.csv`)
Sample customer order data including:
- Order headers and line items
- Customer information
- Fulfillment status
- Shipping details

### Supplier Data (`suppliers.csv`)
Vendor master information including:
- Supplier profiles
- Contact information
- Performance metrics
- Lead time data

## Data Formats

All sample data follows standardized formats:
- CSV files with UTF-8 encoding
- ISO 8601 date formats
- Consistent field naming conventions
- Proper data type specifications

## Usage Instructions

1. **Development Testing**: Load sample data into local development environments
2. **Demo Scenarios**: Use predefined datasets for demonstration walkthroughs  
3. **Performance Testing**: Scale datasets for load testing scenarios
4. **Training**: Provide realistic data for user training sessions

## Data Privacy

All sample data is synthetic and does not contain:
- Real customer information
- Actual business data
- Sensitive or proprietary information
- Personal identifiable information (PII)

## Loading Sample Data

```bash
# Load into PostgreSQL
psql -d inventory_db -f load_sample_data.sql

# Load via API
curl -X POST http://localhost:8080/api/data/import \
  -H "Content-Type: multipart/form-data" \
  -F "file=@products.csv" \
  -F "type=products"
```