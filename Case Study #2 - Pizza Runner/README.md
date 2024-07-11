# Case Study #2: Pizza_Runner

<img src="https://github.com/user-attachments/assets/880dba8f-c3ea-4deb-8825-260b93c5acfa" width="500" height="500">

## Questions & Answers

### A. Pizza Metrics

**1. How many pizzas were ordered?**

````sql
select
  count(*) as total_ordered
from
  customer_orders
````
Answer: 
| total_ordered |
| ------------- |
|        14     |

***

**2. How many unique customer orders were made?**

````sql
select
  count(distinct order_id) as unique_customer_count 
from 
  customer_orders
````

Answer:
| unique_customer_count |
| ------------- |
|        10     |

***

**3. How many successful orders were delivered by each runner?**

````sql
select
  runner_id,
  count(*) as orders_count
from
  runner_orders 
where
  pickup_time <> 'null'
group by
  runner_id 
order by
  runner_id
````

Answer:
| runner_id | orders_count|
|-----------|-------|
|         1 |     4 |
|         2 |     3 |
|         3 |     1 |

***

**4. How many of each type of pizza was delivered?**

````sql
select
  pizza_id,
  count(pizza_id) as pizza_count
from
  customer_orders co
join
  runner_orders ro
on
  co.order_id = ro.order_id 
where
  pickup_time <> 'null'
group by
  pizza_id 
order by
  pizza_id
````

Answer:
| pizza_id | pizza_count |
|----------|-------------|
|        1 |           9 |
|        2 |           3 |

***

**5. How many Vegetarian and Meatlovers were ordered by each customer?**

````sql
select 
  co.customer_id, 
  pn.pizza_name, 
  count(co.order_id) as pizza_count
from
  customer_orders co 
join
  pizza_names pn
on
  co.pizza_id = pn.pizza_id 
group by
  co.customer_id,
  pn.pizza_name
order by
  co.customer_id
````
Answer:
| customer_id | pizza_name | pizza_count |
|-------------|------------|-------------|
|         101 | Meatlovers |           2 |
|         101 | Vegetarian |           1 |
|         102 | Meatlovers |           2 |
|         102 | Vegetarian |           1 |
|         103 | Meatlovers |           3 |
|         103 | Vegetarian |           1 |
|         104 | Meatlovers |           3 |
|         105 | Vegetarian |           1 |
