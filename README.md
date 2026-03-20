# NovaTech Electronics Sales Data Warehouse

End-to-end production-grade sales analytics DW using cloud-native tools.

## Phase 1 – Data Modeling

### Business Grain
One row = one completed product sold (quantity can be >1)

### Star Schema Design
- **fact_sales** → core measures + foreign keys
- **dim_customer** → SCD Type 2 (loyalty_member changes over time)
- **dim_product** → static product info
- **dim_date** → time intelligence

<image-card alt="Star Schema Diagram" src="diagrams/star_schema.png" ></image-card>


Key features:
- Handles loyalty changes with SCD Type 2
- Explodes add-ons purchased in later ETL
- Filters to completed orders only
