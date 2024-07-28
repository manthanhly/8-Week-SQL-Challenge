### B. Customer Transactions

**1. What is the unique count and total amount for each transaction type?**
````sql
select distinct txn_type,
	count(txn_type) as unique_count,
	sum(txn_amount) as total_amount
from customer_transactions
group by txn_type
````
Answer:
| txn_type  | unique_count | total_amount |
|-----------|--------------|--------------|
| deposit   |        2671  |    1,359,168 |
| purchase  |        1617  |      806,537 |
| withdrawal|        1580  |      793,003 |

***

**2. What is the average total historical deposit counts and amounts for all customers?**
````sql
-- CTE to get average deposit and deposit count per customer 
with table_1 as (
select customer_id,
	count(*) as deposit_count,
	avg (txn_amount) as avg_deposit_per_customer
from customer_transactions
where txn_type = 'deposit'
group by customer_id
)

select round(avg(deposit_count),0) as average_deposit_count,
	round(avg(avg_deposit_per_customer),2) as average_deposit
from table_1
````
Answer:
| average_deposit_count | average_deposit |
|-----------------------|-----------------|
|                     5 |        508.61   |

***

**3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**
````sql
with count_table as (
select customer_id,
	extract (month from date_trunc('month',txn_date)) as month_number, -- getting month number
	sum(case when txn_type = 'deposit' then 1 else 0 end) as deposit, -- count the deposit and non-deposit for each customer of each month 
	sum(case when txn_type != 'deposit' then 1 else 0 end) as non_deposit
from customer_transactions
group by extract (month from date_trunc('month',txn_date)), customer_id
having sum(case when txn_type = 'deposit' then 1 else 0 end) > 1
	and sum(case when txn_type != 'deposit' then 1 else 0 end) = 1
order by customer_id
)

select month_number,
	count(customer_id) as customers_count
from count_table
group by month_number
````
Answer:
| month_number | customers_count |
|--------------|-----------------|
|            1 |              53 |
|            2 |              36 |
|            3 |              38 |
|            4 |              22 |

***

**4. What is the closing balance for each customer at the end of the month?**
````sql
-- get total amout of each month and assume that deposit is positive, anything else is negative 
with sum_table as (
select customer_id,
	(date_trunc('month',txn_date) + interval '1 month' - interval '1 day')::date as month_end,
	sum(case
		when txn_type = 'deposit' then txn_amount else txn_amount*(-1)
		end) as total_amount
from customer_transactions
group by customer_id, date_trunc('month',txn_date) + interval '1 month' - interval '1 day'
order by customer_id, date_trunc('month',txn_date) + interval '1 month' - interval '1 day'
)

select customer_id,
	month_end,
	total_amount,
	sum(total_amount) over (partition by customer_id order by month_end) as end_of_month_sum -- getting moving sum 
from sum_table
````
Answer: Showing only 3 customers 
| customer_id | month_end | total_amount | end_of_month_sum |
|-------------|-----------|--------------|------------------|
|          1  | 2020-01-31|         312  |              312 |
|          1  | 2020-03-31|        -952  |             -640 |
|          2  | 2020-01-31|         549  |              549 |
|          2  | 2020-03-31|          61  |              610 |
|          3  | 2020-01-31|         144  |              144 |
|          3  | 2020-02-29|        -965  |             -821 |
|          3  | 2020-03-31|        -401  |            -1222 |
|          3  | 2020-04-30|         493  |             -729 |
|          ...  | ...|         ...  |             ... |
***

**5. What is the percentage of customers who increase their closing balance by more than 5%?**

- Scenario 1 - back to back month changes & repetive customers who have more than 1 time of increasing their balance by more than 5%
````sql
with sum_table as (
select customer_id,
	txn_date,
	(date_trunc('month',txn_date) + interval '1 month' - interval '1 day')::date as month_end,
	sum(case
		when txn_type = 'deposit' then txn_amount else txn_amount*(-1)
		end) as total_amount
from customer_transactions
group by customer_id, date_trunc('month',txn_date) + interval '1 month' - interval '1 day', txn_date
order by customer_id, date_trunc('month',txn_date) + interval '1 month' - interval '1 day'
),

balance_table as (
select customer_id,
	month_end,
	txn_date,
	(date_trunc('month',txn_date) - interval '1 day')::date as previous_month_end,
	sum(total_amount) over (partition by customer_id order by txn_date) as month_closing,
	row_number () over (partition by customer_id, month_end order by txn_date desc) as orders
from sum_table
),

compare_table as (
select bt1.customer_id,
	bt1.month_end,
	bt1.month_closing as current_month_balance,
	bt2.month_closing as next_month_balance
from balance_table bt1 join balance_table bt2
on bt1.customer_id = bt2.customer_id and bt1.month_end = bt2.previous_month_end
where bt1.orders = 1 and bt2.orders = 1 and bt1.month_closing != 0
),

percentage_table as (
select customer_id,
	(next_month_balance/current_month_balance - 1)*100 as changed_percentage,
	case when (current_month_balance < next_month_balance) and
		(next_month_balance/current_month_balance - 1)*100 > 5 then 1 else 0 end as over_5percent_count
from compare_table
)

select round(sum(over_5percent_count)/count(over_5percent_count)::numeric*100,2) as percentage_of_customers
from percentage_table
````
Answer:
| percentage_of_customers |
|-------------------------|
|                  20.97  |

