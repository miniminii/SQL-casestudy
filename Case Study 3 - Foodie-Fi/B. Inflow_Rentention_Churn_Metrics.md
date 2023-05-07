## Case Study #3 - Foodie-Fi 🥑
## Data Analyzing  - Inflow/Retention/Churn  Metrics

This is solution for case study2. (written by Yumin Kim)

#### 1. How many customers has Foodie-Fi ever had?

- Query :
    
    ```sql
    SELECT count(DISTINCT customer_id) AS 'n_customer'
    FROM foodie_fi.subscriptions;
    ```
    
- Results : Total 1000 customer has subscribed.
    
    
    | n_customer |
    | --- |
    | 1000 |

#### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

- Query :
    
    ```sql
    SELECT month(start_date) as 'month',
           count(DISTINCT customer_id) as 'n_customers'
    FROM foodie_fi.subs_plans
    WHERE plan_id=0
    GROUP BY month(start_date);
    ```
    
- Results : The largest number of customers flowed in in March, and the lowest number of subscribers in February.
    
    
    | month | n_customers |
    | --- | --- |
    | 1 | 88 |
    | 2 | 68 |
    | 3 | 94 |
    | 4 | 81 |
    | 5 | 88 |
    | 6 | 79 |
    | 7 | 89 |
    | 8 | 88 |
    | 9 | 87 |
    | 10 | 79 |
    | 11 | 75 |
    | 12 | 84 |

#### 3.. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name*

- Query :
    
    ```sql
    SELECT plan_id,plan_name, count(plan_name) AS n_plans
    FROM foodie_fi.sub_plans
    WHERE start_date >= '2021-01-01'
    GROUP BY plan_id,plan_name
    ORDER BY plan_id;
    ```
    
- Results : Since 2020, there have been the largest number of customers who have churned, but there have also been many customers flowing into ‘pro-annual plan’.
    
    
    | plan_id | plan_name | n_plans |
    | --- | --- | --- |
    | 1 | basic monthly | 8 |
    | 2 | pro monthly | 60 |
    | 3 | pro annual | 63 |
    | 4 | churn | 71 |

**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**

- Query :
    
    ```sql
    SELECT plan_name, count(DISTINCT customer_id) as 'churned customers' , ROUND(100 * count(DISTINCT customer_id) / 
                 (SELECT count(DISTINCT customer_id) 
                  FROM foodie_fi.sub_plans ),2) AS 'percentage'
    FROM foodie_fi.sub_plans
    GROUP BY plan_name
    HAVING plan_name = 'churn'
    ```
    
- Results : 307 customers have churned.
    
    
    | plan_name | churned customers | percentage |
    | --- | --- | --- |
    | churn | 307 | 30.70 |

**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**

- Query :
    
    ```sql
    WITH next_plan_cte AS -- create cte1
        (SELECT *,
              lead(plan_id, 1) over(PARTITION BY customer_id
                                    ORDER BY start_date) AS next_plan
         FROM foodie_fi.subscriptions),
         churners AS -- create cte2
        (SELECT *
         FROM next_plan_cte
         WHERE next_plan=4
           AND plan_id=0)
    
    SELECT count(customer_id) AS 'churn after trial',
           round(100 *count(customer_id)/
                   (SELECT count(DISTINCT customer_id) AS 'distinct customers'
                    FROM foodie_fi.subscriptions), 2) AS 'percentage'
    FROM churners;
    ```
    
- Results : 9.2% of customers have churned straight after their initial free trial. And it also means 90.8% of customers successfully converged into premium plan.
    
    
    | churn after trial | percentage |
    | --- | --- |
    | 92 | 9.2 |

**6. What is the number and percentage of customer plans after their initial free trial?**

- Query :
    
    ```sql
    WITH table1 AS -- create cte1
        (SELECT *,
              lead(plan_id, 1) over(PARTITION BY customer_id
                                    ORDER BY start_date) AS next_plan
         FROM foodie_fi.sub_plans),
    
         table2 AS -- create cte2
        (SELECT *
         FROM table1
         WHERE next_plan IS NOT NULL
           AND plan_id=0)
    
    SELECT next_plan, p.plan_name, count(next_plan) as 'plan after trial', 
           round(100* count(next_plan)/
           (select count(distinct customer_id) from foodie_fi.sub_plans),2) as 'percentage'
    FROM table2 as t 
         INNER JOIN foodie_fi.plans as p ON t.next_plan = p.plan_id
    GROUP BY next_plan,p.plan_name
    ```
    
- Results : They converged into basic monthly > pro monthly > pro annual plan after first time to trial the platform.
    
    
    | next_plan | plan_name | plan after trial | percentage |
    | --- | --- | --- | --- |
    | 1 | basic monthly | 546 | 54.60 |
    | 2 | pro monthly | 325 | 32.50 |
    | 3 | pro annual | 37 | 3.70 |
    | 4 | churn | 92 | 9.20 |

**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**

- Query :
    
    ```sql
    WITH latest_plan_cte AS
      (SELECT *,
              row_number() over(PARTITION BY customer_id
                                ORDER BY start_date DESC) AS latest_plan
       FROM foodie_fi.sub_plans
       WHERE start_date <='2020-12-31' )
    
    SELECT plan_id,
           plan_name,
           count(customer_id) AS customer_count,
           round(100*count(customer_id) /
                   (SELECT COUNT(DISTINCT customer_id)
                    FROM foodie_fi.subscriptions), 2) AS percentage_breakdown
    FROM latest_plan_cte
    WHERE latest_plan = 1
    GROUP BY plan_id,plan_name
    ORDER BY plan_id;
    ```
    
- Results : Until 2020, consumers mostly subscribed pro-monthly.
    
    
    | plan_id | plan_name | customer_count | percentage_breakdown |
    | --- | --- | --- | --- |
    | 0 | trial | 19 | 1.90 |
    | 1 | basic monthly | 224 | 22.40 |
    | 2 | pro monthly | 326 | 32.60 |
    | 3 | pro annual | 195 | 19.50 |
    | 4 | churn | 236 | 23.60 |

**8. How many customers have upgraded to an annual plan in 2020?**

- Query :
    
    ```sql
    WITH previous_plan_cte AS
      (SELECT *,
              lag(plan_id, 1) over(PARTITION BY customer_id
                                   ORDER BY start_date) AS previous_plan_id
       FROM foodie_fi.sub_plans)
    
    SELECT count(customer_id) as 'count'
    FROM previous_plan_cte
    WHERE previous_plan_id<3
      AND plan_id=3
      AND year(start_date) = 2020;
    ```
    
- Results : we can know the number of customer who upgraded the plan at certain year.
    
    
    | count |
    | --- |
    | 195 |

**9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?**

- Query :
    
    ```sql
    WITH first_plan_customer_cte AS
      (SELECT *,
              row_number() over(PARTITION BY customer_id
                                ORDER BY start_date ASC) AS plan_order
       FROM foodie_fi.sub_plans),
    
         annual_plan_customer_cte AS
      (SELECT *
       FROM foodie_fi.sub_plans
       WHERE plan_id=3)
    
    SELECT round(avg(datediff(annual_plan_customer_cte.start_date, first_plan_customer_cte.start_date)), 2)AS avg_conversion_days
    FROM first_plan_customer_cte
    INNER JOIN annual_plan_customer_cte USING (customer_id)
    WHERE plan_order = 1
    ```
    
- Results : Among the customer who converged into annual plan, they takes 104.62 days.
    
    
    | avg_days |
    | --- |
    | 104.62 |

**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)**

- Query :
    
    ```sql
    WITH first_plan_customer_cte AS
      (SELECT *,
              row_number() over(PARTITION BY customer_id
                                ORDER BY start_date ASC) AS  plan_order
       FROM foodie_fi.sub_plans),
    
         annual_plan_customer_cte AS
      (SELECT *
       FROM foodie_fi.sub_plans
       WHERE plan_id=3),
    
         diff_plan_cte AS
      ( SELECT f.customer_id, f.start_date as first_date, a.start_date as upgrade_date, round(datediff(a.start_date, f.start_date) / 30) as date_periods
        FROM first_plan_customer_cte AS f
        INNER JOIN annual_plan_customer_cte AS a ON f.customer_id = a.customer_id
        WHERE plan_order = 1)
    
    SELECT CASE
    		WHEN date_periods = 0 THEN '0 - 30days'
    		ELSE CONCAT((date_periods * 30 + 1), ' - ' , (date_periods+1) * 30, ' days')
    	    END AS time_period, count(customer_id) AS customer_count
    from diff_plan_cte
    GROUP BY time_period ;
    ```
    
- Results : we can narrow down the average value (=104.62 days) into certain periods
    
    
    | time_period | customer_count |
    | --- | --- |
    | 0 - 30days | 41 |
    | 151 - 180 days | 41 |
    | 61 - 90 days | 30 |
    | 31 - 60 days | 25 |
    | 121 - 150 days | 33 |
    | 91 - 120 days | 39 |
    | 181 - 210 days | 32 |
    | 211 - 240 days | 7 |
    | 361 - 390 days | 1 |
    | 241 - 270 days | 4 |
    | 271 - 300 days | 3 |
    | 301 - 330 days | 1 |
    | 331 - 360 days | 1 |

**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**

- Query :
    
    ```sql
    WITH next_plan_cte AS
      (SELECT *,
              lead(plan_id, 1) over(PARTITION BY customer_id
                                    ORDER BY start_date) AS next_plan
       FROM foodie_fi.subscriptions)
    
    SELECT count(*) AS downgrade_count
    FROM next_plan_cte
    WHERE plan_id=2
      AND next_plan=1
      AND year(start_date);
    ```
    
- Results : Hopefully, there is no one who downgraded the plan.
    
    
    | downgrade_count |
    | --- |
    | 0 |
