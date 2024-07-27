### B. Data Analysis Questions

**1. How many customers has Foodie-Fi ever had?**
````sql
select count(distinct customer_id) as total_customers
from subscriptions
````
Answer:
| total_customers |
|-----------------|
|           1000  |

***

**2. What is the monthly distribution of <code>trial</code> plan <code>start_date</code> values for our dataset - use the start of the month as the group by value**
````sql
select date_trunc('month', s.start_date)::date as start_of_month,
	count(*)
from subscriptions s
left join plans p on s.plan_id = p.plan_id
where p.plan_name = 'trial'
group by date_trunc('month', s.start_date)::date
order by date_trunc('month', s.start_date)::date
````
Answer:
| start_of_month | count |
|----------------|-------|
|    2020-01-01 |    88 |
|    2020-02-01 |    68 |
|    2020-03-01 |    94 |
|    2020-04-01 |    81 |
|    2020-05-01 |    88 |
|    2020-06-01 |    79 |
|    2020-07-01 |    89 |
|    2020-08-01 |    88 |
|    2020-09-01 |    87 |
|    2020-10-01 |    79 |
|    2020-11-01 |    75 |
|    2020-12-01 |    84 |

***

**3. What plan <code>start_date</code> values occur after the year 2020 for our dataset? Show the breakdown by count of events for each <code>plan_name</code>**
````sql
with after_2020 as (
select p.plan_name,
	date_trunc('month', s.start_date)::date as start_of_month
from subscriptions s
left join plans p on s.plan_id = p.plan_id
where extract (year from s.start_date) > 2020
order by p.plan_name
)

select plan_name,
	count(plan_name) as plans_count
from after_2020
group by plan_name
order by plan_name
````
Answer:
| plan_name    | plans_count |
|--------------|-------------|
| basic monthly|          8  |
| churn        |         71  |
| pro annual   |         63  |
| pro monthly  |         60  |

***


**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**
````sql
select count(p.plan_name) as churn_count,
	round(count(p.plan_name)*100/(select count(distinct customer_id) from subscriptions)::numeric,1) as churn_percentage
from subscriptions s
left join plans p on s.plan_id = p.plan_id
where p.plan_name = 'churn'
group by plan_name
````
Answer:
| churn_count | churn_percentage |
|-------------|------------------|
|         307 |            30.7  |

***

**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**
````sql
with order_table as (
select s.customer_id,
	s.plan_id,
	p.plan_name,
	s.start_date,
	row_number () over (partition by s.customer_id order by s.start_date) as orders
from subscriptions s
left join plans p on s.plan_id = p.plan_id
)
select count(plan_name) as churn_count,
	count(plan_name)*100/(select count(distinct customer_id) from subscriptions) as churn_percentage
from order_table
where plan_name = 'churn' and orders = 2
````
Answer:
| churn_count | churn_percentage |
|-------------|------------------|
|          92 |                9 |

***

**6. What is the number and percentage of customer plans after their initial free trial?**
````sql
with order_table as (
select s.customer_id,
	s.plan_id,
	p.plan_name,
	s.start_date,
	row_number () over (partition by s.customer_id order by s.start_date) as orders
from subscriptions s
left join plans p on s.plan_id = p.plan_id
)
select plan_name,
	count(plan_name) as counts,
	round(count(plan_name)*100/(select count(distinct customer_id)::numeric from subscriptions),1) as churn_percentage
from order_table
where orders = 2
group by plan_name
order by plan_name
````
Answer:
| plan_name    | counts | churn_percentage |
|--------------|--------|------------------|
| basic monthly|   546  |            54.6  |
| churn        |    92  |             9.2  |
| pro annual   |    37  |             3.7  |
| pro monthly  |   325  |            32.5  |

***

**7. What is the customer count and percentage breakdown of all 5 <code>plan_name</code> values at <code>2020-12-31?</code>**
````sql
with order_table as (
select s.customer_id,
	s.plan_id,
	p.plan_name,
	s.start_date,
	row_number () over (partition by s.customer_id order by s.start_date desc) as orders
from subscriptions s
left join plans p on s.plan_id = p.plan_id
where s.start_date <= '2020-12-31'
)

select plan_name,
	count(plan_name) as counts,
	round(count(plan_name)*100/(select count(distinct customer_id)::numeric from subscriptions),1) as churn_percentage
from order_table
where orders = 1
group by plan_name
order by plan_name
````
Answer:
| plan_name    | counts | churn_percentage |
|--------------|--------|------------------|
| basic monthly|   224  |            22.4  |
| churn        |   236  |            23.6  |
| pro annual   |   195  |            19.5  |
| pro monthly  |   326  |            32.6  |
| trial        |    19  |             1.9  |

***

**8. How many customers have upgraded to an annual plan in 2020?**
````sql
select count(s.customer_id) as total_customer
from subscriptions s
left join plans p on s.plan_id = p.plan_id
where s.start_date <= '2020-12-31' and p.plan_name like '%annual%'
````
Answer:
| total_customer |
|----------------|
|           195  |

***

**9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?**
````sql
with trial_table as (
select s.customer_id,
	p.plan_name,
	s.start_date
from subscriptions s
left join plans p on s.plan_id = p.plan_id
where p.plan_name = 'trial'
order by s.customer_id
),

annual_table as (
select s.customer_id,
	p.plan_name,
	s.start_date
from subscriptions s
left join plans p on s.plan_id = p.plan_id
where p.plan_name like '%annual%'
order by s.customer_id
)

select
	round(avg(at.start_date - tt.start_date),2) as days_average_to_get_annual
from trial_table tt
inner join annual_table at on tt.customer_id = at.customer_id -- using inner join to get customers with annual plan
````
Answer:
| days_average_to_get_annual |
|----------------------------|
|                    104.62  |

***

**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)**
````sql
with trial_table as (
select s.customer_id,
	p.plan_name,
	s.start_date
from subscriptions s
left join plans p on s.plan_id = p.plan_id
where p.plan_name = 'trial'
order by s.customer_id
),

annual_table as (
select s.customer_id,
	p.plan_name,
	s.start_date
from subscriptions s
left join plans p on s.plan_id = p.plan_id
where p.plan_name like '%annual%'
order by s.customer_id
),

combined_table as (
select tt.customer_id,
	at.start_date - tt.start_date as days_to_get_annual
from trial_table tt
inner join annual_table at on tt.customer_id = at.customer_id -- using inner join to get customers with annual plan
),

period_table as (
select customer_id,
	days_to_get_annual,
	case
	   when days_to_get_annual between 0 and 30 then '0-30'
       when days_to_get_annual between 31 and 60 then '31-60'
       when days_to_get_annual between 61 and 90 then '61-90'
       when days_to_get_annual between 91 and 120 then '91-120'
       when days_to_get_annual between 121 and 150 then '121-150'
       when days_to_get_annual between 151 and 180 then '151-180'
       when days_to_get_annual between 181 and 210 then '181-210'
       when days_to_get_annual between 211 and 240 then '211-240'
       when days_to_get_annual between 241 and 270 then '241-270'
       when days_to_get_annual between 271 and 300 then '271-300'
       when days_to_get_annual between 301 and 330 then '301-330'
       when days_to_get_annual between 331 and 360 then '331-360'
      	else '361+'
	end as periods
from combined_table
)

select periods,
	round(avg(days_to_get_annual),2) as average_per_period
from period_table
group by periods
order by periods
````
Answer:
| periods | average_per_period |
|---------|---------------------|
| 0-30    |               9.96 |
| 31-60   |              42.33 |
| 61-90   |              71.44 |
| 91-120  |             100.69 |
| 121-150 |             133.36 |
| 151-180 |             162.06 |
| 181-210 |             190.73 |
| 211-240 |             224.25 |
| 241-270 |             257.20 |
| 271-300 |             285.00 |
| 301-330 |             327.00 |
| 331-360 |             346.00 |

***

**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**
````sql
with basic_table as (
select s.customer_id,
	p.plan_name,
	s.start_date
from subscriptions s
left join plans p on s.plan_id = p.plan_id
where p.plan_name = 'basic monthly'
order by s.customer_id
),

pro_table as (
select s.customer_id,
	p.plan_name,
	s.start_date
from subscriptions s
left join plans p on s.plan_id = p.plan_id
where p.plan_name = 'pro monthly'
order by s.customer_id
)

select *
from basic_table bt
inner join pro_table pt on bt.customer_id = pt.customer_id
where bt.start_date > pt.start_date and extract (year from bt.start_date) = 2020
````
Answer: 0 customer 
| customer_id | plan_name | start_date |
|-------------|-----------|------------|
|      ...    |    ...    |    ...     |
|      ...    |    ...    |    ...     |

