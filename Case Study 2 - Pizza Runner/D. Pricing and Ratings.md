## Case Study #2 - Pizza Runner 🍕
## Data Analyzing (3) - Pricing and Ratings

This is solution for case study2. (written by Yumin Kim)

#### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
- Query :
    
    ```sql
    SELECT sum(case when pizza_id = 1 then 12
                else 10
                end) as total_revenue
    FROM pizza_runner.customer_orders_temp
     INNER JOIN pizza_runner.runner_orders_temp using (order_id)
    WHERE cancellation is NULL
    ```
    
- Results : 
    
    | total_revenue |
    | --- | 
    | 138 |

#### 2. What if there was an additional $1 charge for any pizza extras? Add cheese is $1 extra
- Query : 

  ```sql 
  WITH table1 as (
       SELECT *, length(extras) - length(replace(extras, ",", ""))+1 AS topping_count
       FROM pizza_runner.customer_orders_temp), 

      table2 as (
      SELECT sum(case when pizza_id = 1 then 12
                else 10
                end) as pizza_revenue, 
             sum(topping_count) as topping_revenue
      FROM table1 left join pizza_runner.runner_orders_temp using(order_id)
      WHERE cancellation is NULL )

  SELECT concat('$', pizza_revenue + topping_revenue) as total_revenue
  FROM table2
  ```
- Results : 
    | total_revenue |
    | --- | 
    | $142 |

#### 3. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?
- Query : 
  ```sql
  WITH table1 as (
       SELECT *, length(extras) - length(replace(extras, ",", ""))+1 AS topping_count
       FROM pizza_runner.customer_orders_temp), 

      table2 as (
      SELECT sum(case when pizza_id = 1 then 12
                else 10
                end) as pizza_revenue, 
             sum(topping_count) as topping_revenue,
             round(sum(0.30*distance), 2) AS delivery_cost
      FROM table1 left join pizza_runner.runner_orders_temp using(order_id)
      WHERE cancellation is NULL )

  SELECT concat('$', pizza_revenue + topping_revenue - delivery_cost) as total_revenue
  FROM table2
  ```
- Results : 
    | company_revenue |
    | --- | 
    | $77.38 |


