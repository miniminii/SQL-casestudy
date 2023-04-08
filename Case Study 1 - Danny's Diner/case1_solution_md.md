## Case Study #1 - Danny's Diner 🍜

This is solution for case study1. (written by Yumin Kim)
<br>

#### 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT S.customer_id , CONCAT('$',sum(M.price)) as total_spent
FROM dannys_diner.sales as S left join dannys_diner.menu as M
     on S.product_id = M.product_id
GROUP BY customer_id
ORDER BY customer_id ;
```
| customer_id | total_spent |
| ----------- | --- |
| A           | $76  |
| B           | $74  |
| C           | $36  |

---

<br>

#### 2. How many days has each customer visited the restaurant?

```sql
SELECT customer_id, count(distinct(order_date)) as visit_days
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```
| customer_id | visit_days |
| ----------- | ---------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

- 하루에 중복해서 order한 경우를 고려해서 distinct개념으로 접근해야 함.

---

<br>

#### 3. What was the first item from the menu purchased by each customer?

```sql
WITH first_order AS(
SELECT S.customer_id, S.order_date, M.product_id, M.product_name, DENSE_RANK() OVER(PARTITION BY S.customer_id 
                  ORDER BY S.order_date) AS rownum
FROM dannys_diner.sales AS S left join dannys_diner.menu AS M 
     ON S.product_id = M.product_id )

SELECT customer_id, order_date, product_name, rownum
FROM first_order
WHERE rownum =1
GROUP BY customer_id, order_date, product_name
```  

| customer_id | order_date | product_name | rownum |
| ----------- | ---------- | ------------ | ------ |
| A           | 2021-01-01 | sushi        | 1      |
| A           | 2021-01-01 | curry        | 1      |
| B           | 2021-01-01 | curry        | 1      |
| C           | 2021-01-01 | ramen        | 1      |



- first_order라는 테이블을 하나 만든 다음에
- DENSE RANK() OVER(PARTITION BY ~ )를 써서 가장 customer_id랑 날짜별로 index번호를 부여함.

<br>

```sql
WITH first_order as (
select s.customer_id, s.order_date, m.product_id, m.product_name
from dannys_diner.sales as s left join dannys_diner.menu as m
     on s.product_id = m.product_id
where (s.customer_id, s.order_date) in (
SELECT customer_id, min(order_date)
FROM dannys_diner.sales
GROUP BY customer_id) )

select customer_id, product_name
from first_order
group by customer_id, product_name
```  


| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | curry        |
| C           | ramen        |


- 또는 DENSE_RANK()쓰지말고 min값에 해당하는 친구를 불러오는 방법도 있음.
- 이때 중복되는 C - RAMEN의 경우에는 group by로 다 묶어주면 중복 없어짐.

<br>

```sql
select customer_id, group_concat(distinct(product_name)) as product_name
from first_order
group by customer_id
```  

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry,sushi  |
| B           | curry        |
| C           | ramen        |

- 만약에 같은 customer_id라면 같은 줄에 보여주게 하고 싶다면 group_concat(distinct(변수명)) 으로 표현 가능

---

<br>


#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
select m.product_name, count(m.product_name) as count
from dannys_diner.sales as s left join dannys_diner.menu as m
     on s.product_id = m.product_id
group by m.product_name
order by count desc
limit 1
```
| product_name | count |
| ------------ | ----- |
| ramen        | 8     |

- max 구할 때 order by > limit 1 로 접근할지 / max함수 써서 접근할지 골라야함

---

<br>

#### 5. Which item was the most popular for each customer?

```sql
WITH table1 as (select customer_id, m.product_name, count(*) as ordertime, dense_rank() over(PARTITION BY customer_id ORDER BY count(*) desc) as ranking
from dannys_diner.sales as s left join dannys_diner.menu as m
     on s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name )

SELECT customer_id, product_name, ordertime
FROM table1
WHERE ranking = 1
```
| customer_id | product_name | ordertime |
| ----------- | ------------ | --------- |
| A           | ramen        | 3         |
| B           | curry        | 2         |
| B           | sushi        | 2         |
| B           | ramen        | 2         |
| C           | ramen        | 3         |

- ranking반영된 table 하나 만든 다음에, 1위에 해당하는거 추출

---

<br>

####  6. Which item was purchased first by the customer after they became a member?

```sql
WITH table1 as (SELECT s.customer_id, s.order_date, s.product_id, menu.product_name, DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) as ranking
FROM dannys_diner.sales as s 
     left join dannys_diner.members as mem
     on s.customer_id = mem.customer_id 
     left join dannys_diner.menu as menu 
     on s.product_id = menu.product_id
where s.order_date >= mem.join_date)

SELECT customer_id, order_date, product_name
FROM table1
WHERE ranking = 1
```

| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-07 | curry        |
| B           | 2021-01-11 | sushi        |

- with는 한 번 밖에 못쓴다는 점 .. 

---

 <br>

#### 7. Which item was purchased just before the customer became a member?
```sql
WITH table1 as (SELECT s.customer_id, s.order_date, s.product_id, menu.product_name, DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date desc) as ranking
FROM dannys_diner.sales as s 
     left join dannys_diner.members as mem
     on s.customer_id = mem.customer_id 
     left join dannys_diner.menu as menu 
     on s.product_id = menu.product_id
where s.order_date < mem.join_date )

SELECT customer_id, order_date, product_name
FROM table1
WHERE ranking = 1
```
| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-01 | sushi        |
| A           | 2021-01-01 | curry        |
| B           | 2021-01-04 | sushi        |

- just before 즉, 바로 직전이니까 order_date의 내림차순으로 접근해야 함.

<br>

```sql
SELECT customer_id, order_date, GROUP_CONCAT(distinct product_name) as product_name 
FROM table1
WHERE ranking = 1
GROUP BY customer_id, order_date
```
| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A           | 2021-01-01 | curry,sushi  |
| B           | 2021-01-04 | sushi        |

- 또는 GROUP_CONCAT으로 한 줄로 표현할 수도 있다.

---

<br>

#### 8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT s.customer_id, count(menu.product_name) as total_amt, concat('$', sum(menu.price)) as total_spt
FROM dannys_diner.sales as s 
     left join dannys_diner.members as mem
     on s.customer_id = mem.customer_id 
     left join dannys_diner.menu as menu 
     on s.product_id = menu.product_id
where s.order_date < mem.join_date
GROUP BY s.customer_id
```
| customer_id | total_amt | total_spt |
| ----------- | --------- | --------- |
| A           | 2         | $25       |
| B           | 3         | $40       |

---

<br>

#### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
SELECT customer_id, GROUP_CONCAT(distinct product_name) as product_name, 
       SUM(CASE WHEN product_name = 'sushi' THEN price*20
                ELSE price*10
                END) AS points
FROM dannys_diner.sales as s left join dannys_diner.menu as m
     on s.product_id = m.product_id
GROUP BY customer_id
```
| customer_id | product_name      | points |
| ----------- | ----------------- | ------ |
| A           | curry,ramen,sushi | 860    |
| B           | curry,ramen,sushi | 940    |
| C           | ramen             | 360    |

- 이름도 한 번에 보이고, 포인트 적립도 같이 보이게 !
- 만약에 멤버십 가입 이후에만 포인트가 적립된다는 가정이 있다면, where로 order_date >= join_date하고 A,B의 포인트 sum구하면 됨.

---

<br>

#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
SELECT s.customer_id,
       sum(CASE WHEN order_date BETWEEN join_date AND DATE_ADD(join_date, INTERVAL 6 DAY) THEN price * 20
            WHEN order_date NOT BETWEEN join_date AND DATE_ADD(join_date, INTERVAL 6 DAY) AND product_name = 'sushi' THEN price*20
            ELSE price * 10 END) as point
FROM dannys_diner.sales as s 
     left join dannys_diner.members as mem
     on s.customer_id = mem.customer_id 
     left join dannys_diner.menu as menu 
     on s.product_id = menu.product_id
WHERE order_date <='2021-01-31'
GROUP BY s.customer_id
```
| customer_id | point |
| ----------- | ----- |
| A           | 1370  |
| B           | 820   |
| C           | 360   |




- 날짜 더하려면 DATE_ADD(날짜, 숫자 INTERVAL 단위) 로 쓰면 됨.
- 멤버십 가입을 안한 사람한테도 포인트를 주고 있다는 전제라면 이렇게 하면 됨. 
- 반대로 멤버십 가입자한테만 하고 싶으면 where절에 order_date >=join_date 넣어주면 됨.

```sql
SELECT s.customer_id,
       SUM(IF(order_date BETWEEN join_date AND DATE_ADD(join_date, INTERVAL 6 DAY), price*10*2, IF(product_name = 'sushi', price*10*2, price*10))) AS customer_points
FROM dannys_diner.menu AS m
INNER JOIN dannys_diner.sales AS s ON m.product_id = s.product_id
INNER JOIN dannys_diner.members AS mem USING (customer_id)
WHERE order_date <='2021-01-31'
GROUP BY s.customer_id
ORDER BY s.customer_id;
```
- if로 접근하면 더 간편하긴 함.
