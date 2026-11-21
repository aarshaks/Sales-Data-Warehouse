Sales Analytics Dashboard (Power BI)

An interactive Power BI report that analyses sales performance across products, stores, regions and customers, built on a star-schema data model with DAX measures, time intelligence and a custom dark-blue design.



 Table of contents

1. [Project overview](#project-overview)
2. [Tools and skills](#tools-and-skills)
3. [Data source](#data-source)
4. [Data model](#data-model)
5. [DAX measures](#dax-measures)
6. [Report pages](#report-pages)
7. [Design](#design)
8. [Key insights](#key-insights)
9. [Data notes and limitations](#data-notes-and-limitations)
10. [How to open the report](#how-to-open-the-report)
11. [Repository structure](#repository-structure)
12. [Author](#author)



Project overview

The goal of this project is to turn raw sales records into a clear, interactive report that answers practical business questions:

- How much are we selling, and how is it trending over time?
- Which products, categories and brands drive sales?
- Which regions and stores perform best, and why?
- Who are our customers, and how much does each one spend?

The report has 4 pages, synced slicers, drill-down and rank measures that respond to the user's selections.

Tools and skills

| Area | Used |
|---|---|
| Tool | Power BI Desktop |
| Data source | MySQL database (`sales_warehouse`) |
| Modelling | Star schema, one-to-many relationships, a marked date table, a dedicated measures table |
| DAX | `CALCULATE`, `SAMEPERIODLASTYEAR`, `DATEADD`, `RANKX`, `ALL`, `ALLSELECTED`, `REMOVEFILTERS`, `HASONEVALUE`, `DISTINCTCOUNT`, `DIVIDE`, variables (`VAR` / `RETURN`) |
| Report design | Custom theme, KPI cards, donut charts, heatmap matrix, scatter chart, drill-down, conditional formatting, page-navigation buttons, slicer sync |

 Data source

Data comes from a MySQL database named `sales_warehouse`, loaded into Power BI in import mode.

| Table | Rows | Contents |
|---|---|---|
| `fact_sales` | 5,000 | One row per sale: keys to date, customer, product and store; `quantity_sold`, `unit_price`, `discount`, `total_amount` |
| `dim_date` | 730 | One row per day, 14 Oct 2023 to 12 Oct 2025: year, quarter, month, month name, weekday, `is_weekend` |
| `dim_product` | 500 | `product_name`, `category`, `brand`, `unit_price`, `launch_date` |
| `dim_customer` | 500 | `customer_id`, names, `gender`, contact details, `country`, `state`, `city`, `join_date` |
| `dim_store` | 50 | `store_name`, `region`, `country`, `city` |

The sales value follows this rule in every row: `total_amount = quantity_sold x unit_price - discount`, so `discount` is a money amount, not a percentage.

 Data model

A star schema with one fact table and four dimensions. Each relationship is one-to-many, from the dimension to the fact table, with single-direction filtering.


dim_date ─────┐
dim_product ──┤
dim_customer ─┼──< fact_sales
dim_store ────┘


Model details:

- `dim_date[date]` is marked as the **date table**, so time intelligence works correctly.
- A calculated column `full name` in `dim_customer` joins first and last name.
- All measures live in a separate `calculation table`.
- `month_name` is sorted by the `month` column, so months appear in calendar order.

 DAX measures

Core measures:

| Measure | Purpose |
|---|---|
| `Total Sales` | Net revenue: sum of `total_amount` |
| `Total Quantity` | Units sold |
| `Total Order` | Distinct sales records |
| `Avg Order value` | Total sales divided by total orders |
| `Total Customers` | Customers who actually bought in the current filter |
| `Active store` / `Sales per Store` | Stores with sales, and sales per store |
| `Total Discount` / `Discount %` | Discount given, and as a share of gross revenue |
| `% of Total Sales` | A row's share of the selected total |

Time intelligence:

dax
total_sales ly =
CALCULATE([Total Sales], SAMEPERIODLASTYEAR('sales_warehouse dim_date'[date]))


dax
Sales MoM % =
VAR CurrentSales = [Total Sales]
VAR PriorMonth = CALCULATE([Total Sales], DATEADD('sales_warehouse dim_date'[date], -1, MONTH))
RETURN
IF(NOT ISBLANK(PriorMonth), DIVIDE(CurrentSales - PriorMonth, PriorMonth))


Rank measures (respond to slicers):

dax
Store Rank =
RANKX(ALL('sales_warehouse dim_store'[store_name]), [Total Sales], , DESC)


dax
Customer Rank =
RANKX(ALL('sales_warehouse dim_customer'[full name]), [Total Sales], , DESC)


`Product Rank` follows the same pattern over `product_name`.

 Report pages

The report has a left-hand navigation sidebar with four page buttons and a Year slicer that is synced across all pages.

 1. Sales Overview
- Cards: Total Sales, Total Quantity, Total Orders, Avg Order Value (and an estimated profit card, if enabled).
- Line chart of total sales by month, compared with last year.
- Bar charts by category, top products and region; weekday vs weekend sales.
- Slicers: Year, Region, Category.

 2. Product Performance
- Cards: Products Sold, Avg Selling Price, Total Discount, Discount %, Sales per Product.
- Donut charts: sales share by category and by brand.
- Heatmap matrix: sales by category and brand.
- Top 10 products table with rank and share-of-sales data bars.
- Slicers: Year, Category, Brand, Product.

 3. Store and Region
- Cards: Total Sales, Active Stores, Sales per Store, Orders per Store, Avg Order Value.
- Bar chart of sales by region, with drill-down from region to store.
- Column chart of sales per store by region.
- Scatter chart of orders against average order value for every store.
- Top stores table with rank, data bars and a colour scale.
- Slicers: Year, Region, Category.

 4. Customers
- Cards: Total Customers, Sales per Customer, Orders per Customer, Units per Order.
- Sales by country, state and gender; top customers table.
- Slicers: Year, Gender, State.

 Design

- Canvas: 1280 x 720.
- Navigation: page-navigation buttons in a left sidebar.

 Key insights

All figures are for the full data range with no filters applied.

| Measure | Value |
|---|---|
| Total sales | 13,957,454 |
| Total quantity | 27,719 units |
| Total orders | 5,000 |
| Avg order value | about 2,791.5 |
| Customers / products / stores | 500 / 500 / 50 |
| Total discount | 124,550 (about 0.88% of gross revenue) |

- Categories are evenly balanced. Sports leads at about 2.51M (18.0% of sales) and Electronics is lowest at about 2.03M (14.6%).
- BrandA is the top brand at about 3.08M (22.0%).
- Central has the highest total sales (3.64M) because it has the most stores (13). Per store, West leads at about 287K, and all five regions are within about 7% of each other.
- No single product dominates. The top product, Eight Federal, makes up only about 0.5% of sales.
- Discounts are small, under 1% of gross revenue.
- Every customer is a repeat buyer, at about 10 orders each.

Data notes and limitations

- Partial years.The data starts on 14 Oct 2023 and ends on 12 Oct 2025, so 2023 and 2025 are not full years. Year-over-year and month-over-month figures for October 2023, November 2023 and October 2025 should be read with care.
- No cost data.The model has no product cost, so profit is not part of the core analysis. Any profit figure would be an estimate based on an assumption.
-Duplicate product name. There are 500 products but 499 distinct product names, so one name appears under two brands.
- Location data is synthetic. City names are not real places and countries are assigned to stores at random, so a map by city does not work and country does not match region. Region is used for geographic analysis instead.

How to open the report

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/).
2. Download the `.pbix` file from this repository.
3. Open it in Power BI Desktop.


 Repository structure


.
├── README.md
├── powerbi project 2.pbix      # the Power BI report
└── docs/
    └── images/                  # screenshots of each report page
        ├── overview.png
        ├── product.png
        ├── store.png
        └── customers.png


 Author

AARSHA K S
aarshaks112@gmail.com
https://www.linkedin.com/in/aarsha-ks

Built as a portfolio project to practise data modelling, DAX and report design in Power BI.
