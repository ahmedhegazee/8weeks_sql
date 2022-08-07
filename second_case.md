# Second case 
Case Study #1 - Danny's Diner(https://8weeksqlchallenge.com/case-study-2/).

==========================================================================

### 1. How many pizzas were ordered?

`select count(pizza_id) as totalOrderedPizza from customer_orders1;`

--------------------------------------------------------------------------

### 2. How many unique customer orders were made?

`select count(distinct order_id) as totalOrders from customer_orders1;`

--------------------------------------------------------------------------

### 3. How many successful orders were delivered by each runner?

`select runner_id, count(order_id) as successfulOrders from runner_orders1 where distance is not null group by runner_id;`

--------------------------------------------------------------------------

### 4. How many of each type of pizza was delivered?

`select p.pizza_name,COUNT(o.order_id) as deliveredPiazza from runner_orders1 r join customer_orders1 o on r.order_id = o.order_id join pizza_names p on p.pizza_id = o.pizza_id
where distance is not null group by p.pizza_name;`

--------------------------------------------------------------------------

### 5. How many Vegetarian and Meatlovers were ordered by each customer?

`select o.customer_id,p.pizza_name,COUNT(o.order_id) as orderedPiazza from customer_orders1 o join pizza_names p on p.pizza_id = o.pizza_id
group by o.customer_id,p.pizza_name
order by o.customer_id;`

--------------------------------------------------------------------------

### 6. What was the maximum number of pizzas delivered in a single order?

`select o.order_id, count(pizza_id) as totalDelivered
from runner_orders1 r join customer_orders1 o on o.order_id=r.order_id 
where distance is not null group by o.order_id
order by totalDelivered desc limit 1;`

--------------------------------------------------------------------------

