### Product Analysis

**1. What are the top 3 products by total revenue before discount?**
````sql
select
	pd.product_name,
	sum(s.qty*s.price) as revenue
from 
	sales s 
join 
	product_details pd on s.prod_id = pd.product_id 
group by 	
	pd.aproduct_name 
order by 
	sum(s.qty*s.price) desc 
limit 	
	3
````
Answer:
| product_name                 | revenue |
|------------------------------|---------|
| Blue Polo Shirt - Mens       |  217683 |
| Grey Fashion Jacket - Womens |  209304 |
| White Tee Shirt - Mens       |  152000 |

***

**2. What is the total quantity, revenue and discount for each segment?**
````sql
select
	pd.segment_name,
	sum(s.qty) as quantity,
	sum(s.qty*s.price) as revenue, -- assuming revenue is prior to discount 
	sum(s.qty*s.price*s.discount/100) as discount
from 
	sales s 
join 
	product_details pd on s.prod_id = pd.product_id 
group by 	
	pd.segment_name
````
Answer:
| segment_name | quantity | revenue | discount |
|--------------|----------|---------|----------|
| Shirt        |    11265 |  406143 |    48082 |
| Jeans        |    11349 |  208350 |    23673 |
| Jacket       |    11385 |  366983 |    42451 |
| Socks        |    11217 |  307977 |    35280 |

***

**3. What is the top selling product for each segment?**
````sql
with total_quantity as (
select
	pd.segment_name,
	pd.product_name,
	sum(s.qty) as total_sale -- assuming top selling in term of product quantity 
from 
	sales s 
join 
	product_details pd on s.prod_id = pd.product_id 
group by 	
	pd.segment_name,
	pd.product_name
)
,rank_table as (
select 
	segment_name,
	product_name,
	total_sale,
	row_number () over (partition by segment_name order by total_sale desc) as rank -- ranking by total_sale for each segment
from 
	total_quantity
)

select 
	segment_name,
	product_name,
	total_sale
from
	rank_table
where 
	rank = 1
````
Answer:
| segment_name | product_name                  | total_sale |
|--------------|--------------------------------|------------|
| Jacket       | Grey Fashion Jacket - Womens  |       3876 |
| Jeans        | Navy Oversized Jeans - Womens |       3856 |
| Shirt        | Blue Polo Shirt - Mens        |       3819 |
| Socks        | Navy Solid Socks - Mens       |       3792 |

***

**4. What is the total quantity, revenue and discount for each category?**
````sql
select
	pd.category_name,
	sum(s.qty) as quantity,
	sum(s.qty*s.price) as revenue, -- assuming revenue is prior to discount 
	sum(s.qty*s.price*s.discount/100) as discount
from 
	sales s 
join 
	product_details pd on s.prod_id = pd.product_id 
group by 	
	pd.category_name
````
Answer:
| category_name | quantity | revenue | discount |
|---------------|----------|---------|----------|
| Mens          |    22482 |  714120 |    83362 |
| Womens        |    22734 |  575333 |    66124 |

***

**5. What is the top selling product for each category?**
````sql
with total_quantity as (
select
	pd.category_name,
	pd.product_name,
	sum(s.qty) as total_sale -- assuming top selling in term of product quantity 
from 
	sales s 
join 
	product_details pd on s.prod_id = pd.product_id 
group by 	
	pd.category_name,
	pd.product_name
)
,rank_table as (
select 
	category_name,
	product_name,
	total_sale,
	row_number () over (partition by category_name order by total_sale desc) as rank -- ranking by total_sale for each category
from 
	total_quantity
)

select 
	category_name,
	product_name,
	total_sale
from
	rank_table
where 
	rank = 1
````
Answer:
| category_name | product_name                 | total_sale |
|---------------|------------------------------|------------|
| Mens          | Blue Polo Shirt - Mens       |       3819 |
| Womens        | Grey Fashion Jacket - Womens |       3876 |

***

**6. What is the percentage split of revenue by product for each segment?**
````sql
with revenue_product as (
select
	pd.segment_name,
	pd.product_name,
	sum(s.qty*s.price) as revenue
from 
	sales s 
join 
	product_details pd on s.prod_id = pd.product_id 
group by 	
	pd.segment_name,
	pd.product_name
),
revenue_segment as (
select 
	pd.segment_name,
	sum(s.qty*s.price) as revenue
from 
	sales s 
join 
	product_details pd on s.prod_id = pd.product_id 
group by 	
	pd.segment_name
)

select 
	rp.segment_name,
	rp.product_name,
	round(rp.revenue*100/rs.revenue::numeric,2) as percetange 
from 
	revenue_product rp
join 
	revenue_segment rs on rp.segment_name = rs.segment_name
order by 
	rp.segment_name
````
Answer:
| segment_name | product_name                     | percentage |
|--------------|-----------------------------------|------------|
| Jacket       | Indigo Rain Jacket - Womens      |      19.45 |
| Jacket       | Khaki Suit Jacket - Womens       |      23.51 |
| Jacket       | Grey Fashion Jacket - Womens     |      57.03 |
| Jeans        | Navy Oversized Jeans - Womens    |      24.06 |
| Jeans        | Black Straight Jeans - Womens    |      58.15 |
| Jeans        | Cream Relaxed Jeans - Womens     |      17.79 |
| Shirt        | White Tee Shirt - Mens           |      37.43 |
| Shirt        | Blue Polo Shirt - Mens           |      53.60 |
| Shirt        | Teal Button Up Shirt - Mens      |       8.98 |
| Socks        | Navy Solid Socks - Mens          |      44.33 |
| Socks        | White Striped Socks - Mens       |      20.18 |
| Socks        | Pink Fluro Polkadot Socks - Mens |      35.50 |

***

**7. What is the percentage split of revenue by segment for each category?**
````sql
with revenue_segment as (
select
	pd.category_name,
	pd.segment_name,
	sum(s.qty*s.price) as revenue
from 
	sales s 
join 
	product_details pd on s.prod_id = pd.product_id 
group by 	
	pd.category_name,
	pd.segment_name
),
revenue_category as (
select 
	pd.category_name,
	sum(s.qty*s.price) as revenue
from 
	sales s 
join 
	product_details pd on s.prod_id = pd.product_id 
group by 	
	pd.category_name
)

select 
	rs.category_name,
	rs.segment_name,
	round(rs.revenue*100/rc.revenue::numeric,2) as percetange 
from 
	revenue_segment rs
join 
	revenue_category rc on rs.category_name = rc.category_name
order by 
	rs.category_name
````
Answer:
| category_name | segment_name | percentage |
|---------------|--------------|------------|
| Mens          | Socks        |      43.13 |
| Mens          | Shirt        |      56.87 |
| Womens        | Jeans        |      36.21 |
| Womens        | Jacket       |      63.79 |

***

**8. What is the percentage split of total revenue by category?**
````sql
with revenues as (
select 
	pd.category_name,
	sum(s.qty*s.price) as revenue
from 
	sales s 
join 
	product_details pd on s.prod_id = pd.product_id 
group by 	
	pd.category_name
)

, revenue_total as (
select 
	sum(revenue) as total_revenue
from 
	revenues
)

select 
	r.category_name,
	round(revenue*100/total_revenue::numeric,2) as percentage
from
	revenues r,
	revenue_total rt
````
Answer:
| category_name | percentage |
|---------------|------------|
| Mens          |      55.38 |
| Womens        |      44.62 |

***

**9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)**
````sql
with transactions_table as (
select 
	pd.product_name,
	count(distinct txn_id) as transactions
from 
	sales s 
join 
	product_details pd on s.prod_id = pd.product_id 
where
	s.qty > 0
group by 	
	pd.product_name
)

, total_transactions_table as (
select 
	count(distinct txn_id) as total_transactions
from
	sales
)

select 
	tt.product_name,
	round(tt.transactions::numeric/ttt.total_transactions,3) as penetration
from
	transactions_table tt,
	total_transactions_table ttt
````
Answer:
| product_name                     | penetration |
|----------------------------------|-------------|
| Black Straight Jeans - Womens    |       0.498 |
| Blue Polo Shirt - Mens           |       0.507 |
| Cream Relaxed Jeans - Womens     |       0.497 |
| Grey Fashion Jacket - Womens     |       0.510 |
| Indigo Rain Jacket - Womens      |       0.500 |
| Khaki Suit Jacket - Womens       |       0.499 |
| Navy Oversized Jeans - Womens    |       0.510 |
| Navy Solid Socks - Mens          |       0.512 |
| Pink Fluro Polkadot Socks - Mens |       0.503 |
| Teal Button Up Shirt - Mens      |       0.497 |
| White Striped Socks - Mens       |       0.497 |
| White Tee Shirt - Mens           |       0.507 |

***

**10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?**

