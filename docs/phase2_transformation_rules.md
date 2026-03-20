# Phase 2 – Transformation Rules & Data Mapping  
(NovaTech Electronics Sales Data Warehouse)

Source: Kaggle dataset "Electronic Sales Sep2023-Sep2024"  
Goal: Convert raw CSV → star schema (fact + dimensions) with exploded add-ons and SCD Type 2 on loyalty.

## 1. Filtering Rules
- Keep only rows where `Order Status` = 'Completed'  
- Drop all 'Cancelled' rows

## 2. Grain of fact_sales
One row = **one sold item** (either the main product OR one individual add-on)  
→ If a purchase has a main product + 3 add-ons → it becomes 4 rows in fact_sales  
→ This "explosion" allows detailed analysis of add-on revenue, types, etc.

## 3. Column Mapping & Transformations

| Raw Column          | Example                              | Target Table / Column                  | Transformation / Business Rule                                                                                   | Notes |
|---------------------|--------------------------------------|----------------------------------------|----------------------------------------------------------------------------------------------------------|-------|
| Customer ID         | 1000                                 | dim_customer.customer_id               | Natural key for SCD2 grouping                                                                            | Integer → TEXT safe |
| Age                 | 53, 25, 41                           | dim_customer.age_group                 | Bucket into: 'Under 18', '18-24', '25-34', '35-44', '45-54', '55+'                                      | For better grouping & charts |
| Gender              | Male, Female, #N/A                   | dim_customer.gender                    | Map #N/A → 'Unknown'                                                                                     | Clean nulls |
| Loyalty Member      | Yes / No                             | dim_customer.loyalty_status            | SCD Type 2 tracked attribute (new version on change)                                                     | History tracking |
| Purchase Date       | 2024-03-20                           | dim_date + SCD2 valid_from/to          | Parse to date → date_key = YYYYMMDD integer; used for SCD2 timestamps                                    | Core for time & history |
| Product Type        | Smartphone, Laptop, Headphones       | dim_product.product_type               | Direct copy, normalize to title case                                                                     | — |
| SKU                 | SKU1004, SMP234, HDP456              | dim_product.sku                        | Direct copy – unique product identifier                                                                  | Natural key |
| Rating              | 2, 5                                 | fact_sales.rating                      | Direct copy (only on main product rows)                                                                  | Null for add-on rows? |
| Quantity            | 7, 3, 1                              | fact_sales.quantity                    | Direct copy (usually 1 for add-ons)                                                                      | — |
| Unit Price          | 791.19, 247.03                       | fact_sales.unit_price                  | Direct copy (0 or null for add-ons if not priced separately)                                             | — |
| Total Price         | 5538.33, 741.09                      | fact_sales.total_price                 | Main revenue measure for that row                                                                        | — |
| Add-on Total        | 40.21, 0                             | fact_sales.addon_total                 | Direct copy – separate column for add-on revenue                                                         | Your choice – track separately |
| Payment Method      | Credit Card, Paypal, Cash            | dim_payment.payment_method             | Create dim_payment with surrogate key; link via payment_key in fact_sales                               | New dimension |
| Shipping Type       | Standard, Overnight, Express         | fact_sales.shipping_type               | Direct copy as text                                                                                      | Kept simple in fact |
| Add-ons Purchased   | "Accessory,Impulse Item", empty      | —                                      | Parse comma-separated list → explode: one row per add-on (main product + each add-on)                    | Explosion rule |
| Order Status        | Completed / Cancelled                | —                                      | Filtered out if not Completed                                                                            | — |

## 4. SCD Type 2 Logic (dim_customer)
- Sort rows per Customer ID by Purchase Date  
- Detect changes in Loyalty Member (Yes ↔ No)  
- Create new row/version in dim_customer on each change  
- valid_from = Purchase Date of first row with new status  
- valid_to = Purchase Date of next change (or '9999-12-31')  
- is_current = true only for latest version

## 5. Surrogate Keys
- dim_customer.customer_surrogate_key: auto-increment per version  
- dim_product.product_key: auto-increment per unique SKU  
- dim_date.date_key: YYYYMMDD integer  
- dim_payment.payment_key: auto-increment per unique Payment Method  
- fact_sales: no PK needed (or optional sales_line_key)

## 6. Add-ons Explosion Example
Original row:  
Customer buys Smartphone (SKU1004, $791.19) + "Accessory,Extended Warranty" (add-on total $40.21)

After explosion → 3 rows in fact_sales:  
1. Main product: Smartphone, quantity=1, unit_price=791.19, total_price=791.19, addon_total=0  
2. Add-on 1: Accessory, quantity=1, unit_price=null or 0, total_price=part of add-on, addon_total=part  
3. Add-on 2: Extended Warranty, same logic

(Exact split of add-on total can be equal split or kept full on first add-on – decide in code later)

## Next Phase
Phase 3: Implementation in Termux (pandas + duckdb to build tables and explode add-ons)
