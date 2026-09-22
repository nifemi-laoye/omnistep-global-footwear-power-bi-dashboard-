# omnistep-global-footwear-power-bi-dashboard-
# Omnistep Footwear — Global Sales Performance Dashboard

A Power BI capstone project analyzing global sales, discounting behavior, and customer demographics for Omnistep Footwear, an international performance and lifestyle footwear brand operating across the USA, UK, Germany, India, UAE, and Pakistan.

## Project Overview

Omnistep's executive team had no reliable way to monitor performance in real time across its product categories (Running, Gym, Training, Basketball, Lifestyle). This surfaced three specific problems:

- **Profitability leakage** — high-discount categories (30%+) may have been eroding margins without a matching lift in sales volume
- **Customer misalignment** — no clear view of how customer income level relates to payment method and spend
- **Operational gaps** — no visibility into which regions or shoe sizes were underperforming, leading to inventory imbalances

This dashboard was built to answer three core questions:

1. **Is discounting actually driving sales volume**, or just cutting into margin?
2. **Who is buying, and how** — by income level, payment method, and product size?
3. **Where is revenue concentrated or lagging**, by category, product, and country?

## Data

**Source:** a single flat table of order-level transactions (`order_id`, `order_date`, `category`, `gender`, `size`, `color`, `base_price_usd`, `discount_percent`, `discount_category`, `units_sold`, `payment_method`, `sales_channel`, `country`, `customer_income_level`, `customer_rating`).

**Tables after modeling:**
| Table | Description |
|---|---|
| `Fact_Sales` | One row per order — keys, price, discount, units sold, income level, rating |
| `Dim_Product` | Unique Category / Gender / Size / Color combinations |
| `Dim_Sales` | Unique Payment Method / Sales Channel / Country / Discount Category combinations |
| `Dim_Date` | One row per order date, with Year, Quarter, Month, Weekday columns |

**Cleaning done in Power Query:**
- Set correct data types per column (Date, Text, Currency, Whole Number)
- Found and fixed a scaling bug in `discount_percent` — it was stored as whole numbers (e.g. `30`) instead of decimals (`0.30`), which would have inflated every percentage-based measure by 100x. Fixed with **Add Column → Standard → Divide by 100** at the source, rather than patching the DAX
- Built `Dim_Product` and `Dim_Sales` by selecting the relevant columns, removing duplicates, **then** adding an Index Column — in that order, so each surrogate key maps to exactly one unique combination
- Merged `ProductKey` and `SalesKey` back into `Fact_Sales` using **exact-match merges** (fuzzy matching intentionally left off), then removed the original descriptive text columns, leaving the fact table key-driven and numeric
- Deduplicated `Order Date` into `Dim_Date` and marked it as an official **Date Table** to enable time-intelligence functions

## Data Model

A star schema: one central fact table connected to three dimension tables.

```
Dim_Product ─┐
Dim_Sales ───┼──▶ Fact_Sales
Dim_Date ────┘
```

- `Fact_Sales[ProductKey]` → `Dim_Product[ProductKey]`
- `Fact_Sales[SalesKey]` → `Dim_Sales[SalesKey]`
- `Fact_Sales[Order Date]` → `Dim_Date[Order Date]` *(active relationship, required for time intelligence)*

`Customer Income Level` and `Customer Rating` were kept directly in `Fact_Sales` rather than split into a `Dim_Customer` table, since the dataset has no Customer ID — without one, there's no reliable way to tie multiple orders to the same person.

## DAX Measures

```DAX
Total Orders = DISTINCTCOUNT(Fact_Sales[Order ID])

Total Units Sold = SUM(Fact_Sales[Units Sold])

Total Revenue (Gross) =
SUMX(Fact_Sales, Fact_Sales[Base Price USD] * Fact_Sales[Units Sold])

Total Net Revenue =
SUMX(
    Fact_Sales,
    Fact_Sales[Base Price USD] * Fact_Sales[Units Sold] * (1 - Fact_Sales[Discount Percent])
)

Discount Impact Ratio =
DIVIDE(
    [Total Revenue (Gross)] - [Total Net Revenue],
    [Total Revenue (Gross)], 0
)

Average Discount % = AVERAGE(Fact_Sales[Discount Percent])

Average Customer Rating = AVERAGE(Fact_Sales[Customer Rating])

Average Order Value = DIVIDE([Total Net Revenue], [Total Orders], 0)

Revenue YTD = TOTALYTD([Total Net Revenue], Dim_Date[Order Date])

Revenue Prior Month =
CALCULATE([Total Net Revenue], DATEADD(Dim_Date[Order Date], -1, MONTH))
```

## Key Insights

1. **Discounting isn't driving volume.** Low-discount items (~5% avg) sold roughly **38K units**, versus far fewer units for High-discount items (30% avg) — the opposite of the common assumption that deeper markdowns move more product.
2. **Lifestyle is the leading category**, generating **$1.85M** in Net Revenue out of $9.08M total — narrowly ahead of Training ($1.84M) and Basketball ($1.82M). Within Lifestyle, **Women's Blue** is the single strongest micro-segment at **$136K**.
3. **Revenue is geographically concentrated but not evenly.** The **UAE leads** in Net Revenue, followed by the UK, India, and the US, with **Germany and Pakistan trailing** behind the rest.
4. **No single customer segment dominates.** Income level splits almost evenly (High 33.5%, Low 33.4%, Medium 33.1%), and payment method is similarly balanced (Bank Transfer 25.6%, Card 25.2%, Wallet 24.8%, Cash 24.4%) — but **Bank Transfer leads Net Revenue across every income tier** (High: $802K, Medium: $777K, Low: $768K), not Card or Wallet as the business assumed.
5. **Discounting is healthy overall.** The Discount Impact Ratio sits at **13.4%** of gross revenue — a sustainable level, even though the High discount tier specifically isn't earning its keep in volume.

## Recommendations

- **Stop using discount depth as a volume lever.** Reserve High-tier (30%) discounts for genuinely slow-moving stock, and protect pricing on proven performers like Women's Lifestyle rather than discounting them at the same rate as weaker products.
- **Diagnose before cutting investment in underperforming regions.** Pilot a localized marketing and payment-channel push in Pakistan and Germany for one to two quarters before reducing inventory or spend there.
- **Keep inventory and marketing broad-based.** Since income level, payment method, and product size all show fairly even demand, avoid concentrating resources on one segment — prioritize the Lifestyle → Women → Blue path as a baseline, while keeping the other top categories stocked in balance.
- **Support all four payment methods equally**, and lean into Bank Transfer promotion specifically for high-income customers, since it already leads spend in that segment.

<img width="500" height="275" alt="image" src="https://github.com/user-attachments/assets/c5dde79d-c972-45b0-9679-4cb015d3e2ad" />
<img width="500" height="276" alt="image" src="https://github.com/user-attachments/assets/11ecf745-ad80-425a-a634-f3bbdf039aac" />
<img width="750" height="409" alt="image" src="https://github.com/user-attachments/assets/85178f04-51c7-4928-8612-29d4d65101c9" />


