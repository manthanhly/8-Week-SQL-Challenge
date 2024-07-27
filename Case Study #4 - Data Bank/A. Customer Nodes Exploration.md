### A. Customer Nodes Exploration

**1. How many unique nodes are there on the Data Bank system?**
````sql
select count(distinct node_id) as unique_nodes
from customer_nodes
````
Answer:
| unique_nodes |
|--------------|
|           5  |

***

**2. What is the number of nodes per region?**
````sql
select region_id,
	count(distinct node_id) as number_of_nodes
from customer_nodes
group by region_id
````
Answer:
| region_id | number_of_nodes |
|-----------|-----------------|
|         1 |               5 |
|         2 |               5 |
|         3 |               5 |
|         4 |               5 |
|         5 |               5 |

***

**3. How many customers are allocated to each region?**
````sql
select region_id,
	count(customer_id) as customer_count,
	count(distinct customer_id) as unique_customer_count
from customer_nodes
group by region_id
````
Answer:
region_id|customer_count|unique_customer_count|
---------+--------------+---------------------+
        1|           770|                  110|
        2|           735|                  105|
        3|           714|                  102|
        4|           665|                   95|
        5|           616|                   88|
        
***

**4. How many days on average are customers reallocated to a different node?**
````sql
with change_table as (
select customer_id,
	node_id,
	sum(end_date - start_date) as days_changed
from customer_nodes
where end_date != '9999-12-31' -- exclude the most recent change 
group by customer_id,
	node_id
)

select
	round(avg(days_changed),1) as averaged_days_changed
from change_table
````
Answer:
| averaged_days_changed |
|-----------------------|
|                  23.6 |

***

**5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?**
````sql
with change_table as (
select customer_id,
	node_id,
	region_id,
	sum(end_date - start_date) as days_changed
from customer_nodes
where end_date != '9999-12-31'
group by customer_id,
	node_id,
	region_id
order by customer_id
)

select r.region_name,
   percentile_cont(0.5) within group (order by days_changed) as median,
   round(percentile_cont(0.8) within group (order by days_changed)::numeric,1) as percentile_80,
   round(percentile_cont(0.95) within group (order by days_changed)::numeric,1) as percentile_95
from change_table ct join regions r
on ct.region_id = r.region_id
group by r.region_name
````
Answer:
| region_name | median | percentile_80 | percentile_95 |
|-------------|--------|---------------|---------------|
| Africa      |  22.0  |         35.0  |         54.0  |
| America     |  22.0  |         34.0  |         53.7  |
| Asia        |  22.0  |         34.6  |         52.0  |
| Australia   |  21.0  |         34.0  |         51.0  |
| Europe      |  23.0  |         34.0  |         51.4  |

