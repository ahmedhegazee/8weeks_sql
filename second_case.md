# Second case 
Case Study #2 - Danny's Diner(https://8weeksqlchallenge.com/case-study-2/).

==========================================================================

## Pizza Metrics

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

### 7. For each customer, how many delivered pizzas had at least 1 change, and how many had no changes?

`select o.customer_id,sum( case when(o.exclusions is not null and exclusions !=0) or (o.extras is not null and o.extras!=0)then 1 else 0 end )as leastOneChange,sum( case when (exclusions is null or exclusions = 0) and (extras is null or extras = 0) then 1 else 0 end )as noChange from customer_orders1 o join runner_orders1 r on o.order_id = r.order_id where r.distance is not null GROUP BY o.customer_id;`

--------------------------------------------------------------------------

### 8. How many pizzas were delivered that had both exclusions and extras?

`select o.customer_id,
sum( case when(o.exclusions is not null and exclusions !=0) AND (o.extras is not null and o.extras!=0)then 1 else 0 end )as BothExtasAndExclusions
 from customer_orders1 o join runner_orders1 r on o.order_id = r.order_id
where r.distance is not null
GROUP BY o.customer_id;`

--------------------------------------------------------------------------

### 9. What was the total volume of pizzas ordered for each hour of the day?

`select extract(hour from order_time) as horlyOrder , count(order_id) as TotalOrders
from customer_orders1
group by horlyOrder
order by horlyOrder;`

--------------------------------------------------------------------------

### 10. What was the volume of orders for each day of the week?

`select dayname(order_time) as dailyOrders , count(order_id) as TotalOrders
from customer_orders1
group by dailyOrders
order by dailyOrders;`

==========================================================================

## Runner and Customer Experience

### 1. How many runners signed up for each 1 week period?

`select week(registration_date) as weeklyRegister , count(runner_id) as totalRunners
from runners 
group by weeklyRegister;`

--------------------------------------------------------------------------

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pick up the order?

`select runner_id, round(avg(TIMESTAMPDIFF(MINUTE,order_time,pickup_time)),1) as AvgTime
from runner_orders1 r join customer_orders1 o on r.order_id=o.order_id
where distance is not null;`

--------------------------------------------------------------------------

### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

`with cte as (
select count(o.order_id) as totalPizza ,round((timestampdiff(minute, order_time, pickup_time))) as AvgTime
from customer_orders1 o join runner_orders1 r on  o.order_id=r.order_id
where r.distance is not null
group by o.order_id,pickup_time,order_time
)
select totalPizza, AvgTime from cte group by totalPizza,AvgTime;
`

--------------------------------------------------------------------------

### 4. What was the average distance travelled for each customer?

`select customer_id,round(avg(distance),1) as AvgDistance from runner_orders1 r join customer_orders1 o on r.order_id = o.order_id
group by customer_id;`

--------------------------------------------------------------------------

### 5. What was the difference between the longest and shortest delivery times for all orders?

`select max(timediff) - min(timediff) as diffrence from (
select TIMESTAMPDIFF(minute,order_time,pickup_time) as timediff from runner_orders1 r join customer_orders1 o on r.order_id = o.order_id where distance is not null
group by o.order_id,order_time,pickup_time) as t`

--------------------------------------------------------------------------

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

`select runner_id,order_id, round(distance *60 /duration,1) as speedKMH from runner_orders1 
where distance is not null
group by runner_id,order_id,distance,duration order by runner_id`

--------------------------------------------------------------------------

### 7. What is the successful delivery percentage for each runner?

`select runner_id,round((deliveredOrders/totalOrders)*100,1) as percentage from (select runner_id,sum(
case when(distance !=0 AND distance is not null) then 1 else 0 end
) as deliveredOrders,
 count(runner_id) as totalOrders
from runner_orders1 
group by runner_id)as t;`

==========================================================================

## Ingredient Optimisation

### 1. What are the standard ingredients for each pizza?

`select pizza_name, group_concat(topping_name) as StandardToppings  from pizza_recipes1 r join pizza_toppings t on r.toppings = t.topping_id join pizza_names p on p.pizza_id = r.pizza_id
group by pizza_name;`

--------------------------------------------------------------------------
