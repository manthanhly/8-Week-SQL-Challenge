# Case Study #2: Pizza_Runner

<img src="https://github.com/user-attachments/assets/880dba8f-c3ea-4deb-8825-260b93c5acfa" width="500" height="500">

## Q&A 

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

**3. How many unique customer orders were made?**

````sql
select
  runner_id,
  count(*) as unique_customer_orders
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
| runner_id | unique_customer_orders|
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


