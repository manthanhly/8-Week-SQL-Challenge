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

***

**6. What was the maximum number of pizzas delivered in a single order?**

````sql
select 
  co.order_id, 
  COUNT(co.order_id) as number_of_orders
from 
  customer_orders co 
join 
  runner_orders ro 
on 
  co.order_id = ro.order_id 
where 
  pickup_time <> 'null'
group by 
  co.order_id
order by 
  COUNT(co.order_id) desc 
limit 
  1
````
Answer:
| order_id | number_of_orders |
|----------|------------------|
|        4 |                3 |

***

**7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

````sql
select 
  co.customer_id,
  count(case 
	when (co.exclusions not in ('', 'null') and co.exclusions is not null) 
	or (co.extras not in ('', 'null') and co.extras is not null) then 1
  end) as total_change_count,
  count(case 
	when (co.exclusions in ('', 'null') or co.exclusions is null) 
	and (co.extras in ('', 'null') or co.extras is null) then 1
  end) as total_nochange_count
from 
  customer_orders co 
join 
  runner_orders ro 
on 
  co.order_id = ro.order_id 
where 
  ro.pickup_time is not null and ro.pickup_time <> 'null'
group by 
  co.customer_id
order by 
  co.customer_id
````
Answer:
| customer_id | total_change_count | total_nochange_count |
|-------------|--------------------|----------------------|
|         101 |                  0 |                    2 |
|         102 |                  0 |                    3 |
|         103 |                  3 |                    0 |
|         104 |                  2 |                    1 |
|         105 |                  1 |                    0 |

***

**8. How many pizzas were delivered that had both exclusions and extras?**

````sql
select 
  sum(case 
	when (co.exclusions not in ('', 'null') and co.exclusions is not null) 
	and (co.extras not in ('', 'null') and co.extras is not null) then 1 
	end) as both_exclusions_extras_count
from 
  customer_orders co 
join 
  runner_orders ro 
on 
  co.order_id = ro.order_id 
where 
  ro.pickup_time is not null and ro.pickup_time <> 'null'
````
Answer:
| both_exclusions_extras_count |
|-----------------------------|
|                           1 |

***

**9. What was the total volume of pizzas ordered for each hour of the day?**

````sql
select 
  extract(hour from order_time) as hour,
  count(*) as orders_per_hour
from 
 customer_orders
group by 
  hour 
order by 
  hour
````
Answer:
| hour | orders_per_hour |
|------|-----------------|
|   11 |               1 |
|   13 |               3 |
|   18 |               3 |
|   19 |               1 |
|   21 |               3 |
|   23 |               3 |

***

**10. What was the volume of orders for each day of the week?**

````sql
select
  to_char(order_time, 'day') as day_of_week,
  count(*) as orders_volume
from 
  customer_orders
group by 
  to_char(order_time, 'day')
````
Answer:
| day_of_week | orders_volume |
|-------------|---------------|
|   saturday  |             3 |
|   friday    |             5 |
|   sunday    |             1 |
|   monday    |             5 |
