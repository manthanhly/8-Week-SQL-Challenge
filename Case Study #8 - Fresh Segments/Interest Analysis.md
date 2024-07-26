### Interest Analysis

**1. Which interests have been present in all <code>month_year</code> dates in our dataset?**

Finding the total unique month_year
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

***

**2. Using this same <code>total_months</code> measure - calculate the cumulative percentage of all records starting at 14 months - which <code>total_months</code> value passes the 90% cumulative percentage value?**

***

**3. If we were to remove all <code>interest_id</code> values which are lower than the <code>total_months</code> value we found in the previous question - how many total data points would we be removing?**

***

**4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed <code>interest</code> example for your arguments - think about what it means to have less months present from a segment perspective.**

***

**5. After removing these interests - how many unique interests are there for each month?**

