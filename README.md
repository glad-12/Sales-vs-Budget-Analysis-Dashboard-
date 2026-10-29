
# 📊 Sales vs Budget Performance Analysis (2019 – 2021)

**Power BI | Power Query (ETL) | DAX | Star Schema Data Modeling**

A real-world business intelligence project that combines **actual sales**, **monthly salesperson budgets** and a **product master** into one clean star-schema model, then answers one key question for management: *are we selling above or below budget, and what is driving the result?*

---

##  Table of Contents

1. [Project Overview](#-project-overview)
2. [Business Questions](#-business-questions)
3. [Datasets](#-datasets)
4. [Tools Used](#-tools-used)
5. [Data Cleaning and ETL (Power Query)](#-data-cleaning-and-etl-power-query)
6. [Data Model (Star Schema)](#-data-model-star-schema)
7. [DAX Measures](#-dax-measures)
8. [Dashboard Walkthrough](#-dashboard-walkthrough)
9. [Key Insights](#-key-insights)
10. [Repository Structure](#-repository-structure)
11. [Author](#-author)

---

## Project Overview

A consumer-goods distribution company sells food and beverage products through **Retail**, **Distributor** and **Online** channels. Headline revenue alone does not show whether the sales team is hitting expectations, so management needed:

- Actual revenue compared with budget
- An explanation of the drivers behind the variance
- Evidence on which salespeople, channels and products matter most

**Reporting period:** January 2019 to 14 March 2021 (2021 is a partial year, so it contains only 3 months of data).

---

## Business Questions

- Is the company performing above or below budget, and how has this changed over time?
- Which salespeople deliver against budget, and where are the biggest revenue gaps?
- Which sales channels drive revenue, and how has their contribution changed?
- Which product categories and product groups generate the most revenue?
- How has revenue changed year over year?

---

## Datasets

This is a real-world dataset made up of three Excel files.

| File | What it contains | Used for |
|---|---|---|
| **SalesData.xlsx** | About 260,000 sales-line records (around 53K orders): Order Date, Order Number, Product Key, Salesperson Key, Salesperson, Supervisor, Manager, Channel, Quantity, Unit Price | Revenue, quantity, orders, channel and salesperson analysis |
| **Budget.xlsx** | Monthly sales budget per salesperson, with one sheet each for 2019, 2020 and 2021 | Budget vs actual, variance and attainment |
| **Product.xlsx** | 798 products: ID, Product Name, Product Group, Product Category | Product, group and category analysis |

---

##  Tools Used

| Tool | Purpose |
|---|---|
| **Power BI Desktop** | Data modeling, DAX and dashboards |
| **Power Query (M)** | ETL: cleaning, reshaping and combining data |
| **DAX** | Date table, calculated columns and KPI measures |
| **Excel** | Source files |

---

## Data Cleaning and ETL (Power Query)

### 1. Budget data (the hardest to clean)

The budget file was **not in a table format**. Each year sat on its own sheet, with a title row, a year header, salespeople in rows and months 1 to 12 spread across columns, plus a total column. I used **Power Query ETL** to fix this:

- Removed the title rows and promoted the correct header row
- Removed the `Total` column and total rows so totals are never double counted
- **Unpivoted** the month columns so each row holds one salesperson, one month and one budget value (a proper table format)
- Added an extra **Year** column with a **custom column**, because the year was only written in the sheet header
- Combined the 2019, 2020 and 2021 sheets into one budget table
- Set the correct data types

### 2. Sales and Product data

- Imported both files and removed the report title and extraction rows at the top of the sales sheet
- Promoted headers and set correct data types (dates, whole numbers, decimals, text)
- Checked keys (`ProductKey`, `SalespersonKey`) so they match the dimension tables
- Cleaned text columns and removed blanks and duplicates where needed

### 3. Date tables (DAX)

- Created a new **DimDate** table with a DAX date function
- Added a **Budget Date** column to the budget table with a DAX date function (built from Year and Month), so the budget can connect to the date table

---

##  Data Model (Star Schema)

<img width="1048" height="554" alt="image" src="https://github.com/user-attachments/assets/2f196530-b8bf-4349-bf43-b4c3da0bbd57" />



### Tables in the model

| Table | Type | What it contains |
|---|---|---|
| **Fact_Sales** | Fact | Every sales transaction: Order Date, Order Number, Product Key, Salesperson Key, Quantity and Unit Price. Revenue is calculated as Quantity × Unit Price. |
| **BudgetFact** | Fact | The cleaned monthly budget per salesperson: Salesperson, Year, Month, Budget and the added Budget Date. |
| **DimProduct** | Dimension | Product ID, Product Name, Product Group and Product Category (Food or Drink). |
| **DimSalesPerson** | Dimension | Salesperson, Supervisor, Manager and Channel (Retail, Distributor or Online). |
| **DimDate** | Dimension | A date table with Date, Year, Month and Month Number, used for trends and year-over-year analysis. |

### Relationships

| From (one side) | To (many side) | Why |
|---|---|---|
| DimProduct | Fact_Sales | Analyze sales by product, group and category |
| DimDate | Fact_Sales | Analyze sales over time |
| DimSalesPerson | Fact_Sales | Analyze sales by salesperson, manager and channel |
| DimDate | BudgetFact | Compare budget with revenue by month and year |
| DimSalesPerson | BudgetFact | Compare budget with revenue by salesperson |

> The budget is **not** joined directly to the sales transactions. It connects through the shared dimensions (DimDate and DimSalesPerson), so budget values are never duplicated.

---

## DAX Measures

```DAX
-- Date table
DimDate = CALENDAR ( DATE ( 2019, 1, 1 ), DATE ( 2021, 12, 31 ) )

-- Budget date (calculated column in BudgetFact)
Budget Date = DATE ( BudgetFact[Year], BudgetFact[Month], 1 )

-- Core measures
Total Revenue      = SUMX ( Fact_Sales, Fact_Sales[Quantity] * Fact_Sales[UnitPrice] )
Total Quantity     = SUM ( Fact_Sales[Quantity] )
Total Orders       = DISTINCTCOUNT ( Fact_Sales[OrderNumber] )
Avg Order Value    = DIVIDE ( [Total Revenue], [Total Orders] )
Total Salesperson  = DISTINCTCOUNT ( Fact_Sales[SalespersonKey] )

-- Budget measures
Total Budget        = SUM ( BudgetFact[Budget] )
Budget Variance     = [Total Revenue] - [Total Budget]
Budget Attainment % = DIVIDE ( [Total Revenue], [Total Budget] )

-- Growth
YoY Growth % =
VAR PrevRevenue = CALCULATE ( [Total Revenue], SAMEPERIODLASTYEAR ( DimDate[Date] ) )
RETURN DIVIDE ( [Total Revenue] - PrevRevenue, PrevRevenue )
```

| Measure | Meaning |
|---|---|
| Total Revenue | Quantity × Unit Price |
| Total Quantity | Total units sold |
| Total Orders | Distinct order count |
| Avg Order Value | Revenue ÷ Orders |
| Total Budget | Budget at the selected level (year, month, salesperson) |
| Budget Variance | Actual revenue − Budget |
| Budget Attainment % | Actual revenue ÷ Budget |
| YoY Growth % | Change compared with the prior year |

---

## Dashboard Walkthrough

### Page 1: Revenue vs Budget Performance (2019 to 2021)
<img width="989" height="546" alt="image" src="https://github.com/user-attachments/assets/01112bda-d931-4315-bf37-2fbe50f3713a" />

| KPI | Value |
|---|---|
| Total Revenue | 17.91M |
| Total Budget | 17.27M |
| Budget Variance | 641.92K |
| Budget Attainment % | 103.7% |

| Year | Total Budget | Total Revenue | Budget Attainment % |
|---|---:|---:|---:|
| 2019 | 6,764,182.71 | 6,648,222.05 | 98.3% |
| 2020 | 9,352,029.32 | 10,214,809.43 | 109.2% |
| 2021 (partial) | 1,151,096.83 | 1,046,201.01 | 90.9% |
| **Total** | **17,267,308.86** | **17,909,232.49** | **103.7%** |

Includes a year slicer, a monthly Budget vs Revenue line chart, a yearly ribbon chart and a key insights panel.

### Page 2: Product, Channel and Salesperson Revenue Dashboard
<img width="994" height="564" alt="image" src="https://github.com/user-attachments/assets/96c442fb-63a7-49f7-ba39-1ce30a573fac" />


| KPI | Value |
|---|---|
| Total Quantity | 6M |
| Avg Order Value | ₦340.73 |
| Total Salesperson | 12 |
| Total Orders | 53K |
| YoY Growth % | 6.20% |

- **Revenue by Channel:** Retail 8.70M, Distributor 6.10M, Online 3.11M
- **Avg Order Value by Channel:** Retail ₦390.56, Distributor ₦333.85, Online ₦258.93
- **Revenue by Product Category:** Food 16.36M (91%), Drink 1.54M (9%)
- **Top Product Groups by Revenue:** Wheat Flour (4.47M), Oil (2.89M), Yeasts (2.20M), Flour (1.66M), Liquor (1.14M)
- **Top Salespeople by Revenue:** Carla Ferreira (4.71M), Julio Lima (3.30M), Gustavo Gomes (2.45M), Felipe Goncalves (1.68M), Leonardo Cardoso (1.61M)

### Page 3: Salesperson Analysis (Business Drivers)

<img width="998" height="596" alt="image" src="https://github.com/user-attachments/assets/5e0e4d38-f8b3-4229-b00e-ec351dad25f8" />

- Filters for **Channel**, **Year** and **Product Group**
- **Top Salesperson:** Carla Ferreira with 4.71M revenue
- **Top salesperson per channel:**

| Channel | Top Salesperson | Revenue |
|---|---|---:|
| Retail | Carla Ferreira | 4,707,402.55 |
| Distributor | Julio Lima | 3,301,482.32 |
| Online | Felipe Goncalves | 1,676,337.10 |

- Revenue by Salesperson and Channel chart, and a table of the top salesperson in each product group

---

## Key Insights

1. **Overall:** Revenue of 17.91M beat the 17.27M budget by 641.9K (103.7% attainment).
2. **2020 drove the result:** 109.2% attainment, about 862.8K above budget.
3. **2019 and 2021 fell short:** 2019 reached 98.3% and 2021 reached 90.9%, missing budget by about 116K and 105K.
4. **2021 is a partial year:** It covers only 3 months, and its budget is just 1.15M, so the 90.9% reflects that short period.
5. **Channels:** Retail is the largest channel, but **Online grew fastest** from 2019 to 2020 (0.6M to 2.2M).
6. **Product mix:** Food makes up 91% of revenue, and Wheat Flour is the single biggest product group.
7. **Salespeople:** Carla Ferreira leads in revenue and is the top performer in most product groups.

---

## Repository Structure

```
├── data/
│   ├── SalesData.xlsx
│   ├── Budget.xlsx
│   └── Product.xlsx
├── images/
│   ├── data_model.png
│   ├── page1_revenue_vs_budget.png
│   ├── page2_product_channel_salesperson.png
│   └── page3_salesperson_analysis.png
├── Sales_vs_Budget_Dashboard.pbix
└── README.md
```

---

**Ulegu Gladys Adanma**
Data Analyst | Excel · Power BI · DAX 

- GitHub: [github.com/glad-12](https://github.com/glad-12)
- LinkedIn: [linkedin.com/in/gladys-ulegu-a934a735](https://www.linkedin.com/in/gladys-ulegu-a934a735)
- Email: glademma234@gmail.com

If you found this project useful, please give it a star!
```
