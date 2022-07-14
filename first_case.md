# First case 
Case Study #1 - Danny's Diner(https://8weeksqlchallenge.com/case-study-1/).

==========================================================================

### 1. What is the total amount each customer spent at the restaurant?

`SELECT  sales.customer_id,sum(menu.price) as "total spent $"
  FROM (dannys_diner.menu as menu join dannys_diner.sales as sales on menu.product_id=sales.product_id ) 
GROUP BY (sales.customer_id) ORDER BY(sales.customer_id);`

--------------------------------------------------------------------------

### 2. How many days has each customer visited the restaurant?

`SELECT sales.customer_id, COUNT(DISTINCT sales.order_date)as "visits" from dannys_diner.sales as sales GROUP BY (sales.customer_id) ORDER BY(sales.customer_id);`

--------------------------------------------------------------------------

### 3. What was the first item from the menu purchased by each customer?

`WITH added_row_number AS (
  SELECT
    *,
    ROW_NUMBER() OVER(PARTITION BY sales.customer_id ORDER BY sales.order_date ASC) AS row_number
  FROM dannys_diner.sales as sales
)
SELECT
  customer_id,product_name
FROM added_row_number as s join dannys_diner.menu as menu on s.product_id = menu.product_id
WHERE row_number = 1;
`

--------------------------------------------------------------------------

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

`select product_name, count(product_name) as "times_purchased" from dannys_diner.sales as sales join dannys_diner.menu as menu 
on sales.product_id = menu.product_id 
group by (product_name) 
order by (times_purchased) desc 
limit 1;`

--------------------------------------------------------------------------

### 5. Which item was the most popular for each customer?

`WITH fav_item_cte AS
(
 SELECT s.customer_id, m.product_name, 
  COUNT(m.product_id) AS order_count,
  DENSE_RANK() OVER(PARTITION BY s.customer_id
  ORDER BY COUNT(s.customer_id) DESC) AS rank
FROM dannys_diner.menu AS m
JOIN dannys_diner.sales AS s
 ON m.product_id = s.product_id
GROUP BY s.customer_id, m.product_name
)SELECT customer_id, product_name, order_count
FROM fav_item_cte 
WHERE rank = 1;`

--------------------------------------------------------------------------

### 6. Which item was purchased first by the customer after they became a member?

`WITH first_item_after_membership AS
(
select members.customer_id, sales.order_date, menu.product_name,
  DENSE_RANK() OVER(PARTITION BY members.customer_id
  ORDER BY sales.order_date ASC) AS rank
from (dannys_diner.members as members join dannys_diner.sales as sales on members.customer_id = sales.customer_id and members.join_date <= sales.order_date)join dannys_diner.menu as menu on menu.product_id = sales.product_id
  group by members.customer_id,sales.order_date, menu.product_name
)select customer_id,order_date,product_name
from first_item_after_membership
where rank=1;`

--------------------------------------------------------------------------

### 7. Which item was purchased just before the customer became a member?

`WITH total_spent_and_count_before_membership AS
(
select members.customer_id, sales.order_date, menu.product_name,
  DENSE_RANK() OVER(PARTITION BY members.customer_id
  ORDER BY sales.order_date DESC) AS rank
from (dannys_diner.members as members join dannys_diner.sales as sales on members.customer_id = sales.customer_id and members.join_date > sales.order_date)join dannys_diner.menu as menu on menu.product_id = sales.product_id
  group by members.customer_id,sales.order_date, menu.product_name
)select customer_id,order_date,product_name
from item_before_membership
where rank=1;
`

--------------------------------------------------------------------------

### 8. What is the total items and amount spent for each member before they became a member?

`select members.customer_id, sum(menu.price) as "total_spent",count(distinct sales.product_id) as "total_items"
from (dannys_diner.members as members join dannys_diner.sales as sales on members.customer_id = sales.customer_id and members.join_date > sales.order_date)join dannys_diner.menu as menu on menu.product_id = sales.product_id
group by members.customer_id
order by members.customer_id;`

--------------------------------------------------------------------------

### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

`
WITH price_points AS
 (
 SELECT *, 
 CASE
  WHEN product_id = 1 THEN price * 20
  ELSE price * 10
  END AS points
 FROM dannys_diner.menu
 )
 select sales.customer_id , sum(points)
 from price_points as p join dannys_diner.sales as sales on sales.product_id = p.product_id
 group by sales.customer_id;`

--------------------------------------------------------------------------

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

`
WITH dates_cte AS 
(
 SELECT *, 
 join_date + INTERVAL '6 day' AS valid_date, 
  '2021-01-31' AS last_date
 FROM dannys_diner.members AS m
),
multiple_points as
(SELECT d.customer_id, s.order_date, d.join_date, 
 d.valid_date, d.last_date, m.product_name, m.price,
 SUM(CASE
  WHEN m.product_name = 'sushi' THEN 2 * 10 * m.price
  WHEN s.order_date BETWEEN d.join_date AND d.valid_date THEN 2 * 10 * m.price
  ELSE 10 * m.price
  END) AS points
FROM dates_cte AS d
JOIN dannys_diner.sales AS s
 ON d.customer_id = s.customer_id
JOIN dannys_diner.menu AS m
 ON s.product_id = m.product_id
where s.order_date <'2021-01-31'
GROUP BY d.customer_id, s.order_date, d.join_date, d.valid_date, d.last_date, m.product_name, m.price) 

select m.customer_id,m.join_date,m.valid_date, sum(points)
from multiple_points as m
group by m.customer_id,m.join_date,m.valid_date;`

--------------------------------------------------------------------------

