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

***

**5. Provide a possible reason why the max average composition might change from month to month? Could it signal something is not quite right with the overall business model for Fresh Segments?**

