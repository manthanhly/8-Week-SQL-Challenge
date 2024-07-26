### Data Exploration and Cleansing

**1. Update the <code>fresh_segments.interest_metrics</code> table by modifying the <code>month_year</code> column to be a date data type with the start of the month**
````sql
alter table interest_metrics 
alter column month_year type date using to_date('01-' || month_year, 'DD-MM-YYYY') -- converted month_year to the right format before alter data type
````
Answer:
| _month | _year | month_year | interest_id | composition | index_value | ranking | percentile_ranking |
|--------|-------|------------|-------------|-------------|-------------|---------|--------------------|
| 7      | 2018  | 2018-07-01 | 32486       |       11.89 |        6.19 |       1 |              99.86 |
| 7      | 2018  | 2018-07-01 | 6106        |        9.93 |        5.31 |       2 |              99.73 |
| 7      | 2018  | 2018-07-01 | 18923       |       10.85 |        5.29 |       3 |              99.59 |
| 7      | 2018  | 2018-07-01 | 6344        |       10.32 |         5.1 |       4 |              99.45 |
| 7      | 2018  | 2018-07-01 | 100         |       10.77 |        5.04 |       5 |              99.31 |
| ...     | ... | ... | ...        |       ...|       ... |      ... |            ... |
***

**2. What is count of records in the <code>fresh_segments.interest_metrics</code> for each <code>month_year</code> value sorted in chronological order (earliest to latest) with the null values appearing first?**
````sql
select	month_year, 
	count(*) as records_count
from interest_metrics
group by month_year 
order by month_year nulls first 
````
Answer:
| month_year | records_count |
|------------|---------------|
|            |          1194 |
| 2018-07-01 |           729 |
| 2018-08-01 |           767 |
| 2018-09-01 |           780 |
| 2018-10-01 |           857 |
| 2018-11-01 |           928 |
| 2018-12-01 |           995 |
| 2019-01-01 |           973 |
| 2019-02-01 |          1121 |
| 2019-03-01 |          1136 |
| 2019-04-01 |          1099 |
| 2019-05-01 |           857 |
| 2019-06-01 |           824 |
| 2019-07-01 |           864 |
| 2019-08-01 |          1149 |

***

**3. What do you think we should do with these null values in the <code>fresh_segments.interest_metrics</code>**

Null values rows also have null <code>interest_id</code> so I think we should remove those null values. Nulls values count is about 8% of all values. 
````sql
select sum(case when month_year is null then 1 else 0 end)*100/count(*)::numeric as null_percentage
from interest_metrics
````
| null_percentage   |
|-------------------|
| 8.3654452462691796|

Remove null values:
````sql
delete from interest_metrics
where month_year is null
````

***

**4. How many <code>interest_id</code> values exist in the <code>fresh_segments.interest_metrics</code> table but not in the <code>fresh_segments.interest_map</code> table? What about the other way around?**

We will need to join <code>interest_metrics</code>  and <code>interest_map</code>, but we will have to change interest_id data type to integer after we removed the null values.

````sql
alter table interest_metrics 
alter column interest_id type integer using interest_id::integer
````

We are now able to join those 2 tables.
````sql
select count(distinct ime.interest_id) as metrics_id_count,
	count(distinct ima.id) as map_id_count,
	sum(case when ima.id is null then 1 end) as not_metrics,
	sum(case when ime.interest_id is null then 1 end) as not_maps
from interest_metrics ime
full outer join interest_map ima
on ime.interest_id = ima.id
````
Answer:
| metrics_id_count | map_id_count | not_metrics | not_maps |
|------------------|--------------|-------------|----------|
|             1202 |         1209 |             |        7 |

None of the interest_id exist in <code>interest_metrics</code>  table but not in the <code>interest_map</code>. There are 7 interest_id exist in the other way around. 
***

**5. Summarise the <code>id</code> values in the <code>fresh_segments.interest_map</code> by its total record count in this table**
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
group by total_count
order by total_count
````
Answer:
| total_count | numbers_of_id |
|-------------|---------------|
|           1 |            13 |
|           2 |            12 |
|           3 |            15 |
|           4 |            32 |
|           5 |            38 |
|           6 |            33 |
|           7 |            90 |
|           8 |            67 |
|           9 |            95 |
|          10 |            86 |
|          11 |            94 |
|          12 |            65 |
|          13 |            82 |
|          14 |           480 |

***

**6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where <code>interest_id = 21246</code> in your joined output and include all columns from <code>fresh_segments.interest_metrics</code> and all columns from <code>fresh_segments.interest_map</code> except from the <code>id</code> column.**

Since we already removed the null values so the type of inner join does not really matter now. We can now use left or inner join to join these 2 tables. However, if we have not removed those null values, we should use inner join. 
````sql
select *
from interest_metrics ime
inner join interest_map ima
on ime.interest_id = ima.id
where ime.interest_id = 21246
````
Anwer:
| _month | _year | month_year | interest_id | composition | index_value | ranking | percentile_ranking | id   | interest_name                   | interest_summary                                     | created_at             | last_modified          |
|--------|-------|------------|-------------|-------------|-------------|---------|---------------------|------|--------------------------------|-----------------------------------------------------|------------------------|------------------------|
| 7      | 2018  | 2018-07-01 | 21246       | 2.26        | 0.65        | 722     | 0.96                | 21246| Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 8      | 2018  | 2018-08-01 | 21246       | 2.13        | 0.59        | 765     | 0.26                | 21246| Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 9      | 2018  | 2018-09-01 | 21246       | 2.06        | 0.61        | 774     | 0.77                | 21246| Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 10     | 2018  | 2018-10-01 | 21246       | 1.74        | 0.58        | 855     | 0.23                | 21246| Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 11     | 2018  | 2018-11-01 | 21246       | 2.25        | 0.78        | 908     | 2.16                | 21246| Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 12     | 2018  | 2018-12-01 | 21246       | 1.97        | 0.70        | 983     | 1.21                | 21246| Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 1      | 2019  | 2019-01-01 | 21246       | 2.05        | 0.76        | 954     | 1.95                | 21246| Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 2      | 2019  | 2019-02-01 | 21246       | 1.84        | 0.68        | 1109    | 1.07                | 21246| Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 3      | 2019  | 2019-03-01 | 21246       | 1.75        | 0.67        | 1123    | 1.14                | 21246| Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |
| 4      | 2019  | 2019-04-01 | 21246       | 1.58        | 0.63        | 1092    | 0.64                | 21246| Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11 17:50:04.000 | 2018-06-11 17:50:04.000 |

***

**7. Are there any records in your joined table where the <code>month_year</code> value is before the <code>created_at</code> value from the <code>fresh_segments.interest_map</code> table? Do you think these values are valid and why?**

Yes, there are 188 records.

````sql
select 
	count(*)
from interest_metrics ime
inner join interest_map ima
on ime.interest_id = ima.id
where ime.month_year < ima.created_at 
````
Answer:
| count |
|-------|
|   188 |

And yes these values are valid. On the 2nd question, we were asked to convert month_year to the start of the month. Created_at's month and year is the same as the month_year. 

````sql
select 
	count(*)
from interest_metrics ime
inner join interest_map ima
on ime.interest_id = ima.id
where ime.month_year < date_trunc('month', ima.created_at::date)
````
Answer:
| count |
|-------|
|   0 |
