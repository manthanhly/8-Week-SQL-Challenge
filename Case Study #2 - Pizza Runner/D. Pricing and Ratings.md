### D. Pricing and Ratings

**1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**
````sql
with price_table as (
select ro.runner_id,
	pn.pizza_name,
	case
		when pn.pizza_name = 'Meatlovers' then 12 else 10
	end as price
from runner_orders ro
left join customer_orders co on ro.order_id = co.order_id
left join pizza_names pn on pn.pizza_id = co.pizza_id
where ro.pickup_time != 'null'
)

select runner_id,
	sum(price) as Income
from price_table
group by runner_id
order by runner_id
````
Answer:
| runner_id | income |
|-----------|--------|
|         1 |     70 |
|         2 |     56 |
|         3 |     12 |

***

**2. What if there was an additional $1 charge for any pizza extras?**
- Add cheese is $1 extra
````sql
with price_table as (
select ro.runner_id,
	pn.pizza_name,
	co.extras,
	case
		when pn.pizza_name = 'Meatlovers' then 12 else 10
	end as price,
	case
		when length(extras) > 0 and extras != 'null' then 1 else 0
	end as extras_fee,
	case
		when extras like '%4%' then 1 else 0 -- we know cheese topping_id is 4 from pizza_toppings table
	end as cheese_fee
from runner_orders ro
left join customer_orders co on ro.order_id = co.order_id
left join pizza_names pn on pn.pizza_id = co.pizza_id
where ro.pickup_time != 'null'
order by ro.runner_id)

select runner_id,
	sum(price + extras_fee + cheese_fee) as total_fee
from price_table
group by runner_id
order by runner_id
````
Answer:
| runner_id | total_fee |
|-----------|-----------|
|         1 |        72 |
|         2 |        57 |
|         3 |        13 |

***

**3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.**
````sql
create table customer_ratings (
	rating_id int,
	order_id int,
	customer_id int,
	rating int check (rating between 1 and 5),
	rating_date timestamp default current_timestamp
);

insert into customer_ratings (order_id, customer_id, rating)
values (1, 101, 2),
   (10, 104, 5),
   (7, 105, 3),
   (4, 103, 2),
   (6, 101, 1);
````
***

**4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?**

- <code>customer id</code>
- <code>order_id</code>
- <code>runner_id</code>
- <code>rating</code>
- <code>order_id</code>
- <code>pickup_time</code>
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas

````sql
select co.customer_id,
	co.order_id,
	ro.runner_id,
	cr.rating,
	co.order_time,
	ro.pickup_time,
	max(extract(epoch from ro.pickup_time::timestamp - co.order_time::timestamp)/60) as time_between_order_pickup,
	avg(ro.duration) as duration,
	round(avg(ro.distance/ro.duration*60)::numeric,2)  as avg_speed_kmperhr,
	count(co.order_id) as total_pizza
from customer_orders co
left join runner_orders ro on co.order_id = ro.order_id
left join customer_ratings cr on co.order_id = cr.order_id
where ro.pickup_time != 'null'
group by co.customer_id,
	co.order_id,
	ro.runner_id,
	cr.rating,
	co.order_time,
	ro.pickup_time
order by co.order_id
````
Answer:
| customer_id | order_id | runner_id | rating | order_time             | pickup_time        | time_between_order_pickup | duration           | avg_speed_kmperhr | total_pizza |
|-------------|----------|-----------|--------|------------------------|--------------------|---------------------------|--------------------|------------------|-------------|
|         101 |        1 |         1 |      2 | 2021-01-01 18:05:02.000 | 2021-01-01 18:15:34 |      10.5333333333333333  | 32.0000000000000000 |            37.50 |          1 |
|         101 |        2 |         1 |        | 2021-01-01 19:00:52.000 | 2021-01-01 19:10:54 |      10.0333333333333333  | 27.0000000000000000 |            44.44 |          1 |
|         102 |        3 |         1 |        | 2021-01-02 23:51:23.000 | 2021-01-03 00:12:37 |      21.2333333333333333  | 20.0000000000000000 |            40.20 |          2 |
|         103 |        4 |         2 |      2 | 2021-01-04 13:23:46.000 | 2021-01-04 13:53:03 |      29.2833333333333333  | 40.0000000000000000 |            35.10 |          3 |
|         104 |        5 |         3 |        | 2021-01-08 21:00:29.000 | 2021-01-08 21:10:57 |      10.4666666666666667  | 15.0000000000000000 |            40.00 |          1 |
|         105 |        7 |         2 |      3 | 2021-01-08 21:20:29.000 | 2021-01-08 21:30:45 |      10.2666666666666667  | 25.0000000000000000 |            60.00 |          1 |
|         102 |        8 |         2 |        | 2021-01-09 23:54:33.000 | 2021-01-10 00:15:02 |      20.4833333333333333  | 15.0000000000000000 |            93.60 |          1 |
|         104 |       10 |         1 |      5 | 2021-01-11 18:34:49.000 | 2021-01-11 18:50:20 |      15.5166666666666667  | 10.0000000000000000 |            60.00 |          2 |

***

**5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?**
````sql
with price_table as(
select co.order_id,
	co.customer_id,
	ro.runner_id,
	pn.pizza_name,
	ro.distance,
	case
		when pn.pizza_name = 'Meatlovers' then 12 else 10
	end as price
from customer_orders co
left join runner_orders ro on co.order_id = ro.order_id
left join pizza_names pn on co.pizza_id = pn.pizza_id
where ro.pickup_time != 'null'
order by co.order_id
),

travel_fees as (
select runner_id,
	SUM(distance)*0.3 as travel_fee
from runner_orders
where pickup_time != 'null'
group by runner_id
order by runner_id )

select pt.runner_id,
	round((sum(pt.price) + tf.travel_fee)::numeric,2) as total_fee
from price_table pt join travel_fees tf
on pt.runner_id = tf.runner_id
group by pt.runner_id, tf.travel_fee
order by pt.runner_id
````
Answer:
| runner_id | total_fee |
|-----------|-----------|
|         1 |    89.02  |
|         2 |    77.54  |
|         3 |    15.00  |


