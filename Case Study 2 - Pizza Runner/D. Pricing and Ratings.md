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
