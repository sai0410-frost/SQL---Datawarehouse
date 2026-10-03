# SQL Data Warehouse Project: Medallion Architecture (Bronze / Silver / Gold)

A SQL Server data warehouse that takes sales data from two source systems (CRM and ERP, delivered as CSV files) through three layers: raw, cleaned, and business-ready. The final layer is a star schema for reporting and analytics.

> **Guided project.** I built this by following the *SQL Data Warehouse Project* course by Baraa Khatib Salkini ([Data With Baraa](https://github.com/DataWithBaraa/sql-data-warehouse-project)). The architecture, datasets and original scripts come from that course and are used under its MIT License (see `LICENSE`). The sections [My Work](#my-work) and [Results From My Run](#results-from-my-run) describe what I ran, checked and observed myself.

---

## Architecture

![Data Architecture](docs/data_architecture.png)

| Layer | Purpose | What happens here |
|-------|---------|-------------------|
| **Bronze** | Raw landing zone | Six CSV files (3 CRM, 3 ERP) are bulk-loaded into SQL Server as-is, with no changes |
| **Silver** | Cleaned and standardized | Duplicates removed, coded values converted to readable labels, missing values handled |
| **Gold** | Business-ready | Views that join CRM and ERP data into a star schema |

## Data Sources

- **CRM:** customer info, product info, sales details
- **ERP:** customer demographics (`CUST_AZ12`), customer locations (`LOC_A101`), product categories (`PX_CAT_G1V2`)

Scope is the latest snapshot only, so no historization.

## Data Model (Gold Layer)

![Data Model](docs/data_model.png)

- `gold.fact_sales`: sales transactions (order number, dates, amount, quantity, price)
- `gold.dim_customers`: customer attributes combined from CRM and ERP
- `gold.dim_products`: product attributes combined from CRM and ERP

## Results From My Run

I ran the full pipeline on my own machine (SQL Server Express + SSMS). Row counts and checks:

| Table | Bronze | Silver | Change |
|-------|-------:|-------:|--------|
| `crm_cust_info` | 18,493 | 18,484 | 9 rows removed |
| `crm_prd_info` | 397 | 397 | none removed |
| `crm_sales_details` | 60,398 | 60,398 | none removed |

- **Duplicates:** 6 customer IDs appeared more than once in bronze. Silver has none.
- **Standardization:** gender in bronze was stored as codes (`F`, `M`, `NULL`). In silver it reads `Female`, `Male` and `n/a`.
- **Quality checks:** the gold checks (duplicate customer keys, duplicate product keys, sales rows with no matching customer or product) all returned zero rows. The silver checks I reviewed returned no problem rows, and marital status read `Single` / `Married`.

## My Work

- Set up SQL Server Express and SSMS and ran the complete pipeline end to end: database setup, bronze, silver, gold
- Updated the file paths in the bronze load script for my machine and loaded all six CSV files
- Compared bronze against silver (row counts, duplicate IDs, standardized values) to confirm the cleaning worked
- Ran the quality-check scripts for silver and gold
- Observed that joining the gold views to each other was slow on SQL Server Express (see below)

### Possible improvements

- **Performance:** the gold layer is made of views, and joining them was slow on my Express instance. Persisting gold as indexed tables would avoid recomputing the views on every query.
- **Analytics:** the warehouse is built, but I haven't yet written reports on customer behavior, product performance or sales trends on top of the gold layer.
- **Automation:** the load scripts are run manually, in order. They could be scheduled or orchestrated.

## Tech Stack

SQL Server Express, SQL Server Management Studio (SSMS), T-SQL, Git / GitHub

## Repository Structure

```
├── datasets/
│   ├── source_crm/        # cust_info, prd_info, sales_details (CSV)
│   └── source_erp/        # CUST_AZ12, LOC_A101, PX_CAT_G1V2 (CSV)
├── docs/                  # Architecture, data flow, data model and ETL diagrams, data catalog, naming conventions
├── scripts/
│   ├── init_database.sql  # Creates the DataWarehouse database and bronze/silver/gold schemas
│   ├── bronze/            # ddl_bronze.sql, proc_load_bronze.sql
│   ├── silver/            # ddl_silver.sql, proc_load_silver.sql
│   └── gold/              # ddl_gold.sql (star schema views)
├── tests/                 # quality_checks_silver.sql, quality_checks_gold.sql
├── my_notes.md            # What I observed while running the project
├── LICENSE
└── README.md
```

## How to Run

1. Install SQL Server Express and SSMS, and connect to your instance (for example `localhost\SQLEXPRESS`).
2. Run `scripts/init_database.sql`. Then set the database dropdown to `DataWarehouse`.
3. **Bronze:** run `scripts/bronze/ddl_bronze.sql`. In `proc_load_bronze.sql`, change every file path in the `BULK INSERT` statements to the location of the `datasets` folder on your machine, then run the script and execute `EXEC bronze.load_bronze;`
4. **Silver:** run `scripts/silver/ddl_silver.sql` and `scripts/silver/proc_load_silver.sql`, then `EXEC silver.load_silver;`
5. **Gold:** run `scripts/gold/ddl_gold.sql` to create the views.
6. Run the checks in `tests/` one query at a time. Queries marked *Expectation: No Results* should return nothing.

## Documentation

- `docs/data_catalog.md`: field descriptions for the gold layer
- `docs/naming_conventions.md`: naming rules for tables, columns and files
- `docs/`: architecture, data flow, data integration, data model and ETL diagrams

## Acknowledgments & License

Based on the SQL Data Warehouse Project course by Baraa Khatib Salkini ([Data With Baraa](https://github.com/DataWithBaraa/sql-data-warehouse-project)), released under the MIT License. The original license is included in `LICENSE`.

## About Me

Hi, I'm **Saidas Subhadarshi**, [one line about your background and the role you're aiming for].

- LinkedIn: [saidas-subhadarshi](https://www.linkedin.com/in/saidas-subhadarshi-46553b308)
- GitHub: [sai0410-frost](https://github.com/sai0410-frost)
