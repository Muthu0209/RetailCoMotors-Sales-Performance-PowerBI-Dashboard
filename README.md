# RetailCoMotors-Sales-Performance-PowerBI-Dashboard
Power BI case study analysing RetailCoMotors’ online and reseller sales: data modeling, DAX (CAGR, YTD, YoY), Row-Level Security and interactive dashboards.

**End-to-end Power BI case study: data modeling, DAX measures, Row-Level Security and sales insights across Online and Reseller channels.**


## Table of Contents
1. [Business Context](#-business-context)
2. [Objectives](#-objectives)
3. [Dataset](#-dataset)
4. [Tools & Skills](#-tools--skills)
5. [Approach](#-approach)
6. [Data Model](#-data-model)
7. [DAX Measures](#-dax-measures)
8. [Dashboard Preview](#-dashboard-preview)
9. [Key Insights](#-key-insights)
10. [Row-Level Security](#-row-level-security-rls)
11. [Repository Structure](#-repository-structure)
12. [How to Run](#-how-to-run)
13. [Author](#-author)

---

## Business Context

RetailCoMotors is a retail company that sells a wide range of products through two channels: its **online store** and a **network of resellers**. As a Senior Data Analyst, the task was to analyse the company's sales data to help the CEO and board of directors make decisions on **profitability, market trends and product mix**.

## Objectives

- Build a clean, relational data model from 7 source tables.
- Create DAX measures for customers, resellers, sales, growth and time intelligence.
- Design an interactive, two-page dashboard that answers key business questions.
- Identify the best-performing regions, categories and models.
- Restrict reseller data to **California** using Row-Level Security.

##  Dataset

Source file: `ShoppersSale.xlsx` (7 tables, about 121K order lines)

| Table | Description | Rows |
|---|---|---|
| **Sales** | Fact table: quantities, unit price, cost and sales amount per order line | 121,253 |
| **OrderType** | Sales channel (Online / Reselling) and order identifiers | 121,253 |
| **Customer** | Customer name, city, state, region | 18,485 |
| **Reseller** | Reseller name, business type, location | 702 |
| **Product** | SKU, model, subcategory, category, cost, list price | 397 |
| **Date** | Calendar and fiscal year / quarter / month | 1,280 |
| **SalesArea** | Region, country and group | 11 |

> Row counts exclude the header row. The Customer and Reseller tables each include one placeholder `NA` row for orders with no customer or reseller.

## Tools & Skills

- **Power BI Desktop**: report design, data modeling, RLS
- **Power Query**: data cleaning and transformation
- **DAX**: calculated measures and time intelligence
- Data modeling (star schema), KPI design, data storytelling

##  Approach

### 1. Data preparation (Power Query)
- Converted `DateNode` and the order, due and ship date keys from Text to Date.
- Removed duplicates from the Date table.
- Merged **Sales** with **Customer** and **Reseller** to bring in city, state and region.
- Replaced `"NA"` values with blanks.
- Created combined location columns (state, city, region) from the customer and reseller attributes.

### 2. Data modeling
Star schema with **Sales** as the fact table:

| From (many) | To (one) | Key | Status |
|---|---|---|---|
| Sales | Date | `OrderDateNode` → `DateNode` | Active |
| Sales | Product | `ProductNode` | Active |
| Sales | Reseller | `ResellNode` | Active |
| Sales | Customer | `CustomerNode` | Active |
| Sales | OrderType | `OrderLine` (1:1) | Active |
| Sales | SalesArea | `SalesArea` | Inactive |
| Reseller | SalesArea | `Region` | Active |
| Customer | SalesArea | `Region` | Inactive |

Only one active path is kept between tables to avoid ambiguity. The other relationships are set to inactive.

### 3. DAX measures and visuals
Measures were created for each KPI and used in the visuals listed below.

## DAX Measures

DAX
-- Distinct counts
UniqueCustomerCount = DISTINCTCOUNT ( Customer[Customer ID] )
UniqueResellerCount = DISTINCTCOUNT ( Reseller[Resell ID] )

-- Total sales
TotalSales = SUM ( Sales[Sales Amount] )

-- Compound Annual Growth Rate, FY2019 to FY2021
CAGR_Sales =
VAR StartYear  = "FY2019"
VAR EndYear    = "FY2021"
VAR YearsCount = 2
VAR StartSales = CALCULATE ( [TotalSales], 'Date'[Fiscal Year] = StartYear )
VAR EndSales   = CALCULATE ( [TotalSales], 'Date'[Fiscal Year] = EndYear )
RETURN
    IF ( StartSales > 0, ( EndSales / StartSales ) ^ ( 1 / YearsCount ) - 1, BLANK () )

-- Year-to-date sales
YTD_Sales = TOTALYTD ( SUM ( Sales[Sales Amount] ), 'Date'[DateNode] )

-- Average sales per quarter
AverageSalesPerQuarter =
CALCULATE (
    AVERAGEX (
        SUMMARIZE ( 'Date', 'Date'[Fiscal Year], 'Date'[Fiscal Quarter], "QuarterlySales", [TotalSales] ),
        [QuarterlySales]
    )
)

-- Year-over-year
SalesPreviousYear   = CALCULATE ( [TotalSales], SAMEPERIODLASTYEAR ( 'Date'[DateNode] ) )
YoY_SalesDifference = [TotalSales] - [SalesPreviousYear]
YoY_SalesGrowth     = DIVIDE ( [YoY_SalesDifference], [SalesPreviousYear], 0 )
```

## Dashboard Preview

### Page 1: Year Wise (sales trends)
KPI cards, YTD sales by year and channel, average sales per quarter, total sales by region, and total sales by year and quarter.

![Year Wise page](images/dashboard-year-wise.jpg)

### Page 2: Category (product performance)
Product count by category, total sales by model, total sales by category, and a combo chart of product count by year and category with a YTD sales trend line.

![Category page](images/dashboard-category.jpg)

##  Key Insights

| Metric | Result |
|---|---|
| **Total Sales** | **$113.08M** |
| **Unique Customers** | **~18.5K** |
| **Unique Resellers** | **702** |
| **CAGR (FY2019 to FY2021)** | **49.07%** |

- **Components has the largest product count (189 products)**, followed by Bikes (125), Clothing (48) and Accessories (35).
- **Bikes generate the most revenue: about $96M**, far ahead of Components ($12M), Clothing ($3M) and Accessories ($2M). Breadth of range does not translate into revenue; Bikes are the core business.
- **Top 5 models by sales:** Mountain-200, Road-250, Mountain-100, Touring-1000, Road-650.
- **United States is the largest market**, followed by Canada, Australia, the United Kingdom, France and Germany.
- Sales grew strongly between FY2019 and FY2021, a **49.07% compound annual growth rate**.

##  Row-Level Security (RLS)

A **California** role was created in *Modeling → Manage roles* with this filter on the **Reseller** table:

```DAX
[State] = "California"
```

Testing with *View as → California* limits the report to California resellers, and the numbers change accordingly:

| | Total Sales | CAGR |
|---|---|---|
| All data | $113.08M | 49.07% |
| California role | $10.10M | 24.04% |

(images/rls-california-view.jpg)

##  Repository Structure

```
retailcomotors-sales-powerbi/
├── README.md
├── RetailCoMotors-Sales-Performance-PowerBI Dashboard.pbix              # Power BI report
├── ShoppersSale.xlsx      # Source dataset
└── images/
    ├── dashboard-year-wise.jpg
    ├── dashboard-category.jpg
    └── rls-california-view.jpg
```

## How to Run

1. Clone the repo: `git clone https://github.com/<your-username>/retailcomotors-sales-powerbi.git`
2. Open `MLS3.pbix` in **Power BI Desktop**.
3. If prompted, update the data source: *Home → Transform data → Data source settings* and point to your local `ShoppersSale.xlsx`.
4. Click **Refresh**.
5. To test RLS, go to *Modeling → View as* and select **California**.

## Author

**Your Name**
[LinkedIn](https://www.linkedin.com/in/muthu-viswa/) · [GitHub](https://github.com/Muthu0209))

---
*This project was completed as part of a data analytics course - PowerBI case study.*
