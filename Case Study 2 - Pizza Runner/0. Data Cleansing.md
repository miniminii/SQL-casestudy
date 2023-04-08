## Case Study #2 - Pizza Runner 🍕
## 0. DATA CLEANSING

This is solution for case study2. (written by Yumin Kim)

### `customer_orders` table
- The exclusions and extras columns in customer_orders table will need to be cleaned up before using them in the queries
- (BEFROE)

| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
| -------- | ----------- | -------- | ---------- | ------ | ------------------- |
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3        | 102         | 2        |            |        | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5        | 104         | 1        | null       | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        | null       | null   | 2020-01-08 21:03:13 |
| 7        | 105         | 2        | null       | 1      | 2020-01-08 21:20:29 |
| 8        | 102         | 1        | null       | null   | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10       | 104         | 1        | null       | null   | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |

- In the exclusions and extras columns, there are blank spaces and null values.
- (AFTER)
```sql
DROP TABLE IF EXISTS customer_orders_temp;

CREATE TEMPORARY TABLE customer_orders_temp AS
SELECT order_id,
       customer_id,
       pizza_id,
       CASE
           WHEN exclusions = '' THEN NULL
           WHEN exclusions = 'null' THEN NULL
           ELSE exclusions
       END AS exclusions,
       CASE
           WHEN extras = '' THEN NULL
           WHEN extras = 'null' THEN NULL
           ELSE extras
       END AS extras,
       order_time
FROM pizza_runner.customer_orders;

SELECT * FROM customer_orders_temp;
```

| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
| -------- | ----------- | -------- | ---------- | ------ | ------------------- |
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3        | 102         | 2        |            |        | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5        | 104         | 1        |            | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        |            |        | 2020-01-08 21:03:13 |
| 7        | 105         | 2        |            | 1      | 2020-01-08 21:20:29 |
| 8        | 102         | 1        |            |        | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10       | 104         | 1        |            |        | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |

- 빈칸들을 다 null값으로 대체하는 과정
---
<br>

### `runner_orders` table
- The pickup_time, distance, duration and cancellation columns in runner_orders table will need to be cleaned up before using them in the queries
- (BEFORE)

| order_id | runner_id | pickup_time         | distance | duration   | cancellation            |
| -------- | --------- | ------------------- | -------- | ---------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20km     | 32 minutes |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20km     | 27 minutes |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4km   | 20 mins    |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40         |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15         |                         |
| 6        | 3         | null                | null     | null       | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25km     | 25mins     | null                    |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4 km  | 15 minute  | null                    |
| 9        | 2         | null                | null     | null       | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10km     | 10minutes  | null                    |

- In the pickup_time column, there are null values.
- In the distance column, there are null values. It contains unit - km. The 'km' must also be stripped
- In the duration column, there are null values. The 'minutes', 'mins' 'minute' must be stripped
- In the cancellation column, there are blank spaces and null values.

```sql
DROP TABLE IF EXISTS runner_orders_temp;

CREATE TEMPORARY TABLE runner_orders_temp AS

SELECT order_id,
       runner_id,
       CASE
           WHEN pickup_time LIKE 'null' THEN NULL
           ELSE pickup_time
       END AS pickup_time,
       CASE
           WHEN distance LIKE 'null' THEN NULL
           ELSE CAST(regexp_replace(distance, '[a-z]+', '') AS FLOAT)
       END AS distance,
       CASE
           WHEN duration LIKE 'null' THEN NULL
           ELSE CAST(regexp_replace(duration, '[a-z]+', '') AS FLOAT)
       END AS duration,
       CASE
           WHEN cancellation LIKE '' THEN NULL
           WHEN cancellation LIKE 'null' THEN NULL
           ELSE cancellation
       END AS cancellation
FROM pizza_runner.runner_orders;

SELECT * FROM runner_orders_temp;
```
| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
| -------- | --------- | ------------------- | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4     | 20       |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |                         |
| 6        | 3         |                     |          |          | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       |                         |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |                         |
| 9        | 2         |                     |          |          | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       |                         |
- null이라는 text가 들어가있으면 NULL로 취급
- 칼럼값안에 km, mins, minutes같은 단위 text가 들어있으면 공백으로 대체 : regexp_replace(변수명, 데이터형태, 바꾸려는 값)
- 데이터 타입 변경해주려면 CAST(변수 AS type명)


---
<br>

## [추가] Expanding the comma seperated string into rows

### `pizza recipe` table
- The toppings column in the pizza_recipes table is a comma separated string.
- (BEFORE)

| pizza_id | toppings                |
| -------- | ----------------------- |
| 1        | 1, 2, 3, 4, 5, 6, 8, 10 |
| 2        | 4, 6, 7, 9, 11, 12      |

> Method : Using JSON table functions
- JSON functions are used to split the comma separated string into multiple rows.
- `json_array()` converts the string to a JSON array
- We enclose array elements with double quotes, this is performed using the replace function and we trim the resultant array

```sql
SELECT *,
       json_array(toppings),
       replace(json_array(toppings), ',', '","'),
       trim(replace(json_array(toppings), ',', '","'))
FROM pizza_runner.pizza_recipes;
```
| pizza_id | toppings                | json_array(toppings)        | replace(json_array(toppings), ',', '","') | trim(replace(json_array(toppings), ',', '","')) |
| -------- | ----------------------- | --------------------------- | ----------------------------------------- | ----------------------------------------------- |
| 1        | 1, 2, 3, 4, 5, 6, 8, 10 | ["1, 2, 3, 4, 5, 6, 8, 10"] | ["1"," 2"," 3"," 4"," 5"," 6"," 8"," 10"] | ["1"," 2"," 3"," 4"," 5"," 6"," 8"," 10"]       |
| 2        | 4, 6, 7, 9, 11, 12      | ["4, 6, 7, 9, 11, 12"]      | ["4"," 6"," 7"," 9"," 11"," 12"]          | ["4"," 6"," 7"," 9"," 11"," 12"]                |

- We convert the json data into a tabular data using `json_table()`.
- Syntax: JSON_TABLE(expr, path COLUMNS (column_list) [AS] alias)
- It extracts data from a JSON document and returns it as a relational table having the specified columns
- Each match for the path preceding the COLUMNS keyword maps to an individual row in the result table.

```sql
SELECT t.pizza_id, (j.topping)
FROM pizza_runner.pizza_recipes t
JOIN json_table(trim(replace(json_array(t.toppings), ',', '","')), '$[*]' columns (topping varchar(50) PATH '$')) j ;
```
| pizza_id | topping |
| -------- | ------- |
| 1        | 1       |
| 1        |  2      |
| 1        |  3      |
| 1        |  4      |
| 1        |  5      |
| 1        |  6      |
| 1        |  8      |
| 1        |  10     |
| 2        | 4       |
| 2        |  6      |
| 2        |  7      |
| 2        |  9      |
| 2        |  11     |
| 2        |  12     |
