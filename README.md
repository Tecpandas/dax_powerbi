# DAX Practice – Power BI Project

A hands-on Power BI project built to practice core **DAX** concepts — aggregations, filter context (`CALCULATE`/`FILTER`), context transition (`ALL`/`ALLEXCEPT`/`ALLSELECTED`), logical functions, and set/table functions — on a sales dataset.

## 📁 Data model

| Table | Source | Purpose |
|---|---|---|
| `SALES` | `PowerBI_DAX_Practice_Data.csv` | Fact table — `OrderID`, `OrderDate`, `Product`, `Category`, `Region`, `City`, `Customer`, `Quantity`, `UnitPrice`, `Sales` |
| `DAX_Practice_CustomerList2` | `DAX_Practice_CustomerList2.csv` | Practice table for calculated columns (customer targeting/eligibility logic) |
| `INTERSET` | Calculated table | Customers common to both tables (see below) |
| Auto date tables | Power BI auto-generated | Date hierarchies for `OrderDate` |

No relationships were built between `SALES` and `DAX_Practice_CustomerList2` — deliberately, so every calculation had to be written explicitly rather than relying on the model to resolve context.

## 🧮 Measures (`SALES` table)

**Basic aggregations**
```dax
TOTAL_SALES         = SUM(SALES[Sales])
TOTAL_REVENUE       = SUMX(SALES, SALES[Quantity] * SALES[UnitPrice])
average_unit_price  = AVERAGE(SALES[UnitPrice])
averagre_per_day    = AVERAGEX(SALES, SALES[Quantity] * SALES[UnitPrice])
no_of_order_id      = COUNT(SALES[OrderID])
customer_values     = COUNTA(SALES[Customer])
TOTAL_ORDER         = COUNTROWS(SALES)
UNIQUE_CUSTOMER     = DISTINCTCOUNT(SALES[Customer])
MI_UNIT_PRICES      = MIN(SALES[UnitPrice])
MAX_UNIT_PRICES     = MAX(SALES[UnitPrice])
```

**Filter context with `CALCULATE` / `FILTER`**
```dax
WEST_SALES        = CALCULATE(SUM(SALES[Sales]), SALES[REGION] = "West")
TOTAL_ELECTRONICS = CALCULATE(SUM(SALES[Sales]), SALES[Category] = "Electronics")
region_north      = CALCULATE(SUM(SALES[Sales]), SALES[Region] = "North")
west              = CALCULATE(SUM(SALES[Sales]), FILTER(SALES, SALES[Region] = "west"))
furnite           = CALCULATE(SUM(SALES[Sales]), FILTER(SALES, SALES[Category] = "Furniture"))
sales_over        = CALCULATE(SUM(SALES[Sales]), FILTER(SALES, SALES[Sales] > 10000))

-- multiple conditions
two_condition = CALCULATE(SUM(SALES[Sales]),
    FILTER(SALES, SALES[Region] = "West"),
    FILTER(SALES, SALES[Category] = "Electronics"))

conditional = CALCULATE(SUM(SALES[Sales]),
    FILTER(SALES, SALES[Category] = "Electronic" && SALES[Region] = "West"))
```

**Context transition — `ALL` / `ALLEXCEPT` / `ALLSELECTED`**
```dax
all               = CALCULATE(SUM(SALES[Sales]), ALL(SALES[Region]))
all_except        = CALCULATE(SUM(SALES[Sales]), ALLEXCEPT(SALES, SALES[Region]))
Sales_Keep_Region = CALCULATE(SUM(SALES[SALES]), ALLEXCEPT(SALES, SALES[REGION]))
all_selected      = CALCULATE(SUM(SALES[Sales]), ALLSELECTED(SALES))
```

## 🧩 Calculated columns (`DAX_Practice_CustomerList2`)

Logical/branching functions applied on top of the measures above:

```dax
target = IF(SALES[TOTAL_SALES] > 50000, "target achieved", "not achieved")

conditions =
IF(SALES[TOTAL_SALES] > 100000, "excellent",
  IF(SALES[TOTAL_SALES] > 75000, "good",
    IF(SALES[TOTAL_SALES] > 50000, "average", "poor")))

Sales Category =
SWITCH(TRUE(),
    SALES[TOTAL_SALES] >= 100000, "excellent",
    SALES[sales_over] >= 75000, "good",
    SALES[TOTAL_SALES] >= 50000, "average",
    "poor")

Eligibility =
IF(AND(SALES[customer_values] >= 10000, SALES[TOTAL_ORDER] > 10),
    "Eligible", "Not Eligible")

Priority = IF(OR(SALES[TOTAL_SALES] >= 100000, SALES[TOTAL_ORDER] >= 10),
    "high priority", "normal priority")

Status = IF(NOT(SALES[TOTAL_SALES] >= 100000), "high sales", "low sales")
```

## 🔗 Calculated table — set functions

```dax
INTERSET =
INTERSECT(
    VALUES(DAX_Practice_CustomerList2[Column1]),
    VALUES(SALES[Customer])
)
```
Returns customers that appear in both the sales data and the practice customer list.

## 📊 Report page

- **Pie chart** — `target` (achieved/not achieved) split by region count
- **KPI card** — `TOTAL_SALES` tracked against the `conditions` tier
- **Clustered column chart** — Sum of `Sales`

## 🧠 Concepts covered

- Aggregation functions (`SUM`, `AVERAGE`, `COUNT`, `COUNTA`, `COUNTROWS`, `DISTINCTCOUNT`, `MIN`, `MAX`)
- Iterators (`SUMX`, `AVERAGEX`)
- Filter context manipulation (`CALCULATE`, `FILTER`)
- Context transition (`ALL`, `ALLEXCEPT`, `ALLSELECTED`)
- Logical functions (`IF`, `SWITCH`, `AND`, `OR`, `NOT`)
- Set/table functions (`INTERSECT`, `VALUES`)

## 🛠️ Tools

Power BI Desktop, DAX

## ▶️ How to view

1. Clone/download this repo
2. Open `DAX.pbix` in Power BI Desktop
3. Check the Data view (calculated columns) and Model view (`INTERSET` table) alongside the report page
