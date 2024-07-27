### C. Ingredient Optimisation

**1. What are the standard ingredients for each pizza?**
````sql
with new_pizza_recipes as (
select pizza_id,
	unnest(string_to_array(toppings, ','))::integer as topping
from pizza_recipes
)

select 
	pt.topping_name,
	count(distinct npr.pizza_id) as pizzas_count
from new_pizza_recipes npr
join pizza_toppings pt on npr.topping = pt.topping_id
group by pt.topping_name
having count(distinct npr.pizza_id) = 2
````
Answer:
| topping_name | pizzas_count |
|--------------|--------------|
| Mushrooms        |            2 |
| Cheese    |            2 |

***

**2. What was the most commonly added extra?**
````sql
with extra_orders as (
select unnest(string_to_array(co.extras, ', '))::integer as extra
from customer_orders co
where co.extras is not null AND co.extras <> '' AND co.extras <> 'null'
)

select pt.topping_name,
	count(pt.topping_name) as extra_count
from extra_orders ea
join pizza_toppings pt on ea.extra = pt.topping_id
group by pt.topping_name
order by count(pt.topping_name) desc
````
Answer:
| topping_name | extra_count |
|--------------|-------------|
| Bacon        |          4  |
| Chicken      |          1  |
| Cheese       |          1  |

***

**3. What was the most common exclusion?**
````sql
with exclusions_orders as (
select unnest(string_to_array(co.exclusions, ', '))::integer as exclusions
from customer_orders co
where co.exclusions is not null AND co.exclusions <> '' AND co.exclusions <> 'null'
)
select pt.topping_name,
	count(pt.topping_name) as exclusions_count
from exclusions_orders ea
join pizza_toppings pt on ea.exclusions = pt.topping_id
group by pt.topping_name
order by count(pt.topping_name) desc
````
Answer:
| topping_name | exclusions_count |
|--------------|------------------|
| Cheese       |                4 |
| Mushrooms    |                1 |
| BBQ Sauce    |                1 |

***

**4. Generate an order item for each record in the <code>customers_orders</code> table in the format of one of the following:**

- <code>Meat Lovers</code>
- <code>Meat Lovers - Exclude Beef</code>
- <code>Meat Lovers - Extra Bacon</code>
- <code>Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers</code>

````sql
with results as (
select co.order_id,
	co.pizza_id,
	pn.pizza_name,
	case
		when co.exclusions is null or co.exclusions = '' or co.exclusions = 'null'
		then 0
		else unnested_exclusions.exclusions ::integer
	end as exclusions,
	case
		when co.extras is null or co.extras = '' or co.extras = 'null'
		then 0
		else unnested_extras.extras ::integer
	end as extras
from customer_orders co
left join lateral (
	select unnest(string_to_array(co.exclusions, ', ')) as exclusions
	) as unnested_exclusions on true
left join lateral (
	select unnest(string_to_array(co.extras, ', ')) as extras
	) as unnested_extras on true
left join pizza_names pn on co.pizza_id=pn.pizza_id
order by co.order_id
),

status_table as (
select distinct r.order_id,
	pt.topping_name,
	case
		when r.extras = pt.topping_id then 'Extra'
		when r.exclusions = pt.topping_id then 'Exclude'
		else null
	end as status
from results r
join pizza_toppings pt on r.exclusions = pt.topping_id  or r.extras = pt.topping_id
),

new_table as (
select distinct r.order_id,
	r.pizza_id,
    r.pizza_name,
    r.exclusions,
    r.extras,
    st.status,
    st.topping_name
from results r
left join status_table st on r.order_id = st.order_id
)
select distinct order_id,
	case
    	when exclusions = 0 and extras = 0 then pizza_name
        else concat(
        	pizza_name,
            case
               when exclusions <> 0 then ' - Exclude ' || string_agg(case when status = 'Exclude' then topping_name end, ', ')
               else ''
            end,
            case
               when extras <> 0 then ' - Extra ' || string_agg(case when status = 'Extra' then topping_name end, ', ')
               else ''
         end
    )
	end as combined
from new_table
group by order_id, 
	exclusions, 
	extras, 
	pizza_name
order by order_id
````
Answer:
| order_id | combined                                                        |
|----------|-----------------------------------------------------------------|
|        1 | Meatlovers                                                      |
|        2 | Meatlovers                                                      |
|        3 | Meatlovers                                                      |
|        3 | Vegetarian                                                      |
|        4 | Meatlovers - Exclude Cheese                                     |
|        4 | Vegetarian - Exclude Cheese                                     |
|        5 | Meatlovers - Extra Bacon                                        |
|        6 | Vegetarian                                                      |
|        7 | Vegetarian - Extra Bacon                                        |
|        8 | Meatlovers                                                      |
|        9 | Meatlovers - Exclude Cheese - Extra Bacon, Chicken              |
|       10 | Meatlovers                                                      |
|       10 | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |

***

**5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the <code>customer_orders</code> table and add a <code>2x</code> in front of any relevant ingredients**
- For example: <code>"Meat Lovers: 2xBacon, Beef, ... , Salami"</code>

````sql
-- find exclusions table
with exclude_table as (
select co.order_id,
	co.pizza_id,
	co.exclusions,
	pt.topping_id,
	pt.topping_name
from customer_orders co
join pizza_toppings pt on pt.topping_id = any(string_to_array(co.exclusions, ', ')::int[])
where length(exclusions) > 0 and exclusions != 'null'
order by co.order_id),

-- find extra table 
extra_table as (
select co.order_id,
	co.pizza_id,
	co.extras,
	pt.topping_id,
	pt.topping_name
from customer_orders co
join pizza_toppings pt on pt.topping_id = any(string_to_array(co.extras, ', ')::int[])
where length(extras) > 0 and extras != 'null'
order by co.order_id
),

-- the main order with no exclusions and extras
main_orders as (
select co.order_id,
  co.pizza_id,
  pt.topping_id,
  pt.topping_name
from customer_orders co
inner join pizza_recipes pr ON pr.pizza_id = co.pizza_id
cross join lateral unnest(string_to_array(pr.toppings, ', ')) as t(topping)
inner join pizza_toppings pt on pt.topping_id = t.topping::int
order by co.order_id, co.pizza_id, pt.topping_id
),

orders_with_exclusions_extras as (
-- 1st part is main_orders after exclusions
select mo.order_id,
	mo.pizza_id,
	mo.topping_id,
	mo.topping_name
from main_orders mo
left join exclude_table et on mo.order_id = et.order_id and mo.pizza_id = et.pizza_id and mo.topping_id = et.topping_id
where et.topping_id is null -- this is to take out the exclusions
union all

-- recall the extra table to merge/union with the main_orders after exclusions

select order_id,
	pizza_id,
	topping_id,
	topping_name
from extra_table
),

-- find pizza_name and counts the ingredients
pizza_name_ingredient_counts as (
select order_id,
	pn.pizza_name,
	topping_name,
	count(topping_id) as counts
from orders_with_exclusions_extras as owee
inner join pizza_names pn on pn.pizza_id = owee.pizza_id
group by order_id,
	pn.pizza_name,
	topping_name
order by order_id,
	pn.pizza_name,
	topping_name
)

select
  order_id,
  concat(pizza_name,': ', string_agg(topping, ', ')) AS combined_name -- include the pizza_name
from (
  select -- using counts to find the amount of each ingredients
      order_id,
      pizza_name,
      case
          when counts > 1 THEN counts || 'x' || topping_name
          else topping_name
      end as topping
  from
      pizza_name_ingredient_counts
  order by
      order_id,
      pizza_name,
      topping_name -- sort by names of ingredient
) subquery
group by order_id,
  pizza_name
````
Answer:
| order_id | combined_name                                                                          |
|----------|---------------------------------------------------------------------------------------|
|        1 | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami      |
|        2 | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami      |
|        3 | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami      |
|        3 | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                 |
|        4 | Meatlovers: 2xBacon, 2xBBQ Sauce, 2xBeef, 2xChicken, 2xMushrooms, 2xPepperoni, 2xSalami|
|        4 | Vegetarian: Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                         |
|        5 | Meatlovers: 2xBacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
|        6 | Vegetarian: Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                 |
|        7 | Vegetarian: Bacon, Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes          |
|        8 | Meatlovers: Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami      |
|        9 | Meatlovers: 2xBacon, BBQ Sauce, Beef, 2xChicken, Mushrooms, Pepperoni, Salami          |
|       10 | Meatlovers: 3xBacon, 2xBeef, 3xCheese, 2xChicken, 2xPepperoni, 2xSalami                |

***

**6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?**
````sql
-- find exclusions table
with exclude_table as (
select co.order_id,
	co.pizza_id,
	co.exclusions,
	pt.topping_id,
	pt.topping_name
from customer_orders co
join pizza_toppings pt on pt.topping_id = any(string_to_array(co.exclusions, ', ')::int[])
where length(exclusions) > 0 and exclusions != 'null'
order by co.order_id
),

-- find extra table 
extra_table as (
select co.order_id,
	co.pizza_id,
	co.extras,
	pt.topping_id,
	pt.topping_name
from customer_orders co
join pizza_toppings pt on pt.topping_id = any(string_to_array(co.extras, ', ')::int[])
where length(extras) > 0 and extras != 'null'
order by co.order_id
),

-- the main order with no exclusions and extras
main_orders as (
select co.order_id,
  co.pizza_id,
  pt.topping_id,
  pt.topping_name
from customer_orders co
inner join pizza_recipes pr ON pr.pizza_id = co.pizza_id
cross join lateral unnest(string_to_array(pr.toppings, ', ')) as t(topping)
inner join pizza_toppings pt on pt.topping_id = t.topping::int
order by co.order_id, co.pizza_id, pt.topping_id
),

orders_with_exclusions_extras as (
-- 1st part is main_orders after exclusions
select mo.order_id,
	mo.pizza_id,
	mo.topping_id,
	mo.topping_name
from main_orders mo
left join exclude_table et on mo.order_id = et.order_id and mo.pizza_id = et.pizza_id and mo.topping_id = et.topping_id
where et.topping_id is null -- this is to take out the exclusions
union all
-- recall the extra table to merge/union with the main_orders after exclusions
select order_id,
	pizza_id,
	topping_id,
	topping_name
from extra_table
)
-- find total usage per ingredient for all orders
select topping_name,
	count(topping_id) as total_quantity
from orders_with_exclusions_extras as owee
inner join pizza_names pn on pn.pizza_id = owee.pizza_id
group by topping_name
order by count(topping_id) desc
````
Answer:
| topping_name | total_quantity |
|--------------|----------------|
| Bacon        |            14 |
| Mushrooms    |            12 |
| Chicken      |            11 |
| Cheese       |            11 |
| Pepperoni    |            10 |
| Salami       |            10 |
| Beef         |            10 |
| BBQ Sauce    |             8 |
| Tomato Sauce |             4 |
| Onions       |             4 |
| Tomatoes     |             4 |
| Peppers      |             4 |





