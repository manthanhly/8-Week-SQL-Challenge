### A. Customer Journey 

Based off the 8 sample customers provided in the sample from the <code>subscriptions</code> table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

Answer:

````sql
select s.customer_id,
	p.plan_name,
	s.start_date
from subscriptions s
left join plans p on s.plan_id = p.plan_id
where s.customer_id in (1, 2, 11, 13, 15, 16, 18, 19)
order by s.customer_id
````

- Customer 1 - decided to go with basic after trial

| customer_id | plan_name    | start_date |
|-------------|--------------|------------|
|           1 | trial        | 2020-08-01 |
|           1 | basic monthly| 2020-08-08 |

- Customer 2 - decided to upgrade to pro annual after trial

| customer_id | plan_name | start_date |
|-------------|-----------|------------|
|           2 | trial     | 2020-09-20 |
|           2 | pro annual| 2020-09-27 |

- Customer 11 - cancelled after trial

| customer_id | plan_name | start_date |
|-------------|-----------|------------|
|          11 | trial     | 2020-11-19 |
|          11 | churn     | 2020-11-26 |

- Customer 13 - go with basic after trial then decided to upgrade to pro monthly after 3 months 

| customer_id | plan_name    | start_date |
|-------------|--------------|------------|
|          13 | trial        | 2020-12-15 |
|          13 | basic monthly| 2020-12-22 |
|          13 | pro monthly  | 2021-03-29 |

- Customer 15 - might either forget to cancel/downgrade or want to keep the pro monthly. Customer decided to cancel after 1 month -> customer forgot to cancel after trial
  
| customer_id | plan_name  | start_date |
|-------------|------------|------------|
|          15 | trial      | 2020-03-17 |
|          15 | pro monthly| 2020-03-24 |
|          15 | churn      | 2020-04-29 |

- Customer 16 - keep basic month after trial and decided to upgrade to pro annual after 4 months 

| customer_id | plan_name    | start_date |
|-------------|--------------|------------|
|          16 | trial        | 2020-05-31 |
|          16 | basic monthly| 2020-06-07 |
|          16 | pro annual   | 2020-10-21 |

- Customer 18 - might either forget to cancel/downgrade or want to keep the pro monthly
  
| customer_id | plan_name  | start_date |
|-------------|------------|------------|
|          18 | trial      | 2020-07-06 |
|          18 | pro monthly| 2020-07-13 |

- Customer 19 - most likely wanted to use the service after trial, go with pro monthly and decided to upgrade to pro annual after 2 months to save money

| customer_id | plan_name  | start_date |
|-------------|------------|------------|
|          19 | pro monthly| 2020-06-29 |
|          19 | pro annual | 2020-08-29 |
