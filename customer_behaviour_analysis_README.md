# Customer Shopping Behavior Analysis

End-to-end analytics project on retail customer data: **Python** for cleaning and feature engineering, **MySQL** for business analysis, and **Power BI** for an interactive KPI dashboard.

<img width="1427" height="862" alt="image" src="https://github.com/user-attachments/assets/a51cc321-a8dd-4131-94c1-9e94e9645173" />

## Business Problem

A retail company wants to understand how customers shop so it can improve sales, satisfaction, and loyalty. Management has noticed shifting purchase patterns across demographics, product categories, and sales channels, and wants to know which factors (discounts, reviews, seasons, shipping, subscriptions) influence purchasing and repeat buying.

**Core question:** How can the company use customer shopping data to identify trends, improve engagement, and optimize marketing and product strategy?

## Dataset

One row per customer, describing their latest purchase.

| Group | Columns |
|---|---|
| Customer | customer ID, age, gender, location |
| Product | item purchased, category, size, color, season |
| Transaction | purchase amount (USD), review rating, shipping type, discount applied, promo code used, payment method |
| Behaviour | subscription status, previous purchases, frequency of purchases |

**Limitation:** the data is a snapshot, not a full transaction history. There are no past order values, only the *number* of previous purchases and a purchase-frequency label, so lifetime value and trend-over-time analysis are out of scope. The raw CSV is not included in this repository.

## Workflow

```
Raw CSV  ->  Python (clean + engineer features)  ->  MySQL (10 business queries)  ->  Power BI (dashboard)
```

### 1. Data preparation and EDA (Python, Pandas)
Notebook: [`customer_behaviour_trends_analysis.ipynb`](customer_behaviour_trends_analysis.ipynb)

| Step | What was done | Why |
|---|---|---|
| Inspect | `head`, `info`, `describe(include='all')`, null counts | Understand structure, types, and gaps |
| Missing values | Filled missing **review ratings with the median rating of the same category** (not the global median) | Ratings differ by category; a global median would bias clothing toward footwear and vice versa. The median is also robust to outliers |
| Column names | Lower-cased, spaces replaced with underscores, `purchase_amount_(usd)` renamed to `purchased_amount` | Consistent snake_case avoids quoting problems in Python and SQL |
| Feature: `age_group` | `pd.qcut` into 4 equal-sized groups (young adult, adult, middle aged, senior) | Enables demographic analysis |
| Feature: `purchase_frequency_days` | Mapped text frequencies (Weekly, Fortnightly, Monthly, Quarterly, Annually, ...) to number of days | Makes frequency comparable and calculable |
| Redundancy check | Verified `discount_applied == promo_code_used` for every row, then dropped `promo_code_used` | Two columns carried identical information |

### 2. Business analysis (MySQL)
Script: [`customer_behavior_portfolio.sql`](customer_behavior_portfolio.sql)

| # | Business question | Techniques |
|---|---|---|
| 1 | Revenue from male vs female customers | `SUM`, `GROUP BY` |
| 2 | Customers who used a discount but still spent above the average purchase | Subquery |
| 3 | Top 5 products by average review rating | `AVG`, `ROUND`, `ORDER BY`, `LIMIT` |
| 4 | Average purchase amount: Standard vs Express shipping | Filtered aggregation |
| 5 | Do subscribers spend more? (total and average spend) | Group comparison |
| 6 | Top 5 products with the highest share of discounted purchases | Conditional aggregation (`CASE`) |
| 7 | Segment customers into New / Returning / Loyal by previous purchases | `CASE` segmentation |
| 8 | Top 3 most purchased products within each category | CTE + `ROW_NUMBER() OVER (PARTITION BY ...)` |
| 9 | Are repeat buyers (more than 5 previous purchases) likely to subscribe? | Filtered group count |
| 10 | Revenue by age group | `SUM`, `GROUP BY`, `ORDER BY` |

`ROW_NUMBER` is used in query 8 rather than `RANK`/`DENSE_RANK` because several products tie on order count, and exactly three rows per category are needed.

### 3. Dashboard (Power BI)
File: [`customer_behaviour.pbix`](customer_behaviour.pbix) (open with Power BI Desktop; connect to your own MySQL instance or refresh from the cleaned data)

- **KPI cards:** number of customers, average purchase amount, average review rating (DAX measures)
- **Charts:** subscription split (donut), revenue by category, sales by category, revenue by age group, sales by age group
- **Slicers:** subscription status, gender, category, shipping type, so every visual can be filtered interactively

<!-- Add images/dashboard.png and uncomment the dashboard screenshot at the top of this README -->

## Key Observations

These come from running the queries above on this dataset:

- **Shipping:** Express-shipping customers have a higher average purchase amount than Standard-shipping customers.
- **Age:** Young adults generate the most revenue among the four age groups.
- **Subscriptions:** Subscribed and non-subscribed customers spend a similar amount on average, but non-subscribers contribute more total revenue because there are more of them.
- **Subscription gap among loyal buyers:** Most repeat buyers (more than 5 previous purchases) are *not* subscribed, which suggests the subscription offer is not attractive enough to the customers who already buy often.

## Suggested Actions (for discussion, not proven)

- Test targeted subscription offers aimed at repeat buyers who have not subscribed.
- Consider promoting faster delivery options, given higher spend among Express-shipping customers.
- Use top-rated products in marketing, and review products that depend heavily on discounts to sell.

## Repository Contents

| File | Description |
|---|---|
| `customer_behaviour_trends_analysis.ipynb` | Python cleaning, EDA, and feature engineering |
| `customer_behavior_portfolio.sql` | 10 business-question SQL queries (MySQL) |
| `customer_behaviour.pbix` | Power BI dashboard |
| `LICENSE` | MIT |

## How to Reproduce

1. Run the notebook on `customer_shopping_behavior.csv` and export the cleaned file.
2. Import the cleaned data into MySQL as table `customer_shopping_behavior` in database `customer_behaviour`.
3. Run `customer_behavior_portfolio.sql`.
4. Open `customer_behaviour.pbix` in Power BI Desktop and point it at your data source.

## Tools

Python (Pandas) · Google Colab / Jupyter · MySQL · Power BI (DAX)
