### Segment Analysis

**1. Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year**

Top 10 interests:
````sql
with months_table as (
select interest_id,
	count(*) as total_months -- counting ids appearance 
from interest_metrics
group by interest_id
)

, filtered_table as (
select interest_id
from months_table
where total_months >= 6
)
, order_table as (
select im.interest_id,
	im.month_year,
	im.composition,
	row_number () over (partition by im.interest_id order by im.composition desc) as orders -- assign the max composition of each interest to be number 1
from interest_metrics im 
join filtered_table ft
on im.interest_id = ft.interest_id
)

select 
	interest_id,
	month_year,
	composition as top_composition
from order_table
where orders = 1
order by composition desc
limit 10
````
Answer:

| interest_id | month_year | top_composition |
|-------------|------------|-----------------|
|       21057 | 2018-12-01 |            21.2 |
|        6284 | 2018-07-01 |           18.82 |
|          39 | 2018-07-01 |           17.44 |
|          77 | 2018-07-01 |           17.19 |
|       12133 | 2018-10-01 |           15.15 |
|        5969 | 2018-12-01 |           15.05 |
|         171 | 2018-07-01 |           14.91 |
|        4898 | 2018-07-01 |           14.23 |
|        6286 | 2018-07-01 |            14.1 |
|           4 | 2018-07-01 |           13.97 |

Bottom 10 interests:
````sql
with months_table as (
select interest_id,
	count(*) as total_months -- counting ids appearance 
from interest_metrics
group by interest_id
)

, filtered_table as (
select interest_id
from months_table
where total_months >= 6
)
, order_table as (
select im.interest_id,
	im.month_year,
	im.composition,
	row_number () over (partition by im.interest_id order by im.composition) as orders -- removed desc to find the smallest value for each interest 
from interest_metrics im 
join filtered_table ft
on im.interest_id = ft.interest_id
)

select 
	interest_id,
	month_year,
	composition as top_composition
from order_table
where orders = 1
order by composition -- removed desc to find the smallest value of all interest 
limit 10
````
Answer:
| interest_id | month_year | top_composition |
|-------------|------------|-----------------|
|       45524 | 2019-05-01 |            1.51 |
|       44449 | 2019-04-01 |            1.52 |
|       39336 | 2019-05-01 |            1.52 |
|       34083 | 2019-06-01 |            1.52 |
|        4918 | 2019-05-01 |            1.52 |
|       20768 | 2019-05-01 |            1.52 |
|       35742 | 2019-06-01 |            1.52 |
|        6127 | 2019-05-01 |            1.53 |
|       36877 | 2019-05-01 |            1.53 |
|        6314 | 2019-06-01 |            1.53 |


***

**2. Which 5 interests had the lowest average ranking value?**
````sql
with months_table as (
select interest_id,
	count(*) as total_months -- counting ids appearance 
from interest_metrics
group by interest_id
)

, filtered_table as (
select interest_id
from months_table
where total_months >= 6
)

select im.interest_id,
	round(avg(im.ranking),2) as average_ranking
from interest_metrics im 
join filtered_table ft
on im.interest_id = ft.interest_id
group by im.interest_id 
order by avg(im.ranking) 
limit 5
````
Answer:
| interest_id | average_ranking |
|-------------|-----------------|
|       41548 |            1.00 |
|       42203 |            4.11 |
|         115 |            5.93 |
|         171 |            9.36 |
|           4 |           11.86 |

***

**3. Which 5 interests had the largest standard deviation in their percentile_ranking value?**
````sql
with months_table as (
select interest_id,
	count(*) as total_months -- counting ids appearance 
from interest_metrics
group by interest_id
)

, filtered_table as (
select interest_id
from months_table
where total_months >= 6
)

select im.interest_id,
	round(stddev(im.percentile_ranking)::numeric,2) as stddev_percentile_ranking
from interest_metrics im 
join filtered_table ft
on im.interest_id = ft.interest_id
group by im.interest_id 
order by stddev(im.ranking) desc 
limit 5
````
Answer:
| interest_id | stddev_percentile_ranking |
|-------------|---------------------------|
|          23 |                     30.18 |
|       20764 |                     28.97 |
|       10839 |                     25.61 |
|        6298 |                     23.78 |
|         114 |                     23.74 |

***

**4. For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?**

Maxium percentile_ranking for each interest.
````sql
with months_table as (
select interest_id,
	count(*) as total_months -- counting ids appearance 
from interest_metrics
group by interest_id
)

, filtered_table as (
select interest_id
from months_table
where total_months >= 6
)
, order_table as (
select im.interest_id,
	im.month_year,
	im.percentile_ranking,
	row_number () over (partition by im.interest_id order by im.percentile_ranking desc) as orders
from interest_metrics im 
join filtered_table ft
on im.interest_id = ft.interest_id
where im.interest_id in (23, 20764, 10839, 6298, 114)
)

select 
	interest_id,
	month_year,
	percentile_ranking 
from order_table
where orders = 1
````
Answer:
| interest_id | month_year | percentile_ranking |
|-------------|------------|--------------------|
|          23 | 2018-07-01 |              86.69 |
|         114 | 2018-07-01 |              93.42 |
|        6298 | 2018-07-01 |              63.92 |
|       10839 | 2018-07-01 |              75.03 |
|       20764 | 2018-07-01 |              86.15 |

Minimum percentile_ranking for each interest
````sql
with months_table as (
select interest_id,
	count(*) as total_months -- counting ids appearance 
from interest_metrics
group by interest_id
)

, filtered_table as (
select interest_id
from months_table
where total_months >= 6
)
, order_table as (
select im.interest_id,
	im.month_year,
	im.percentile_ranking,
	row_number () over (partition by im.interest_id order by im.percentile_ranking) as orders
from interest_metrics im 
join filtered_table ft
on im.interest_id = ft.interest_id
where im.interest_id in (23, 20764, 10839, 6298, 114)
)

select 
	interest_id,
	month_year,
	percentile_ranking 
from order_table
where orders = 1
````
Answer:
| interest_id | month_year | percentile_ranking |
|-------------|------------|--------------------|
|          23 | 2019-08-01 |               7.92 |
|         114 | 2019-08-01 |               7.92 |
|        6298 | 2019-08-01 |               1.13 |
|       10839 | 2019-03-01 |               4.84 |
|       20764 | 2019-08-01 |              11.23 |

***

**5. How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?**

