### Index Analysis

The <code>index_value</code> is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.

Average composition can be calculated by dividing the <code>composition</code> column by the <code>index_value</code> column rounded to 2 decimal places.

**1. What is the top 10 interests by the average composition for each month?**
````sql
with average_composition_table as (
select interest_id,
	month_year,
	round((composition/index_value)::numeric,2) as average_composition
from interest_metrics
)
, rank_table as (
select interest_id,
	month_year,
	average_composition,
	rank() over (partition by month_year order by average_composition desc) as ranking
from average_composition_table
)

select interest_id,
	month_year,
	average_composition
from rank_table
where ranking <= 10
order by month_year,
	ranking
````
Answer: - only shows value of 2018-07-01
| interest_id | month_year | average_composition |
|-------------|------------|---------------------|
|        6324 | 2018-07-01 |                7.36 |
|        6284 | 2018-07-01 |                6.94 |
|        4898 | 2018-07-01 |                6.78 |
|          77 | 2018-07-01 |                6.61 |
|          39 | 2018-07-01 |                6.51 |
|       18619 | 2018-07-01 |                6.10 |
|        6208 | 2018-07-01 |                5.72 |
|       21060 | 2018-07-01 |                4.85 |
|       21057 | 2018-07-01 |                4.80 |
|          82 | 2018-07-01 |                4.71 |
|         ... | ... |                ...|

***

**2. For all of these top 10 interests - which interest appears the most often?**
````sql
with average_composition_table as (
select interest_id,
	month_year,
	round((composition/index_value)::numeric,2) as average_composition
from interest_metrics
)
, rank_table as (
select interest_id,
	month_year,
	average_composition,
	rank() over (partition by month_year order by average_composition desc) as ranking
from average_composition_table
)
, top_composition_table as (
select interest_id,
	month_year,
	average_composition 
from rank_table
where ranking <= 10
)

select average_composition,
	count(*) as frequency 
from top_composition_table
group by average_composition
order by count(*) desc
limit 1
````
Answer:
| average_composition | frequency |
|---------------------|-----------|
|                5.65 |         3 |

***

**3. What is the average of the average composition for the top 10 interests for each month?**
````sql
with average_composition_table as (
select interest_id,
	month_year,
	round((composition/index_value)::numeric,2) as average_composition
from interest_metrics
)
, rank_table as (
select interest_id,
	month_year,
	average_composition,
	rank() over (partition by month_year order by average_composition desc) as ranking
from average_composition_table
)
, top_composition_table as (
select interest_id,
	month_year,
	average_composition 
from rank_table
where ranking <= 10
)

select month_year,
	round(avg(average_composition),2) as average_of_average
from top_composition_table
group by month_year
````
Answer:
| month_year | average_of_average |
|------------|---------------------|
| 2018-07-01 |               0.58 |
| 2018-08-01 |               0.66 |
| 2018-09-01 |               0.71 |
| 2018-10-01 |               0.63 |
| 2018-11-01 |               0.75 |
| 2018-12-01 |               0.73 |
| 2019-01-01 |               0.73 |
| 2019-02-01 |               0.56 |
| 2019-03-01 |               0.61 |
| 2019-04-01 |               0.76 |
| 2019-05-01 |               1.07 |
| 2019-06-01 |               1.22 |
| 2019-07-01 |               0.97 |
| 2019-08-01 |               0.88 |

***

**4. What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.**
````sql
with average_composition_table as (
select ime.interest_id,
	ima.interest_name,
	ime.month_year,
	round((ime.composition/ime.index_value)::numeric,2) as average_composition
from interest_metrics ime
join interest_map ima
on ime.interest_id = ima.id
)
, rank_table as (
select interest_name,
	month_year,
	average_composition,
	rank() over (partition by month_year order by average_composition desc) as ranking
from average_composition_table
)
, max_table as (
select interest_name,
	month_year,
	average_composition as max_index_composition
from rank_table 
where ranking = 1 -- fixing the max of for each month, also needs to show interest_name
)

, combined_table as (
select interest_name,
	month_year,
	max_index_composition,
	avg(max_index_composition) over (order by month_year rows between 2 preceding and current row) as _3_month_moving_avg, -- moving average function
	lag(interest_name,1) over (order by month_year) as _1_month_ago_name,
	lag(max_index_composition,1) over (order by month_year) as _1_month_ago_value,
	lag(interest_name,2) over (order by month_year) as _2_months_ago_name,
	lag(max_index_composition,2) over (order by month_year) as _2_months_ago_value
from max_table
)

select month_year,
	interest_name,
	max_index_composition,
	round(_3_month_moving_avg,2),
	concat(_1_month_ago_name,': ',_1_month_ago_value) as _1_month_ago,
	concat(_2_months_ago_name,': ',_2_months_ago_value) as _2_month_ago
from combined_table
where month_year >= '2018-09-01'
````
Answer:
| month_year | interest_name                | max_index_composition | round | _1_month_ago                     | _2_month_ago                     |
|------------|------------------------------|-----------------------|-------|---------------------------------|---------------------------------|
| 2018-09-01 | Work Comes First Travelers   |                 8.26 |  7.61 | Las Vegas Trip Planners: 7.21    | Las Vegas Trip Planners: 7.36    |
| 2018-10-01 | Work Comes First Travelers   |                 9.14 |  8.20 | Work Comes First Travelers: 8.26 | Las Vegas Trip Planners: 7.21    |
| 2018-11-01 | Work Comes First Travelers   |                 8.28 |  8.56 | Work Comes First Travelers: 9.14 | Work Comes First Travelers: 8.26 |
| 2018-12-01 | Work Comes First Travelers   |                 8.31 |  8.58 | Work Comes First Travelers: 8.28 | Work Comes First Travelers: 9.14 |
| 2019-01-01 | Work Comes First Travelers   |                 7.66 |  8.08 | Work Comes First Travelers: 8.31 | Work Comes First Travelers: 8.28 |
| 2019-02-01 | Work Comes First Travelers   |                 7.66 |  7.88 | Work Comes First Travelers: 7.66 | Work Comes First Travelers: 8.31 |
| 2019-03-01 | Alabama Trip Planners        |                 6.54 |  7.29 | Work Comes First Travelers: 7.66 | Work Comes First Travelers: 7.66 |
| 2019-04-01 | Solar Energy Researchers     |                 6.28 |  6.83 | Alabama Trip Planners: 6.54      | Work Comes First Travelers: 7.66 |
| 2019-05-01 | Readers of Honduran Content  |                 4.41 |  5.74 | Solar Energy Researchers: 6.28   | Alabama Trip Planners: 6.54      |
| 2019-06-01 | Las Vegas Trip Planners      |                 2.77 |  4.49 | Readers of Honduran Content: 4.41| Solar Energy Researchers: 6.28   |
| 2019-07-01 | Las Vegas Trip Planners      |                 2.82 |  3.33 | Las Vegas Trip Planners: 2.77    | Readers of Honduran Content: 4.41|
| 2019-08-01 | Cosmetics and Beauty Shoppers|                 2.73 |  2.77 | Las Vegas Trip Planners: 2.82    | Las Vegas Trip Planners: 2.77    |

***

**5. Provide a possible reason why the max average composition might change from month to month? Could it signal something is not quite right with the overall business model for Fresh Segments?**

