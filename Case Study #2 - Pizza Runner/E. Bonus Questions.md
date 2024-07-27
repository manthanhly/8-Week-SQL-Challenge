### E. Bonus Questions 

**If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an <code>INSERT</code> statement to demonstrate what would happen if a new <code>Supreme</code> pizza with all the toppings was added to the Pizza Runner menu?**
````sql
insert into pizza_names (pizza_id, pizza_name)
values (3,'Supreme')

insert into pizza_recipes (pizza_id, toppings)
values (3, array_to_string(array(select generate_series(1,10)), ', '))
````
