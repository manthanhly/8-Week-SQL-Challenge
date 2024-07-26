### Interest Analysis

**1. Which interests have been present in all <code>month_year</code> dates in our dataset?**

Finding the total unique month_year - max value is 14. 
````sql
select count(distinct ime.month_year) as month_year_count
from interest_metrics ime
inner join interest_map ima
on ime.interest_id = ima.id
````
Answer:
| month_year_count |
|------------------|
|               14 |

Finding number of interests have been present in 14 unique months - 480 interests.
````sql
with total_table as (
select ima.id,
	ima.interest_name,
	count(*) as total_count -- counting ids 
from interest_metrics ime
join interest_map ima
on ime.interest_id = ima.id
group by ima.id,
	ima.interest_name
)

select total_count,
	count(total_count) as numbers_of_id -- finding the number of ids per their total_count 
from total_table
where total_count = 14
group by total_count
````
Answer:
| total_count | numbers_of_id |
|-------------|---------------|
|          14 |          480  |


***

**2. Using this same <code>total_months</code> measure - calculate the cumulative percentage of all records starting at 14 months - which <code>total_months</code> value passes the 90% cumulative percentage value?**
````sql
with months_table as (
select interest_id,
	count(*) as total_months -- counting ids appearance 
from interest_metrics
group by interest_id
)
,count_table as (
select total_months,
	count(*) as interest_id_count -- finding the number of ids per their total_count 
from months_table
group by total_months
)

select 
	total_months,
	round((sum(interest_id_count) over (order by total_months desc)*100/sum(interest_id_count) over ()),2) as cummulative_percentage
from count_table
````
Answer:
| total_months | cummulative_percentage |
|--------------|-------------------------|
|           14 |                   39.93 |
|           13 |                   46.76 |
|           12 |                   52.16 |
|           11 |                   59.98 |
|           10 |                   67.14 |
|            9 |                   75.04 |
|            8 |                   80.62 |
|            7 |                   88.10 |
|            6 |                   90.85 |
|            5 |                   94.01 |
|            4 |                   96.67 |
|            3 |                   97.92 |
|            2 |                   98.92 |
|            1 |                  100.00 |

The value of 6 months is the point where the cummulative percentage first exceeds 90%. 
***

**3. If we were to remove all <code>interest_id</code> values which are lower than the <code>total_months</code> value we found in the previous question - how many total data points would we be removing?**

We found 6 as the total months from previous quesiton. 
````sql
with months_table as (
select interest_id,
	count(*) as total_months -- counting ids appearance 
from interest_metrics
group by interest_id
)

select count(*) as total_to_remove
from months_table
where total_months < 6
````
Answer:
| total_to_remove |
|-----------------|
|             110 |

***

**4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed <code>interest</code> example for your arguments - think about what it means to have less months present from a segment perspective.**

From a business perspective, removing interest_ids with fewer than 6 months of activity can make sense if your focus is on long-term engagement and retention. However, it’s essential to weigh this against the potential loss of insights into early-stage interactions and trends. 

***

**5. After removing these interests - how many unique interests are there for each month?**

