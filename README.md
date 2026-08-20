# Customer Shopping Behavior Analysis

End-to-end analysis of 3,900 retail transactions — cleaned in Python, queried in MySQL, and visualised in Power BI — to work out what actually drives revenue for a US apparel retailer, and what the company should stop doing.

**Headline:** the business runs on 43% discounted orders that generate a *lower* average order value than full-price ones, and 79.9% of its customers are repeat buyers, but only 27.6% of them subscribe.

---

## Why I built this

The brief was a fairly common one: management could see revenue going up and down but had no idea which levers moved it. Discounts, subscriptions, shipping upgrades, seasonal pushes — all of it was running at once, and nobody could say which was worth the money.

So the question I set out to answer was:

> How can the company use its consumer shopping data to identify trends, improve engagement, and optimise marketing and product strategy?

## The data

| | |
|---|---|
| Rows | 3,900 transactions |
| Columns | 18 raw → 19 after cleaning |
| Coverage | 50 US states, 4 categories, 25 products |
| Revenue | $233,081 |
| Avg. order value | $59.76 |
| Avg. review rating | 3.75 / 5 |

Fields cover demographics (age, gender, location), the purchase itself (item, category, amount, season, size, colour), and behaviour (review rating, subscription status, shipping type, discount, previous purchases, payment method, purchase frequency).

### 1. Python — cleaning and feature engineering

Nothing exotic here, just the work that makes the rest trustworthy:

- **37 missing review ratings** filled with the median rating of the same product category, rather than dropping rows and losing the revenue attached to them.
- **`Promo Code Used` turned out to be an exact duplicate of `Discount Applied`** — I checked equality across all 3,900 rows before dropping it, since keeping both would double-count the same behaviour.
- Column names standardised to lower snake_case so the table drops cleanly into MySQL and Power BI.
- Two new fields:
  - `age_group` — quartile binning into Young Adult / Adult / Middle-aged / Senior.
  - `purchase_frequency_days` — maps text like "Fortnightly" to 14, so buying cadence is actually sortable.

### 2. SQL — ten business questions

The cleaned table goes into MySQL (`CUSTOMER_BEHAVIOR.CUSTOMER`) and gets queried for:

1. Revenue and average purchase by gender
2. Discount users who still spend above average
3. Top 5 products by average review rating
4. Standard vs. Express shipping — does delivery speed track spend?
5. Subscribers vs. non-subscribers
6. Top 5 products by discount rate
7. Customer segmentation into New / Returning / Loyal (CTE + CASE)
8. Top 3 products within each category (`ROW_NUMBER() OVER (PARTITION BY ...)`)
9. Do repeat buyers (5+ prior purchases) subscribe?
10. Revenue contribution by age group

Mostly aggregation and grouping, with window functions and CTEs where ranking or segmentation was needed.

### 3. Power BI — the dashboard

A single-page interactive dashboard so a non-analyst can answer their own questions:

- KPI cards: number of customers, average purchase amount, average review rating
- Revenue by category and by age group
- Gender and subscription splits
- Shipping type comparison
- Slicers on gender, category, age group and subscription status

## What the data said

**Discounting isn't working.** 1,677 of 3,900 orders (43%) carried a discount. Those orders average **$59.28** — versus **$60.13** for full-price orders. The programme is giving away margin without lifting basket size.

**And it's only reaching half the customer base.** Every discount and every active subscription in this dataset belongs to a male customer. 63.2% of male customers got a discount, 39.7% subscribe — for female customers both figures are 0%, yet they still bring in $75,191 at a slightly *higher* average order ($60.25 vs. $59.54).

**Loyalty exists; the subscription funnel doesn't capture it.** 3,116 customers (79.9%) have 11+ prior purchases. Among repeat buyers with more than five prior orders, only 27.6% subscribe. The repeat behaviour a subscription is meant to create has already happened — it just isn't being converted.

**Almost every "obvious" segment is flat.** Seasonal revenue varies by only 7.6% (Summer $55,777 → Fall $60,018). Age quartile shares sit between 23.9% and 26.7%. The six payment methods land within 1% of each other. Average order value across six shipping options spans just $2.27. Category revenue differences come from **order volume, not basket size** — Clothing leads with $104,264 on 1,737 orders while AOV barely moves ($57.17–$60.26).

**Best-selling and best-rated are different products.** Top revenue: Blouse, Shirt, Dress, Pants, Jewelry — all within $400 of each other. Top rated: Gloves (3.86), Sandals (3.84), Boots (3.82). No overlap, which is a cross-sell opportunity sitting in plain sight.

## What I'd recommend

1. **Cap the discount programme** and reserve it for reactivation and first orders — it currently costs margin on 43% of orders and buys nothing.
2. **Trigger a subscription offer at the fifth order.** The loyalty is already there; only 27.6% of it converts.
3. **Open discounts and subscriptions to female customers** — an untouched segment already buying at full price.
4. **Bundle high-rated accessories with high-volume apparel** to lift basket size, since discounts demonstrably don't.
5. **Stop segmenting by season, age or payment method.** Segment on behaviour — the rest are cosmetic.

## Limitations

The dataset is a single cross-sectional snapshot. It records how many prior purchases a customer has, but no order dates — so cohort retention, churn timing and true CLV aren't calculable here. The perfectly clean gender split on discounts and subscriptions is unusual enough that I'd validate it against the source system before anyone acts on it. Order timestamps and acquisition channel would be the two highest-value additions.

## Tools

Python (pandas, Jupyter) · MySQL · Power BI · SQLAlchemy / mysql-connector

## How to run it

```bash
# 1. Clean the data
jupyter notebook python/customer_shopping_behavior_analysis.ipynb

# 2. Load the cleaned table into MySQL, then
mysql -u <user> -p < sql/Customer_Behavior_Analysis.sql

# 3. Open the dashboard
powerbi/Customer_Shopping_Behavior_Analysis.pbix
```

The full write-up, with charts and the reasoning behind each recommendation, is in [`report/`](report/).

---

## Built By Jayesh
**Data Analyst**
[LinkedIn](https://www.linkedin.com/in/jayesh-s-5566b9220/) · [Portfolio](https://jayesh-analytics.github.io/)

If you spot something I've missed or would have approached differently, I'd genuinely like to hear it.