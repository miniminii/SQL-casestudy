## Case Study #2 - Pizza Runner 🍕

### Introduction
Danny was scrolling through his Instagram feed when something really caught his eye - “80s Retro Styling and Pizza Is The Future!”

Danny was sold on the idea, but he knew that pizza alone was not going to help him get seed funding to expand his new Pizza Empire - so he had one more genius idea to combine with it - he was going to *Uberize* it - and so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

### Entity Relationship Diagram
![2PizzaRunner_ERdiagram](https://user-images.githubusercontent.com/116126763/221021905-45de5075-1355-4efe-bbd9-7099d30f0608.PNG)

### Datasets used

**Table 1:** `runners`  
The runners table shows the registration_date for each new runner.

**Table 2:** `customer_orders`  
Customer pizza orders are captured in the customer_orders table with 1 row for each individual pizza that is part of the order.The pizza_id relates to the type of pizza which was ordered whilst the exclusions are the ingredient_id values which should be removed from the pizza and the extras are the ingredient_id values which need to be added to the pizza.

Note that customers can order multiple pizzas in a single order with varying exclusions and extras values even if the pizza is the same type!

**Table 3:** `runner_orders`  
After each orders are received through the system - they are assigned to a runner - however not all orders are fully completed and can be cancelled by the restaurant or the customer. The pickup_time is the timestamp at which the runner arrives at the Pizza Runner headquarters to pick up the freshly cooked pizzas. The distance and duration fields are related to how far and long the runner had to travel to deliver the order to the respective customer.

**Table 4:** `pizza_names`  
At the moment - Pizza Runner only has 2 pizzas available the Meat Lovers or Vegetarian!

**Table 5:** `pizza_recipes`  
Each pizza_id has a standard set of toppings which are used as part of the pizza recipe.

**Table 6:** `pizza_toppings`  
This table contains all of the topping_name values with their corresponding topping_id value.

### Data Cleansing
There are some known data issues with few tables. Data cleaning was performed and saved in temporary tables before attempting the case study.

`customer_orders` table

- The exclusions and extras columns in customer_orders table will need to be cleaned up before using them in the queries
- In the exclusions and extras columns, there are blank spaces and null values.

`runner_orders` table

- The pickup_time, distance, duration and cancellation columns in runner_orders table will need to be cleaned up before using them in the queries
- In the pickup_time column, there are null values.
- In the distance column, there are null values. It contains unit - km. The 'km' must also be stripped
- In the duration column, there are null values. The 'minutes', 'mins' 'minute' must be stripped
- In the cancellation column, there are blank spaces and null values.
