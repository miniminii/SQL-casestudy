## Case Study #3 - Foodie-Fi 🥑
## 0. Customer Journey

This is solution for case study2. (written by Yumin Kim)

Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

- Query :

```sql
DROP TABLE IF EXISTS subs_plans;
CREATE TABLE subs_plans AS (
	SELECT s.customer_id,
		s.plan_id,
		p.plan_name,
		p.price,
		s.start_date
	FROM foodie_fi.subscriptions AS s
		JOIN foodie_fi.plans AS p ON s.plan_id = p.plan_id
);
```

```sql
SELECT customer_id,
	plan_name,
	start_date
FROM foodie_fi.subs_plans
WHERE customer_id IN (1, 2, 11, 13, 15, 16, 18, 19)
ORDER BY customer_id,
	plan_id ASC;
```

- Results :

| customer_id | plan_name | start_date |
| --- | --- | --- |
| 1 | trial | 2020-08-01 |
| 1 | basic monthly | 2020-08-08 |
| 2 | trial | 2020-09-20 |
| 2 | pro annual | 2020-09-27 |
| 11 | trial | 2020-11-19 |
| 11 | churn | 2020-11-26 |
| 13 | trial | 2020-12-15 |
| 13 | basic monthly | 2020-12-22 |
| 13 | pro monthly | 2021-03-29 |
| 15 | trial | 2020-03-17 |
| 15 | pro monthly | 2020-03-24 |
| 15 | churn | 2020-04-29 |
| 16 | trial | 2020-05-31 |
| 16 | basic monthly | 2020-06-07 |
| 16 | pro annual | 2020-10-21 |
| 18 | trial | 2020-07-06 |
| 18 | pro monthly | 2020-07-13 |
| 19 | trial | 2020-06-22 |
| 19 | pro monthly | 2020-06-29 |
| 19 | pro annual | 2020-08-29 |
- **Client #1**: upgraded to the basic monthly subscription within their 7 day trial period.
- **Client #2**: upgraded to the pro annual subscription within their 7 day trial period.
- **Client #11**: cancelled their subscription within their 7 day trial period.
- **Client #13**: upgraded to the basic monthly subscription within their 7 day trial period and upgraded to pro annual 3 months later.
- **Client #15**: upgraded to the pro annual subscription within their 7 day trial period and cancelled the following month.
- **Client #16**: upgraded to the basic monthly subscription after their 7 day trial period and upgraded to pro annual almost 5 months later.
- **Client #18**: upgraded to the pro monthly subscription within their 7 day trial period.
- **Client #19**: upgraded to the pro monthly subscription within their 7 day trial period and upgraded to pro annual 2 months later.
