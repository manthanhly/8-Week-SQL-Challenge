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

***

**4. What is the average discount value per transaction?**

***

**5. What is the percentage split of all transactions for members vs non-members?**

***

**6. What is the average revenue for member transactions and non-member transactions?**
