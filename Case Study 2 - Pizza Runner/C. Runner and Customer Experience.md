## Case Study #2 - Pizza Runner 🍕
## Data Analyzing (2) - Runner and Customer Experience

This is solution for case study2. (written by Yumin Kim)

#### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

1. Create CTE and select
    - runner_id
    - Actual registration date
    - Starting week. Calculate the starting week by
        - subtract the registration_date from '2021-01-01'
        - get the remainder of subtraction above from dividing by 7 (days in week)
        - subtract registration_date from remainder to derive start week.
2. Select runner_id and starting_week from CTE
3. Group and Order by starting week.
- Query :
    
    ```sql
    WITH runner_signups as (SELECT runner_id,
    		registration_date,
    		date_sub(registration_date, INTERVAL datediff(registration_date,'2021-01-01') %7 DAY) 
                 AS starting_week
    FROM pizza_runner.runners )
    
    SELECT starting_week,
    	count(runner_id) AS n_runners
    from runner_signups
    GROUP BY starting_week
    ORDER BY starting_week;
    ```
    
- Results :We can know the pattern when the runner first signed up the delivery service.
    
    
    | starting_week | n_runners |
    | --- | --- |
    | 2021-01-01 | 2 |
    | 2021-01-08 | 1 |
    | 2021-01-15 | 1 |

#### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

- Query :
    
    ```sql
    SELECT runner_id,
           round(avg(TIMESTAMPDIFF(MINUTE, order_time, pickup_time)), 2) avg_arrival_time
    FROM runner_orders_temp
    INNER JOIN customer_orders_temp USING (order_id)
    WHERE cancellation IS NULL
    GROUP BY runner_id;
    ```
    
- Results : It takes average 10 ~ 25 minutes for runners to arrive at pickup.
    
    
    | runner_id | avg_arrival_time |
    | --- | --- |
    | 1 | 15.33 |
    | 2 | 23.40 |
    | 3 | 10.00 |

#### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

- Query :
    
    ```sql
    WITH order_count_cte AS
      (SELECT order_id,
              COUNT(order_id) AS pizzas_order_count,
              TIMESTAMPDIFF(MINUTE, order_time, pickup_time) AS prep_time
       FROM runner_orders_temp
       INNER JOIN customer_orders_temp USING (order_id)
       WHERE cancellation IS NULL
       GROUP BY order_id)
    SELECT pizzas_order_count,
           round(avg(prep_time), 2) AS prep_time
    FROM order_count_cte
    GROUP BY pizzas_order_count;
    ```
    
- Results : There is a positive correlation between the number of orders and prep time. The more pizzas are orders, the more it takes to be prepared.
    
    
    | pizzas_order_count | prep_time |
    | --- | --- |
    | 1 | 12.00 |
    | 2 | 18.00 |
    | 3 | 29.00 |

#### 4a. What was the average distance travelled for each customer?

- Query :
    
    ```sql
    SELECT customer_id,
           round(avg(distance), 2) AS 'avg_distance'
    FROM runner_orders_temp
    INNER JOIN customer_orders_temp USING (order_id)
    WHERE cancellation IS NULL
    GROUP BY customer_id
    ORDER BY customer_id;
    ```
    
- Results : customers are located in average 10 ~ 25km far away.
    
    
    | customer_id | avg_distance |
    | --- | --- |
    | 101 | 20.00 |
    | 102 | 16.73 |
    | 103 | 23.40 |
    | 104 | 10.00 |
    | 105 | 25.00 |

#### 4b. What was the average distance travelled for each runner?

- Query :
    
    ```sql
    SELECT runner_id,
           round(avg(distance), 2) AS 'avg_distance'
    FROM runner_orders_temp
    WHERE cancellation IS NULL
    GROUP BY runner_id
    ORDER BY runner_id;
    ```
    
- Results : we can monitor the runner’s deliver distances.
    
    
    | runner_id | avg_distance |
    | --- | --- |
    | 1 | 15.85 |
    | 2 | 23.93 |
    | 3 | 10.00 |

#### 5. What was the difference between the longest and shortest delivery times for all orders?

- Query :
    
    ```sql
    SELECT MIN(duration) AS min_duration,
           MAX(duration) AS max_duration,
           MAX(duration) - MIN(duration) AS time_diff
    FROM runner_orders_temp;
    ```
    
- Results : longest time was 40 minutes, and shortest was 10 minutes. It seems there is quite big gap.
    
    
    | min_duration | max_duration | time_diff |
    | --- | --- | --- |
    | 10 | 40 | 30 |

#### 6. What is the successful delivery percentage for each runner?

- Query :
    
    ```sql
    SELECT runner_id,
           COUNT(pickup_time) AS delivered_orders,
           COUNT(*) AS total_orders,
           ROUND(100 * COUNT(pickup_time) / COUNT(*)) AS delivered_percentage
    FROM runner_orders_temp
    GROUP BY runner_id
    ORDER BY runner_id;
    ```
    
- Results : We can monitor the runner’s delivery quality.
    
    
    | runner_id | delivered_orders | total_orders | delivered_percentage |
    | --- | --- | --- | --- |
    | 1 | 4 | 4 | 100.0 |
    | 2 | 3 | 4 | 75.0 |
    | 3 | 1 | 2 | 50.0 |
