# My Notes: Running the Data Warehouse Project

Notes from running the SQL Data Warehouse course project on my own machine (SQL Server Express + SSMS), October 2026.

## What I did

1. Created the `DataWarehouse` database and the bronze / silver / gold schemas (`init_database.sql`)
2. Created the bronze tables, edited the six `BULK INSERT` file paths for my machine, and ran `EXEC bronze.load_bronze;`
3. Created the silver tables and ran `EXEC silver.load_silver;`
4. Created the gold views (`dim_customers`, `dim_products`, `fact_sales`)
5. Ran the quality checks in `tests/` for silver and gold

## What I observed

**Row counts, bronze to silver**

| Table | Bronze | Silver |
|-------|-------:|-------:|
| `crm_cust_info` | 18,493 | 18,484 |
| `crm_prd_info` | 397 | 397 |
| `crm_sales_details` | 60,398 | 60,398 |

- Customers lost 9 rows in silver. 6 customer IDs appeared more than once in bronze, and silver has no duplicates.
- Gender in bronze was `F`, `M` and `NULL`. In silver it became `Female`, `Male` and `n/a`.
- Marital status in silver reads `Single` / `Married`.
- All three gold checks returned zero rows: no duplicate customer keys, no duplicate product keys, and no sales row without a matching customer or product.

**Performance**

Selecting from each gold view on its own was fast. A query joining `fact_sales` to both dimensions ran for several minutes before returning on my Express instance. My guess is that the views recompute their keys on every query, but I still need to confirm that by reading `ddl_gold.sql`. A possible fix is storing gold as indexed tables.

## Still to confirm by reading the scripts

I want to be able to explain these in my own words, so I'll fill them in after reading the code:

- [ ] Which record `proc_load_silver.sql` keeps when a customer ID is duplicated (and why)
- [ ] What happens to customers with a `NULL` ID
- [ ] What cleaning silver applies to `crm_sales_details` (dates, sales amount) given that no rows were removed
- [ ] How `ddl_gold.sql` creates the surrogate keys
- [ ] Whether `proc_load_bronze.sql` truncates tables before loading

To see where the 9 removed customer rows came from, run:

```sql
-- Rows with a NULL customer ID in bronze
SELECT COUNT(*) AS null_id_rows FROM bronze.crm_cust_info WHERE cst_id IS NULL;

-- Extra rows from duplicated (non-null) IDs
SELECT SUM(cnt - 1) AS extra_duplicate_rows FROM (
  SELECT cst_id, COUNT(*) AS cnt
  FROM bronze.crm_cust_info
  WHERE cst_id IS NOT NULL
  GROUP BY cst_id HAVING COUNT(*) > 1) d;
```

If the two results add up to 9, that fully explains the difference.

## Problems I hit and how I handled them

- **File paths:** the bronze load script contained the course author's file paths. I replaced all six with my own.
- **Slow gold join:** described above.
- **Unsaved changes:** edited scripts and the queries I typed in new windows need to be saved to the project folder, or they are lost.
