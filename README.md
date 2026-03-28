# PSIL Customer Demand Intelligence System

<div align="center">

**A Power BI analytics solution that predicts when customers will reorder, what they buy, and what to stock — built for Premier Standard Industrial Limited, Nigeria.**

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-2D6E2D?style=flat&logoColor=white)
![Power Query M](https://img.shields.io/badge/Power%20Query%20M-7DC242?style=flat&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Records](https://img.shields.io/badge/Dataset-2%2C000%20records-lightgrey?style=flat)
![Revenue](https://img.shields.io/badge/Revenue%20analysed-%E2%82%A612.05B-green?style=flat)

</div>

---

## The Business Problem

Premier Standard Industrial Limited (PSIL) is a 40-year-old Nigerian distribution company and sole distributor of Alltech Coppens fish feeds in Nigeria and West Africa. Operating across five branches — Lagos, Aba, Ibadan, Port Harcourt, and Abuja — they distribute four product lines to ten active customer accounts.

Management had no systematic way to anticipate demand. Orders were reactive. Stockouts were untracked. Customer purchasing patterns were unknown.

The Managing Director asked five questions:

1. When is **Customer A** likely to place their next order?
2. What **product sizes** does Customer A typically buy?
3. What **quantity** should we prepare?
4. Which **customers** are likely to order soon?
5. Which **products** should we restock immediately?

This solution answers all five — with data.

---

## Dashboards

### Page 1 — Executive Demand Overview
KPI cards for overdue customers, revenue, and projected pipeline. Revenue by branch, monthly volume trend, and orders by channel.

![Executive Overview](01_executive_overview.png)

---

### Page 2 — Customer Reorder Predictor
Predicts the next order date for every customer based on their individual reorder cycle. Priority table sorted by urgency — most overdue first — with colour-coded status badges.

![Customer Reorder Predictor](02_customer_reorder_predictor.png)

---

### Page 3 — Size & Quantity Profile
100% stacked bar showing each customer's size preference mix. Size preference heatmap. Min / Avg / Max quantity reference table per customer per size.

![Size & Quantity Profile](03_size_quantity_profile.png)

---

### Page 4 — Seasonality & Demand Planning
Monthly clustered column chart comparing 2023 vs 2024. Quarterly demand heatmap by product. Year-on-year comparison and demand highlight cards.

![Seasonality & Demand](04_seasonality_demand.png)

---

### Page 5 — Inventory Risk & Reorder Planner
Full inventory risk table: CRITICAL · WARNING · MONITOR · ADEQUATE per product/size. Stock vs Reorder Point chart. Demand velocity heatmap.

![Inventory Reorder Planner](05_inventory_reorder_planner.png)

---

## Key Findings

| Finding | Detail |
|---|---|
| **6 of 10 customers overdue** | Six accounts had passed their predicted next order date as of 24 June 2024 |
| **Customer F — highest risk** | 7 days overdue · ₦7.17M average order value · unique 25kg size preference |
| **10kg dominates** | Primary size for 9 of 10 customers at 40–49% of each customer's volume |
| **January & May are peak months** | Highest demand for agro-allied products — aligns with Nigerian aquaculture cycles |
| **2 products CRITICAL** | Printing Ink 10kg (10-day cover vs 13-day lead) · Poultry Feed 50kg (12 vs 14 days) |
| **Revenue evenly distributed** | All five branches within 6% of each other — no single-point concentration risk |
| **H1 2024 down 12.8%** | Like-for-like volume decline vs H1 2023 — flagged for management investigation |

---

## Data Model

```
FactOrders (2,000 rows)
  ├── DimDate             — 1,096 rows · Jan 2023–Dec 2025 · marked as Date Table
  ├── DimCustomer         — 10 customers
  ├── DimProduct          — 16 product/size combinations
  └── CalcReorderProfile  — 10 rows · one per customer · built in Power Query
        ├── AvgInterOrderDays
        ├── PredictedNextOrder = LastOrderDate + ROUND(AvgInterOrderDays, 0)
        ├── DaysUntilNextOrder = PredictedNextOrder − #date(2024, 6, 24)
        └── Confidence = High | Medium | Low
```

---

## Selected DAX Measures

```dax
-- Predicted next order status
Order Due Status =
    VAR d = [Days Until Next Order]
    RETURN SWITCH(TRUE(),
        d < 0,   "OVERDUE",
        d = 0,   "DUE TODAY",
        d <= 3,  "DUE THIS WEEK",
        d <= 7,  "DUE IN 7 DAYS",
                 "NOT DUE SOON")

-- Inventory risk classification
Inventory Risk Status =
    VAR R = DIVIDE([Avg Inventory Level], [Demand Velocity])
    RETURN SWITCH(TRUE(),
        R < 5,   "CRITICAL - Reorder Now",
        R < 8,   "WARNING - Reorder Soon",
        R < 12,  "MONITOR",
                 "ADEQUATE")

-- Recommended reorder quantity
Recommended Reorder Qty =
    VAR D = [Demand Velocity]
    VAR L = [Avg Lead Time Days]
    VAR S = ROUND(D * 0.5 * SQRT(L), 0)
    RETURN ROUND((D * L * 2) + S, 0)
```

---

## Dataset

| Field | Type | Description |
|---|---|---|
| `Order_ID` | Text | Unique order identifier |
| `Date` | Date | Order placement date |
| `Branch` | Text | Lagos · Aba · Ibadan · Port Harcourt · Abuja |
| `Customer` | Text | Customer A through Customer J |
| `Product` | Text | Fish Feed · Fish Meal · Poultry Feed · Printing Ink |
| `Size_Label` | Text | 5kg · 10kg · 25kg · 50kg |
| `Quantity_Units` | Integer | Units ordered per line |
| `Revenue` | Decimal | Net revenue in Nigerian Naira after discount |
| `Inventory_Level` | Integer | Stock on hand at time of order |
| `Lead_Time_Days` | Integer | Supplier lead time — 7 to 21 days |

> The dataset was simulated to answer the MD's five business questions. A second version (`premier_standard_inventory_updated.csv`) adjusts inventory levels to demonstrate all four risk statuses across the product range.

---

## Deliverables

| File | Description |
|---|---|
| `PSIL_BI_Analysis.pbix` | Full Power BI report — five dashboards |
| `premier_standard_inventory_demand_dataset.csv` | 2,000-record simulated order dataset |
| `premier_standard_inventory_updated.csv` | Adjusted inventory for full risk status demonstration |
| `PSIL_Customer_Demand_Intelligence_Report.pdf` | 18-page executive intelligence report |

---

## Brand

| Role | Hex |
|---|---|
| Primary green | `#2D6E2D` |
| Accent lime | `#7DC242` |
| Logo black | `#1A1A1A` |
| Canvas | `#F2F7F2` |

---

<div align="center">

Built by **Moyin Odumewu** · Business Intelligence  Analyst

</div>
