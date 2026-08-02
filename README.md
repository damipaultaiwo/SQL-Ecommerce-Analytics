# SQL Business Analysis: Olist E-Commerce Dataset

## Overview
This project applies SQL directly (not just pandas) to a real, relational e-commerce
dataset to answer a business question end to end: which product categories and regions
drive revenue, where does delivery performance break down, and which customers and
sellers matter most.

Nine queries run against an eight-table relational database of ~100,000 orders,
progressing from basic aggregation through multi-table joins, `HAVING`, CTEs and
subqueries, to window functions.

## Repository Structure
```
SQL-Ecommerce-Analytics/
├── data/                     # raw CSVs — not committed (see setup below)
├── notebooks/
│   └── olist_sql_analysis.ipynb
└── README.md
```

## Business Problem
Olist is a Brazilian e-commerce marketplace connecting small sellers to major online
storefronts. With ~100k orders across thousands of sellers and dozens of product
categories, three questions matter most to the business:

- Which product categories and states generate the most revenue?
- Where does delivery performance break down, and does it affect customer satisfaction?
- How concentrated is the platform across regions and sellers?

## Dataset
| Property | Detail |
|----------|--------|
| Source | Brazilian E-Commerce Public Dataset by Olist (Kaggle, `olistbr/brazilian-ecommerce`) |
| Records | 99,441 orders (Sep 2016 – Sep 2018), 112,650 order items, 99,224 reviews |
| Tables | 8 relational tables (orders, customers, order_items, products, sellers, payments, reviews, category translation) |
| Format | CSV, loaded into SQLite for querying |

## SQL Techniques Demonstrated
| # | Query | Technique |
|---|-------|-----------|
| 1 | Order status breakdown | `GROUP BY`, `COUNT` |
| 2 | Revenue by product category | 3-table `JOIN`, aggregation |
| 3 | Revenue by customer state | Multi-table `JOIN` |
| 4 | Avg. delivery time by state | Date arithmetic, `WHERE` |
| 5 | Above-average-spend customers | `WITH` (CTE) + scalar subquery |
| 6 | Problem categories (volume vs. satisfaction) | `HAVING` |
| 7 | Top 3 sellers per state | Window function — `RANK() OVER (PARTITION BY ...)` |
| 8 | Monthly revenue, running total | Window function — cumulative `SUM() OVER (...)` |
| 9 | Delivery speed vs. review score | `CASE WHEN` bucketing |

## Key Findings

**Delivery performance is the strongest driver of customer satisfaction in the data.**

| Delivery outcome | Orders | Share | Avg. review |
|---|---|---|---|
| On time or early | 88,658 | 92.0% | 4.29 |
| Late (up to 7 days) | 4,428 | 4.6% | 3.18 |
| Very late (7+ days) | 3,273 | 3.4% | 1.73 |

Orders arriving more than a week late score **60% lower** than on-time deliveries.
Delays cluster geographically in the North and Northeast — Roraima averages 29.4 days
against roughly 20 days in better-served states — pointing to a distribution-network gap
rather than random variance.

**Revenue is heavily concentrated.** São Paulo alone accounts for R$5.20M, **38% of all
platform revenue**; the top three states together represent **63%**.

**Volume and value diverge across categories.** `health_beauty` leads on total revenue
(R$1.26M / 8,836 orders) but `watches_gifts` earns nearly as much (R$1.21M) from 36%
fewer orders — R$214 per order versus R$142.

**Two high-volume categories underperform on satisfaction**: `bed_bath_table`
(9,313 orders, 3.90) and `computers_accessories` (6,649 orders, 3.93). At that volume,
a sub-4.0 score affects far more customers than a small category's poor rating.

**Growth flattened in 2018.** Cumulative revenue reached R$13.59M. Monthly revenue rose
steadily through 2017, spiked to R$1.01M in November 2017 (+52% MoM, consistent with
Black Friday), then plateaued around R$850K–1.0M through mid-2018.

## Analytical Notes
Two data-quality issues surfaced during analysis and are documented in the notebook:

- **`customer_id` is unique per order**, not per person (`customer_unique_id` is the
  durable identifier). Query 5 therefore returns highest-value *orders*, not verified
  repeat customers — a distinction that would change any retention recommendation built
  on top of it.
- **The first and last months are partial** (Sep 2016: R$267; Sep 2018: R$145). These are
  artifacts of the collection window, not real collapses, and would distort a naive
  trend line.

## How to Replicate

### Prerequisites
```
Python 3.8+
pandas
matplotlib
jupyter
sqlite3 (standard library)
```

Install dependencies:
```bash
pip install pandas matplotlib jupyter
```

### Steps
1. Download the dataset from Kaggle (`olistbr/brazilian-ecommerce`).
2. Place the CSV files in `data/`.
3. From `notebooks/`, run `jupyter notebook` and open `olist_sql_analysis.ipynb`.
4. Run all cells. The first cell builds `olist.db` from the CSVs; subsequent runs reuse it.

Note: `olist_geolocation_dataset.csv` is not used by any query in this notebook.

## Responsible Use
This is a historical (2016–2018), anonymized public dataset used for learning purposes
only. No real customer-identifying information is present or should be added.
