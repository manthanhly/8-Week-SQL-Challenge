### Transaction Analysis

**1. How many unique transactions were there?**
````sql
select 
	count(distinct txn_id) as unique_transactions_count
from 
	sales
````
Answer:
| unique_transactions_count |
|---------------------------|
|                      2500 |

***

**2. What is the average unique products purchased in each transaction?**
````sql
with sum_table as (
select 
	txn_id,
	sum(qty) as total_products -- each transaction already has unique products id, so we get the total qty for each transaction
from 
	sales
group by
	txn_id
)

select 
	round(avg(total_products),1) as average_products_purchased
from 
	sum_table
````
Answer:
| average_products_purchased |
|----------------------------|
|                        18.1 |

***

**3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?**
- Pre-Discount
````sql
with sum_table as (
select 
	txn_id,
	sum(qty*price) as revenue
from 
	sales
group by
	txn_id
)

select 
	percentile_cont(0.25) within group(order by revenue) as percentile_25th,
	percentile_cont(0.5) within group(order by revenue) as percentile_50th,
	percentile_cont(0.75) within group(order by revenue) as percentile_75th
from 
	sum_table
````
Answer:
| percentile_25th | percentile_50th | percentile_75th |
|-----------------|-----------------|-----------------|
|          375.75 |           509.5 |           647.0 |

- Post-Discount
````sql
with sum_table as (
select 
	txn_id,
	sum(qty*price*(100-discount)/100) as revenue
from 
	sales
group by
	txn_id
)

select 
	percentile_cont(0.25) within group(order by revenue) as percentile_25th,
	percentile_cont(0.5) within group(order by revenue) as percentile_50th,
	percentile_cont(0.75) within group(order by revenue) as percentile_75th
from 
	sum_table
````
Answer:
| percentile_25th | percentile_50th | percentile_75th |
|-----------------|-----------------|-----------------|
|          323.75 |           438.0 |          570.25 |

***

**4. What is the average discount value per transaction?**
````sql
with sum_table as (
select 
	txn_id,
	sum(qty*price*discount/100) as discount_value
from 
	sales
group by
	txn_id
)

select 
	round(avg(discount_value),1) as avg_discount
from 
	sum_table
````
Answer:
| avg_discount |
|--------------|
|         59.8 |

***

**5. What is the percentage split of all transactions for members vs non-members?**
````sql
with count_table as (
select 
	member,
	count(distinct txn_id) as counts -- getting unique transactions to get correct count 
from 
	sales
group by
	member
),

sum_table as (
select 
	sum(counts) as total_counts
from 
	count_table
)

select 
	ct.member,
	round(ct.counts/st.total_counts,2) as perc
from 
	count_table ct, sum_table st
````
Answer:
| member | perc |
|--------|------|
| false  | 0.40 |
| true   | 0.60 |

***

**6. What is the average revenue for member transactions and non-member transactions?**
````sql
with count_table as (
select 
	member,
	txn_id,
	sum(qty*price) as revenue_per_transaction
from 
	sales
group by
	member,
	txn_id
)
select
	member,
	round(avg(revenue_per_transaction),2) as avg_revenue
from 
	count_table
group by
	member
````
Answer:
| member | avg_revenue |
|--------|-------------|
| false  |      515.04 |
| true   |      516.27 |

