### Data Exploration and Cleansing

**1. Update the <code>fresh_segments.interest_metrics</code> table by modifying the <code>month_year</code> column to be a date data type with the start of the month**

***

**2. What is count of records in the <code>fresh_segments.interest</code>_metrics for each <code>month_year</code> value sorted in chronological order (earliest to latest) with the null values appearing first?**

***

**3. What do you think we should do with these null values in the <code>fresh_segments.interest_metrics</code>**

***

**4. How many <code>interest_id</code> values exist in the <code>fresh_segments.interest_metrics</code> table but not in the <code>fresh_segments.interest_map</code> table? What about the other way around?**

***

**5. Summarise the <code>id</code> values in the <code>fresh_segments.interest_map</code> by its total record count in this table**

***

**6. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where <code>interest_id = 21246</code> in your joined output and include all columns from <code>fresh_segments.interest_metrics</code> and all columns from <code>fresh_segments.interest_map</code> except from the <code>id</code> column.**

***

**7. Are there any records in your joined table where the <code>month_year</code> value is before the <code>created_at</code> value from the <code>fresh_segments.interest_map</code> table? Do you think these values are valid and why?**

