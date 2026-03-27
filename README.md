# PSIL Customer Demand Intelligence System

> A full-stack Power BI analytics solution built to predict customer reorder cycles, profile size preferences, quantify inventory risk, and surface seasonal demand patterns — delivered as a cold-contact consulting pitch to Premier Standard Industrial Limited.

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-2D6E2D?style=flat)
![Power Query](https://img.shields.io/badge/Power%20Query%20M-7DC242?style=flat)
![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Status](https://img.shields.io/badge/Status-Consulting%20Project-orange?style=flat)
![Region](https://img.shields.io/badge/Region-Nigeria%20%C2%B7%20West%20Africa-blue?style=flat)

> **Note:** This project uses a simulated dataset (2,000 records) generated to answer the Managing Director's five business questions. The analytical framework, DAX measures, and dashboard architecture are production-ready and can be connected to live ERP data with a single source change.

---

## Contents

- [01 — Project overview](#01--project-overview)
- [02 — Business context](#02--business-context)
- [03 — Dataset structure](#03--dataset-structure)
- [04 — Repository structure](#04--repository-structure)
- [05 — Power BI architecture](#05--power-bi-architecture)
- [06 — DAX measures reference](#06--dax-measures-reference)
- [07 — Dashboard pages](#07--dashboard-pages)
- [08 — Key findings](#08--key-findings)
- [09 — How to use](#09--how-to-use)
- [10 — Roadmap](#10--roadmap)

---

## 01 — Project overview

Premier Standard Industrial Limited is a 40-year-old Nigerian distribution company and the sole distributor of Alltech Coppens fish feeds in Nigeria and West Africa. They also distribute printing inks and printing plates across five branches: Lagos, Aba, Ibadan, Port Harcourt, and Abuja.

This project was initiated as a cold-contact business intelligence pitch. After demonstrating a Power BI sales dashboard to the Managing Director, five strategic questions were posed. This repository contains the complete analytical solution built to answer them — including the dataset, the Power BI model, all DAX measures, dashboard designs, and a formal executive report.

### The five questions answered

| # | Question |
|---|---|
| 1 | When is Customer A likely to place their next order? |
| 2 | What product sizes does Customer A typically purchase? |
| 3 | What quantity should be prepared for their next order? |
| 4 | Which customers are most likely to order soon? |
| 5 | Which products should be restocked immediately? |

---

## 02 — Business context

| Dimension | Detail |
|---|---|
| Company | Premier Standard Industrial Limited (PSIL) |
| Founded | 1985 — incorporated in Lagos, Nigeria |
| Branches | Lagos (HQ), Aba, Ibadan, Port Harcourt, Abuja |
| Product lines | Fish Feed, Fish Meal, Poultry Feed, Printing Ink |
| Key partnership | Sole distributor of Alltech Coppens feeds in Nigeria and West Africa |
| Dataset period | January 2023 to June 2024 (18 months) |
| Total records | 2,000 order transactions across 10 customers |
| Total revenue | ₦12.05 billion net of discounts |

---

## 03 — Dataset structure

The dataset was generated to simulate a realistic order history for Premier Standard. It contains 2,000 rows and 16 columns covering all five branches, four product lines, four size variants, and three order channels.

```
File: premier_standard_inventory_demand_dataset.csv
Rows: 2,000  |  Columns: 16  |  Period: Jan 2023 – Jun 2024

Columns:
  Order_ID          Text      Unique order identifier (e.g. ORD10473)
  Date              Date      Order placement date
  Branch            Text      Lagos | Aba | Ibadan | Port Harcourt | Abuja
  Customer          Text      Customer A through Customer J
  Product           Text      Fish Feed | Fish Meal | Poultry Feed | Printing Ink
  Size_Label        Text      5kg | 10kg | 25kg | 50kg
  Size_KG           Integer   Numeric size for weight calculations
  Quantity_Units    Integer   Units ordered per line
  Total_Weight_KG   Integer   Quantity_Units x Size_KG
  Unit_Price        Decimal   Price per unit in Nigerian Naira
  Discount_Rate     Decimal   0.0 or 0.05 (binary — no discount or 5%)
  Discount_Amount   Decimal   Naira value of discount applied
  Revenue           Decimal   Net revenue after discount
  Inventory_Level   Integer   Stock on hand at time of order
  Lead_Time_Days    Integer   Supplier lead time (7 to 21 days)
  Order_Channel     Text      Direct Sales | Distributor | Retail Partner
```

> **Important:** Inventory_Level values were adjusted in a second dataset version (`premier_standard_inventory_updated.csv`) to produce all four inventory risk statuses — CRITICAL, WARNING, MONITOR, and ADEQUATE — for demonstration purposes. The original dataset produces only WARNING status due to uniform stock levels across all SKUs.

---

## 04 — Repository structure

```
psil-demand-intelligence/
├── data/
│   ├── premier_standard_inventory_demand_dataset.csv    # Original dataset
│   └── premier_standard_inventory_updated.csv           # Adjusted inventory levels
│
├── powerbi/
│   └── PSIL_BI_Analysis.pbix                            # Full Power BI report
│
├── report/
│   └── PSIL_Customer_Demand_Intelligence_Report.pdf     # Executive report (18 pages)
│
├── screenshots/
│   ├── 01_executive_overview.png
│   ├── 02_customer_reorder_predictor.png
│   ├── 03_size_quantity_profile.png
│   ├── 04_seasonality_demand.png
│   └── 05_inventory_reorder_planner.png
│
└── README.md
```

---

## 05 — Power BI architecture

### Data model — star schema

```
FactOrders              Central fact table — 2,000 rows — all transactional data
  └── DimCustomer       10 customers — segment, region, account manager
  └── DimProduct        16 product/size combinations — cost, lead time
  └── DimDate           1,096 rows — Jan 2023 to Dec 2025 — marked as Date Table
  └── CalcReorderProfile  10 rows — one per customer — computed in Power Query
      ├── LastOrderDate
      ├── AvgInterOrderDays
      ├── StdDevDays (via range proxy: Max – Min)
      ├── PredictedNextOrder = LastOrderDate + ROUND(AvgInterOrderDays, 0)
      ├── DaysUntilNextOrder = PredictedNextOrder – #date(2024, 6, 24)
      └── Confidence (High | Medium | Low based on range/avg ratio)
```

### Relationships

| From | To | Type | Direction |
|---|---|---|---|
| `FactOrders[Date]` | `DimDate[Date]` | Many-to-One | Single |
| `FactOrders[Customer]` | `DimCustomer[Customer]` | Many-to-One | Single |
| `FactOrders[Product+Size]` | `DimProduct[Product+Size]` | Many-to-One | Single |
| `FactOrders[Branch]` | `DimBranch[Branch]` | Many-to-One | Single |
| `CalcReorderProfile[Customer]` | `DimCustomer[Customer]` | Many-to-One | Single |

---

## 06 — DAX measures reference

All measures are stored in a dedicated `_Measures` table.

### Core sales metrics

```dax
Total Revenue      = SUM(FactOrders[Revenue])
Total Units        = SUM(FactOrders[Quantity_Units])
Total Orders       = DISTINCTCOUNT(FactOrders[Order_ID])
Avg Order Qty      = AVERAGE(FactOrders[Quantity_Units])
Avg Order Revenue  = DIVIDE([Total Revenue], [Total Orders])
```

### Reorder timing

```dax
Reference Date =
    CALCULATE(MAX(FactOrders[Date]))

Days Until Next Order =
    VAR RefDate  = CALCULATE(MAX(FactOrders[Date]), ALL(FactOrders))
    VAR PredDate = SELECTEDVALUE(CalcReorderProfile[PredictedNextOrder])
    RETURN DATEDIFF(RefDate, PredDate, DAY)

Order Due Status =
    VAR d = [Days Until Next Order]
    RETURN SWITCH(TRUE(),
        ISBLANK(d), "Unknown",
        d < 0,      "OVERDUE",
        d = 0,      "DUE TODAY",
        d <= 3,     "DUE THIS WEEK",
        d <= 7,     "DUE IN 7 DAYS",
        d <= 30,    "DUE THIS MONTH",
                    "NOT DUE SOON")

Customers Overdue =
    COUNTROWS(FILTER(CalcReorderProfile, [DaysUntilNextOrder] < 0))
```

### Inventory risk

```dax
Reorder Point =
    VAR D = [Demand Velocity]
    VAR L = [Avg Lead Time Days]
    VAR S = ROUND(D * 0.5 * SQRT(L), 0)
    RETURN ROUND((D * L) + S, 0)

Inventory Risk Status =
    VAR R = DIVIDE([Avg Inventory Level], [Demand Velocity])
    RETURN SWITCH(TRUE(),
        R < 5,   "CRITICAL - Reorder Now",
        R < 8,   "WARNING - Reorder Soon",
        R < 12,  "MONITOR",
                 "ADEQUATE")
```

### YoY comparison (like-for-like H1 only)

```dax
YoY Growth Display =
    VAR Current2024 = CALCULATE(SUM(FactOrders[Quantity_Units]),
        FILTER(ALL(DimDate), DimDate[Year] = 2024 && DimDate[MonthNumber] <= 6))
    VAR Prior2023   = CALCULATE(SUM(FactOrders[Quantity_Units]),
        FILTER(ALL(DimDate), DimDate[Year] = 2023 && DimDate[MonthNumber] <= 6))
    RETURN FORMAT(DIVIDE(Current2024 - Prior2023, Prior2023), "+0.0%;-0.0%")
           & " vs Jan–Jun 2023"
```

---

## 07 — Dashboard pages

| Page | Name | Key visuals |
|---|---|---|
| 01 | Executive overview | KPI cards, revenue by branch, monthly volume trend, orders by channel |
| 02 | Customer reorder predictor | Predicted next order, priority table sorted by urgency, avg qty chart |
| 03 | Size & quantity profile | 100% stacked bar, size preference heatmap, min/avg/max qty table |
| 04 | Seasonality & demand | Monthly column chart (2023 vs 2024), quarterly heatmap, YoY comparison |
| 05 | Inventory reorder planner | Risk status table, stock vs reorder point chart, demand velocity heatmap |

**Brand palette:** Dark green `#2D6E2D` · Lime green `#7DC242` · Logo black `#1A1A1A` · Canvas `#F2F7F2`

---

## 08 — Key findings

| Finding | Detail |
|---|---|
| Reorder cycles | All 10 customers order every 2.7 to 3.7 days — just-in-time behaviour |
| Size dominance | 10kg is the primary size for 9 of 10 customers (40–49% of volume) |
| Customer F anomaly | Only customer who prefers 25kg (38%) — highest avg order value at ₦7.17M |
| Revenue distribution | All five branches within 6% of each other — no single-point concentration risk |
| Seasonal peaks | January and May are the highest-demand months for agro-allied products |
| Critical inventory | Printing Ink 10kg (10-day cover vs 13-day lead) and Poultry Feed 50kg confirmed stockout risks |
| H1 2024 decline | Like-for-like volume down 12.8% vs H1 2023 — warrants investigation |

---

## 09 — How to use

### Prerequisites

- Power BI Desktop (latest version — free from Microsoft)
- Dataset CSV files from the `data/` folder
- Power BI Pro or Premium Per User licence (only needed for publishing)

### Getting started

```bash
# 1. Clone the repository
git clone https://github.com/yourusername/psil-demand-intelligence

# 2. Open the Power BI file
# File → Open → psil-demand-intelligence/powerbi/PSIL_BI_Analysis.pbix

# 3. Update the data source path
# Home → Transform Data → Source step in FactOrders
# Update the file path to your local data/ folder

# 4. Close & Apply
# Power BI reloads all tables and recalculates all measures

# 5. Navigate using the left sidebar
```


Built by **Moyin Odumewu** — Business Intelligence Division  
Dataset: simulated — reference date 24 June 2024 — Premier Standard Industrial Limited
