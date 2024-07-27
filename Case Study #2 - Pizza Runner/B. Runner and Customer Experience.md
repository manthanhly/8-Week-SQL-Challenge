### B. Runner and Customer Experience 

**1. How many runners signed up for each 1 week period? (i.e. week starts <code>2021-01-01</code>)**
````sql
select to_char(date_trunc('week',registration_date) + interval '4 days', 'yyyy-mm-dd') as week, -- adding 4 days because the start of the week is 2020-12-28 
	count(runner_id) as runner
from runners
group by date_trunc('week',registration_date) + interval '4 days'
order by date_trunc('week',registration_date) + interval '4 days'
````
Answer:
| week      | runner |
|-----------|--------|
| 2021-01-01 |      2 |
| 2021-01-08 |      1 |
| 2021-01-15 |      1 |

***

**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**
````sql
select ro.runner_id,
	round(avg(extract (epoch from ro.pickup_time::timestamp - co.order_time::timestamp)/60),0) as diff
from customer_orders co join runner_orders ro on co.order_id = ro.order_id
where ro.pickup_time <> 'null'
group by ro.runner_id
order by ro.runner_id
````
Answer:
| runner_id | diff |
|-----------|------|
|         1 |   16 |
|         2 |   24 |
|         3 |   10 |

***

**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**
````sql
with pizzacountvstime as (
select co.order_id,
	count(co.pizza_id) as count_pizza,
	round(max(extract(epoch from ro.pickup_time::timestamp - co.order_time::timestamp)/60),0) as prepare_time
from customer_orders co inner join runner_orders ro on co.order_id = ro.order_id
where ro.pickup_time <> 'null'
group by co.order_id
)

select count_pizza,
	round(avg(prepare_time),2) as avg
from pizzacountvstime
group by count_pizza
order by count_pizza
````
Answer:
| count_pizza | avg  |
|-------------|------|
|           1 | 12.20 |
|           2 | 18.50 |
|           3 | 29.00 |

***

**4. What was the average distance travelled for each customer?**
- Will need to remove unit in distance column.

````sql
update runner_orders
set distance = regexp_replace(distance, '\s?km', '')
````
- Also need to update data type from varchat to float and replace 'null' values with null

````sql
update runner_orders
set distance = ''
where distance = 'null'

alter table runner_orders
alter column distance type float
using nullif(distance,'')::float
````
- Now, I can work on the question.

````sql
select co.customer_id,
	round(avg(ro.distance)::numeric,2) as average_distance -- round function does not work with float value so we need to convert it to numeric 
from customer_orders co
join runner_orders ro on co.order_id = ro.order_id
where distance is not null
group by co.customer_id
order by co.customer_id
````
Answer:
| customer_id | average_distance |
|-------------|------------------|
|         101 |            20.00 |
|         102 |            16.73 |
|         103 |            23.40 |
|         104 |            10.00 |
|         105 |            25.00 |

***

**5. What was the difference between the longest and shortest delivery times for all orders?**
- We will also need to edit the duration column: take out unit, replace 'null' with null and update datatype to integer.

````sql
update runner_orders
set duration = REGEXP_REPLACE(duration, '\s?min', '')

update runner_orders
set duration = ''
where duration = 'null'

alter table runner_orders
alter column duration type integer
using nullif(duration,'')::integer
````

- Now, I can work on the question.
````sql
select (max(duration) - min(duration)) as maxmin_diff
from customer_orders co
join runner_orders ro on co.order_id = ro.order_id
where duration is not null
````
Anwer:
| maxmin_diff |
|-------------|
|          30 |

***

**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**
````sql
select ro.runner_id,
	co.order_id,
	round(avg(ro.distance/ro.duration*60)::numeric,2)  as speed_kmperhr
from customer_orders co
join runner_orders ro
on co.order_id = ro.order_id
where duration is not null
group by ro.runner_id, 
	co.order_id
order by ro.runner_id, 
	speed_kmperhr
````
Answer:
| runner_id | order_id | speed_kmperhr |
|-----------|----------|---------------|
|         1 |        1 |        37.50  |
|         1 |        3 |        40.20  |
|         1 |        2 |        44.44  |
|         1 |       10 |        60.00  |
|         2 |        4 |        35.10  |
|         2 |        7 |        60.00  |
|         2 |        8 |        93.60  |
|         3 |        5 |        40.00  |

***

**7. What is the successful delivery percentage for each runner?**
````sql
select runner_id,
	round(sum(case when distance is null or duration is null then 0 else 1 end)*100/count(*),2) as percentage
from runner_orders
group by runner_id
order by runner_id
````
Answer:
| runner_id | percentage |
|-----------|------------|
|         1 |    100.00  |
|         2 |     75.00  |
|         3 |     50.00  |


