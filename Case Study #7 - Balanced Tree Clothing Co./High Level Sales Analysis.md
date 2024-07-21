### High Level Sales Analysis

**1. What was the total quantity sold for all products?**
````sql
select 
	pd.product_name,
	sum(s.qty) as total_qty
from 
	sales s
join
	product_details pd on s.prod_id = pd.product_id 
group by 
	pd.product_name 
````
Answer:
| product_name                   | total_qty |
|----------------------------------|-----------|
| White Tee Shirt - Mens           |      3800 |
| Navy Solid Socks - Mens          |      3792 |
| Grey Fashion Jacket - Womens     |      3876 |
| Navy Oversized Jeans - Womens    |      3856 |
| Pink Fluro Polkadot Socks - Mens |      3770 |
| Khaki Suit Jacket - Womens       |      3752 |
| Black Straight Jeans - Womens    |      3786 |
| White Striped Socks - Mens       |      3655 |
| Blue Polo Shirt - Mens           |      3819 |
| Indigo Rain Jacket - Womens      |      3757 |
| Cream Relaxed Jeans - Womens     |      3707 |
| Teal Button Up Shirt - Mens      |      3646 |

***

**2. What is the total generated revenue for all products before discounts?**
````sql
select 
	pd.product_name,
	sum(s.qty*s.price) as total_revenue_before_discount
from 
	sales s
join
	product_details pd on s.prod_id = pd.product_id 
group by 
	pd.product_name 
````
Answer:
| product_name            | total_revenue_before_discount |
|----------------------------------|-------------------------------|
| White Tee Shirt - Mens           |                       152000  |
| Navy Solid Socks - Mens          |                       136512  |
| Grey Fashion Jacket - Womens     |                       209304  |
| Navy Oversized Jeans - Womens    |                        50128  |
| Pink Fluro Polkadot Socks - Mens |                       109330  |
| Khaki Suit Jacket - Womens       |                        86296  |
| Black Straight Jeans - Womens    |                       121152  |
| White Striped Socks - Mens       |                        62135  |
| Blue Polo Shirt - Mens           |                       217683  |
| Indigo Rain Jacket - Womens      |                        71383  |
| Cream Relaxed Jeans - Womens     |                        37070  |
| Teal Button Up Shirt - Mens      |                        36460  |

***

**3. What was the total discount amount for all products?**
````sql
select 
	pd.product_name,
	sum(s.qty*s.price*s.discount/100) as total_discount
from 
	sales s
join
	product_details pd on s.prod_id = pd.product_id 
group by 
	pd.product_name 
````
Answer:
| product_name                 |total_discount|
|----------------------------------|----------------|
| White Tee Shirt - Mens           |          17968 |
| Navy Solid Socks - Mens          |          16059 |
| Grey Fashion Jacket - Womens     |          24781 |
| Navy Oversized Jeans - Womens    |           5538 |
| Pink Fluro Polkadot Socks - Mens |          12344 |
| Khaki Suit Jacket - Womens       |           9660 |
| Black Straight Jeans - Womens    |          14156 |
| White Striped Socks - Mens       |           6877 |
| Blue Polo Shirt - Mens           |          26189 |
| Indigo Rain Jacket - Womens      |           8010 |
| Cream Relaxed Jeans - Womens     |           3979 |
| Teal Button Up Shirt - Mens      |           3925 |

