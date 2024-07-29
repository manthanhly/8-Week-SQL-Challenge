### Bonus Challenge 

Use a single SQL query to transform the <code>product_hierarchy</code> and <code>product_prices</code> datasets to the <code>product_details</code> table.

Hint: you may want to consider using a recursive CTE to solve this problem!

````sql
with recursive hierarchy as (
   select
       id,
       level_text as category_name,
       null::varchar as segment_name,
       null::varchar as style_name,
       id as category_id,
       null::integer as segment_id,
       null::integer as style_id
   from product_hierarchy
   where level_name = 'Category'
  
   union all
  
   select
       ph.id,
       h.category_name,
       case
           when ph.level_name = 'Segment' then ph.level_text
           else h.segment_name
       end,
       case
           when ph.level_name = 'Style' then ph.level_text
           else h.style_name
       end,
       case
           when ph.level_name = 'Category' then ph.id
           else h.category_id
       end,
       case
           when ph.level_name = 'Segment' then ph.id
           else h.segment_id
       end,
       case
           when ph.level_name = 'Style' then ph.id
           else h.style_id
       end
   from product_hierarchy ph
   join hierarchy h on ph.parent_id = h.id
)
````
Answer:
| product_id | price | concat                          | category_id | segment_id | style_id | category_name | segment_name | style_name         |
|------------|-------|---------------------------------|-------------|------------|----------|---------------|--------------|-------------------|
| c4a632     |    13 | Navy Oversized Jeans - Womens   |           1 |          3 |        7 | Womens        | Jeans        | Navy Oversized     |
| e83aa3     |    32 | Black Straight Jeans - Womens   |           1 |          3 |        8 | Womens        | Jeans        | Black Straight     |
| e31d39     |    10 | Cream Relaxed Jeans - Womens    |           1 |          3 |        9 | Womens        | Jeans        | Cream Relaxed      |
| d5e9a6     |    23 | Khaki Suit Jacket - Womens      |           1 |          4 |       10 | Womens        | Jacket       | Khaki Suit         |
| 72f5d4     |    19 | Indigo Rain Jacket - Womens     |           1 |          4 |       11 | Womens        | Jacket       | Indigo Rain        |
| 9ec847     |    54 | Grey Fashion Jacket - Womens    |           1 |          4 |       12 | Womens        | Jacket       | Grey Fashion       |
| 5d267b     |    40 | White Tee Shirt - Mens          |           2 |          5 |       13 | Mens          | Shirt        | White Tee          |
| c8d436     |    10 | Teal Button Up Shirt - Mens     |           2 |          5 |       14 | Mens          | Shirt        | Teal Button Up     |
| 2a2353     |    57 | Blue Polo Shirt - Mens          |           2 |          5 |       15 | Mens          | Shirt        | Blue Polo          |
| f084eb     |    36 | Navy Solid Socks - Mens         |           2 |          6 |       16 | Mens          | Socks        | Navy Solid         |
| b9a74d     |    17 | White Striped Socks - Mens      |           2 |          6 |       17 | Mens          | Socks        | White Striped      |
| 2feb6b     |    29 | Pink Fluro Polkadot Socks - Mens|           2 |          6 |       18 | Mens          | Socks        | Pink Fluro Polkadot|
