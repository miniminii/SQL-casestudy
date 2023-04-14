## Case Study #2 - Pizza Runner 🍕
## Data Analyzing (1) - Order Metrics

This is solution for case study2. (written by Yumin Kim)

#### 1. How many pizzas were ordered?

- Query :
    
    ```sql
    SELECT count(pizza_id) AS "total_order"
    FROM pizza_runner.customer_orders_temp;
    ```
    
- Results : 14 orders were made right now.
    
    
    | total_order |
    | --- |
    | 14 |

**2. How many unique customer orders were made?**

- Query :
    
    ```sql
    SELECT 
      COUNT(DISTINCT order_id) AS 'n_order'
    FROM customer_orders_temp;
    ```
    
- Results : 10 unique orders were made until now.
    
    
    | n_order |
    | --- |
    | 10 |

**3. How many times has each customer orderd/reorderd pizza?**

- Query :

```sql
-- distict time
SELECT customer_id, GROUP_CONCAT(distinct order_time)as order_date, 
        count(distinct order_time) as n_order
FROM pizza_runner.customer_orders_temp
GROUP BY customer_id
ORDER BY customer_id;
```

```sql
-- distinct day
SELECT customer_id, GROUP_CONCAT(distinct date_format(order_time, '%Y-%m-%d')) as order_date, 
        count(distinct(date_format(order_time, '%Y-%m-%d'))) as n_order
FROM pizza_runner.customer_orders_temp
GROUP BY customer_id
ORDER BY customer_id;
```

- Results : Almost every customer (except customer_id = ‘105’) reorder pizza.
    
    
    | customer_id | order_date | n_order |
    | --- | --- | --- |
    | 101 | 2020-01-01 18:05:02, 2020-01-01 19:00:52, 2020-01-08 21:03:13 | 3 |
    | 102 | 2020-01-02 23:51:23,2020-01-09 23:54:33 | 2 |
    | 103 | 2020-01-04 13:23:46,2020-01-10 11:22:59 | 2 |
    | 104 | 2020-01-08 21:00:29,2020-01-11 18:34:49 | 2 |
    | 105 | 2020-01-08 21:20:29 | 1 |
    
    | customer_id | order_date | n_order |
    | --- | --- | --- |
    | 101 | 2020-01-01,2020-01-08 | 2 |
    | 102 | 2020-01-02,2020-01-09 | 2 |
    | 103 | 2020-01-04,2020-01-10 | 2 |
    | 104 | 2020-01-08,2020-01-11 | 2 |
    | 105 | 2020-01-08 | 1 |

**4. How long does it takes to re-order pizza by each customer ?** 

- Query :
    
    ```sql
    WITH Table1 AS (
    SELECT customer_id, max(order_time) as last_order, min(order_time) as first_order, count(distinct order_time) as n_order
    FROM pizza_runner.customer_orders_temp
    GROUP BY customer_id
    ORDER BY customer_id)
    
    SELECT customer_id, CONCAT(ROUND(DATEDIFF( last_order,first_order)/(n_order-1),2),'days') as cycle
    FROM Table1
    WHERE n_order <> 1;
    ```
    
- Results : 4.875days (average for all customers) takes to reorder.
    
    
    | customer_id | cycle |
    | --- | --- |
    | 101 | 3.50days |
    | 102 | 7.00days |
    | 103 | 6.00days |
    | 104 | 3.00days |

**5. How many of each type of pizza was delivered?**

- Query :
    
    ```sql
    SELECT p.pizza_name,
    	count(c.*) AS n_pizza_type
    FROM pizza_runner.customer_orders_temp AS c
    	LEFT JOIN pizza_runner.pizza_names AS p ON p.pizza_id = c.pizza_id
    	LEFT JOIN pizza_runner.runner_orders_temp AS r ON c.order_id = r.order_id
    WHERE cancellation IS NULL
    GROUP BY p.pizza_name
    ORDER BY n_pizza_type DESC;
    ```
    
- Results : meatlovers are orders 9 times, and vegetarian orders 3 times.
    
    
    | pizza_name | n_pizza_type |
    | --- | --- |
    | Meatlovers | 9 |
    | Vegetarian | 3 |

**6. How many Vegetarian and Meatlovers were ordered by each customer?**

- Query :
    
    ```sql
    SELECT customer_id,
           SUM(CASE
                   WHEN pizza_id = 1 THEN 1
                   ELSE 0
               END) AS 'meat_lovers',
           SUM(CASE
                   WHEN pizza_id = 2 THEN 1
                   ELSE 0
               END) AS 'vegetarian'
    FROM pizza_runner.customer_orders_temp
    GROUP BY customer_id
    ORDER BY customer_id;
    ```
    
- Results : there are someone who ordered both, and who ordered only meat, who ordered only vegetarian.
    
    
    | customer_id | meat_lovers | vegetarian |
    | --- | --- | --- |
    | 101 | 2 | 1 |
    | 102 | 2 | 1 |
    | 103 | 3 | 1 |
    | 104 | 3 | 0 |
    | 105 | 0 | 1 |

**7. Which pizza was the most popular for each customer?**

- Query : Except 1 person, most loved ‘Meatlovers’ pizza.
    
    ```sql
    WITH order_table AS (
    SELECT customer_id, p.pizza_name, count(*) as ordertime, 
           dense_rank() over(PARTITION BY customer_id ORDER BY count(*) desc) as ranking
    FROM pizza_runner.customer_orders_temp AS c
    	LEFT JOIN pizza_runner.pizza_names AS p ON p.pizza_id = c.pizza_id
    GROUP BY customer_id, p.pizza_name)
    
    SELECT customer_id, pizza_name, ordertime
    FROM order_table
    WHERE ranking = 1;
    ```
    
- Results :
    
    
    | customer_id | pizza_name | ordertime |
    | --- | --- | --- |
    | 101 | Meatlovers | 2 |
    | 102 | Meatlovers | 2 |
    | 103 | Meatlovers | 3 |
    | 104 | Meatlovers | 3 |
    | 105 | Vegetarian | 1 |

**8. What was the maximum number of pizzas delivered in a single order?**

- Query :
    
    ```sql
    SELECT customer_id,
           order_id,
           count(order_id) AS pizza_count
    FROM pizza_runner.customer_orders_temp
    GROUP BY order_id
    ORDER BY pizza_count DESC
    LIMIT 1;
    ```
    
- Results :  The person,  who ordered the most , ordered 3 pizza at one time.
    
    
    | customer_id | order_id | pizza_count |
    | --- | --- | --- |
    | 103 | 4 | 3 |

**9. What was the total volume of pizzas ordered for each hour of the day?**

- Query :
    
    ```sql
    SELECT hour(order_time) AS 'Hour',
           count(order_id) AS 'n_order',
           round(100*count(order_id) /sum(count(order_id)) over(), 2) AS 'ratio'
    FROM pizza_runner.customer_orders_temp
    GROUP BY 1
    ORDER BY 1;
    ```
    
- Results : We can find the hourly pattern when consumer usually order pizza.
    
    
    | Hour | n_order | ratio |
    | --- | --- | --- |
    | 11 | 1 | 7.14 |
    | 13 | 3 | 21.43 |
    | 18 | 3 | 21.43 |
    | 19 | 1 | 7.14 |
    | 21 | 3 | 21.43 |
    | 23 | 3 | 21.43 |

**10. What was the volume of orders for each day of the week?**

- The DAYOFWEEK() function returns the weekday index for a given date ( 1=Sunday, 2=Monday, 3=Tuesday, 4=Wednesday, 5=Thursday, 6=Friday, 7=Saturday )
- DAYNAME() returns the name of the week day
- Query :
    
    ```sql
    SELECT dayname(order_time) AS 'Day Of Week',
           count(order_id) AS 'n_order',
           round(100*count(order_id) /sum(count(order_id)) over(), 2) AS 'ratio'
    FROM pizza_runner.customer_orders_temp
    GROUP BY 1
    ORDER BY 2 DESC;
    ```
    
- Results : Also can find the weekly pattern when consumer usually order pizza.
    
    
    | Day of Week | n_order | ratio |
    | --- | --- | --- |
    | Wednesday | 5 | 35.71 |
    | Saturday | 5 | 21.4335.71 |
    | Thursday | 3 | 21.43 |
    | Friday | 1 | 7.14 |


    

