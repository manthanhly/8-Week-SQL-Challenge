# Case Study #1 - Danny's Diner

<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" width="500" height="500">

## Introduction

Danny’s Diner, a new restaurant specializing in sushi, curry, and ramen, needs help analyzing customer data to improve experiences and guide business decisions. This project involves using sample data to provide insights into customer behavior and spending patterns.

## Questions & Answers

### Case Study Questions

**1. What is the total amount each customer spent at the restaurant?**
````sql
select customer_id, sum(price) as total_amount 
from dannys_diner.sales a
join dannys_diner.menu b on a.product_id = b.product_id
group by customer_id
order by customer_id
````
Answer:
| customer_id | total_amount |
|-------------|--------------|
| A           |           76 |
| B           |           74 |
| C           |           36 |

***

**2. How many days has each customer visited the restaurant?**
````sql
select customer_id, 
  count(distinct order_date) as days_visited
from dannys_diner.sales 
group by customer_id
order by customer_id
````
Answer:
| customer_id | days_visited |
|-------------|--------------|
| A           |            4 |
| B           |            6 |
| C           |            2 |

***

**3. What was the first item from the menu purchased by each customer?**
````sql
with rankings as (
select customer_id, 
	product_name, 
	dense_rank() over (partition by customer_id order by order_date) as ranks 
from dannys_diner.sales a 
join dannys_diner.menu b on a.product_id = b.product_id
 ) 

select distinct customer_id, 
	product_name
from rankings 
where ranks = 1
````
Answer:
| customer_id | product_name |
|-------------|--------------|
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

***

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
````sql
with total as (
select b.product_name, 
	count(a.product_id) as total_purchased 
from dannys_diner.sales a
join dannys_diner.menu b on a.product_id = b.product_id 
group by b.product_name
)

select product_name, 
	total_purchased
from total
where total_purchased = (select max(total_purchased) from total)
````
Answer:
| product_name | total_purchased |
|--------------|-----------------|
| ramen        | 8               |

***

**5. Which item was the most popular for each customer?**
````sql
with ranking as (
select a.customer_id, 
	b.product_name, 
	count(*) as total_order, 
	dense_rank() over (partition by a.customer_id order by count(*) desc) as rank
from dannys_diner.sales a
join dannys_diner.menu b on a.product_id = b.product_id
group by a.customer_id, 
	b.product_name
)
  
select customer_id, 
	product_name, 
	total_order
from ranking
where rank = 1
````
Answer:
| customer_id | product_name | total_order |
|-------------|--------------|-------------|
| A           | ramen        | 3           |
| B           | sushi        | 2           |
| B           | curry        | 2           |
| B           | ramen        | 2           |
| C           | ramen        | 3           |

***

**6. Which item was purchased first by the customer after they became a member?**
````sql
with ranking as (
select 
a.customer_id,
      b.product_name,
      dense_rank() over (partition by a.customer_id order by a.order_date) as rank
from dannys_diner.sales a
join dannys_diner.menu b on a.product_id = b.product_id
join dannys_diner.members c on a.customer_id = c.customer_id
where c.join_date is null or a.order_date >= c.join_date
)

select distinct customer_id, 
	product_name
from ranking
where rank = 1
````
Answer:
| customer_id | product_name |
|-------------|--------------|
| A           | curry        |
| B           | sushi        |

***

**7. Which item was purchased just before the customer became a member?**
````sql
with ranking as (
select a.customer_id,
	b.product_name,
	dense_rank() over (partition by a.customer_id order by a.order_date desc) as rank
from dannys_diner.sales a
join dannys_diner.menu b on a.product_id = b.product_id
join dannys_diner.members c on a.customer_id = c.customer_id
where c.join_date is null or a.order_date < c.join_date
)

select distinct customer_id, 
	product_name
from ranking
where rank = 1
````
Answer:
| customer_id | product_name |
|-------------|--------------|
| A           | curry        |
| A           | sushi        |
| B           | sushi        |

***

**8. What is the total items and amount spent for each member before they became a member?**
````sql
select a.customer_id,
    count(a.customer_id) as total_items,
    sum(b.price) as total_expense
from dannys_diner.sales a
join dannys_diner.menu b on a.product_id = b.product_id
join dannys_diner.members c on a.customer_id = c.customer_id
where a.order_date < c.join_date
group by a.customer_id
order by a.customer_id
````
Answer:
| customer_id | total_items | total_expense |
|-------------|-------------|---------------|
| A           | 2           | 25            |
| B           | 3           | 40            |

***

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
````sql
with points_count as (
select a.customer_id,
	case when b.product_name = 'sushi' then b.price*20 else b.price*10 end as points
from dannys_diner.sales a
join dannys_diner.menu b on a.product_id = b.product_id
)
    
select customer_id, 
	sum(points) as total_points
from points_count
group by customer_id
order by customer_id
````
Answer:
| customer_id | total_points |
|-------------|--------------|
| A           | 860          |
| B           | 940          |
| C           | 360          |

***

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
````sql
with points_count as (
select a.customer_id,
	a.order_date,
    b.product_name,
    b.price,
    case
      	when a.order_date between c.join_date and c.join_date + interval '6 days' then b.price*20 
  		when b.product_name = 'sushi' then b.price*20
  		else b.price*10 
  	end as points 
from dannys_diner.sales a
join dannys_diner.menu b on a.product_id = b.product_id
join dannys_diner.members c on a.customer_id = c.customer_id
where extract(month from a.order_date) = 1
)

select customer_id, 
	sum(points) as total_points
from points_count
group by customer_id
order by customer_id 
````

Answer:
| customer_id | total_points |
|-------------|--------------|
| A           | 1370         |
| B           | 820          |

***
### Bonus Quetions 

**Join All The Things**
````sql
select
	a.customer_id,
  	to_char(a.order_date, 'yyyy-mm-dd') as order_date,
    b.product_name,
    b.price,
    case 
    	when a.order_date < c.join_date then 'n'
        when a.order_date >= c.join_date then 'y'
        else 'n' 
    end as member
from dannys_diner.sales a
join dannys_diner.menu b on a.product_id = b.product_id
left join dannys_diner.members c on a.customer_id = c.customer_id
order by a.customer_id, 
	order_date, 
	price desc 
````
***

**Ranking Of Customer Products**
````sql
with memberstatus as (
select
	a.customer_id,
    to_char(a.order_date, 'yyyy-mm-dd') as order_date,
    b.product_name,
    b.price,
    case 
    	when a.order_date < c.join_date then 'n'
        when a.order_date >= c.join_date then 'y'
        else 'n' 
    end as member
from dannys_diner.sales a
join dannys_diner.menu b on a.product_id = b.product_id
left join dannys_diner.members c on a.customer_id = c.customer_id
)
    
select customer_id,
	order_date,
    product_name,
    price,
    member,
    case 
    	when member = 'y' then dense_rank () over (partition by customer_id, member order by order_date) else null
    end as ranking
from memberstatus
order by customer_id, 
	order_date, 
	price desc;
````
Answer:
| customer_id | order_date | product_name | price | member | ranking |
|-------------|------------|--------------|-------|--------|---------|
| A           | 2021-01-01 | curry        | 15    | n      |         |
| A           | 2021-01-01 | sushi        | 10    | n      |         |
| A           | 2021-01-07 | curry        | 15    | y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | y      | 3       |
| B           | 2021-01-01 | curry        | 15    | n      |         |
| B           | 2021-01-02 | curry        | 15    | n      |         |
| B           | 2021-01-04 | sushi        | 10    | n      |         |
| B           | 2021-01-11 | sushi        | 10    | y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | n      |         |
| C           | 2021-01-01 | ramen        | 12    | n      |         |
| C           | 2021-01-07 | ramen        | 12    | n      |         |
