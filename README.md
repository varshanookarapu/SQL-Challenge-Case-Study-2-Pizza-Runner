# SQL Case Study 2: Pizza Runner

## A. Pizza Metrics

**Question 1:** How many pizzas were ordered?

---

## SQL Code

```sql
SELECT COUNT(order_id) AS pizzas_ordered
FROM customer_orders;

SELECT COUNT(DISTINCT order_id) AS unique_customer_orders
FROM customer_orders; ```


**Question 2:** How many unique customer orders were made?

---

## SQL Code
```sql
SELECT COUNT(DISTINCT order_id) AS unique_customer_orders
FROM customer_orders; ```


Question 3: How many of each type of pizza was delivered?

## SQL Code

WITH customer_orders_pizza_names AS (
    SELECT order_id,
           customer_id,
           co.pizza_id,
           exclusions,
           extras,
           order_time,
           pizza_name
    FROM customer_orders co
    LEFT JOIN pizza_names pn ON co.pizza_id = pn.pizza_id
)
SELECT pizza_name, COUNT(pizza_id) AS total_pizzas_delivered
FROM customer_orders_pizza_names
GROUP BY pizza_name;


Question 4: How many Vegetarian and Meatlovers were ordered by each customer? - Join customer_orders and pizza_names tables

## SQL Code

WITH customer_orders_pizza_names AS (
    SELECT order_id,
           customer_id,
           co.pizza_id,
           exclusions,
           extras,
           order_time,
           pizza_name
    FROM customer_orders co
    LEFT JOIN pizza_names pn ON co.pizza_id = pn.pizza_id
)
SELECT customer_id,
       pizza_name,
       COUNT(pizza_id) AS pizzas_ordered
FROM customer_orders_pizza_names
GROUP BY customer_id, pizza_id, pizza_name
ORDER BY customer_id;


Question 5: How many pizzas were delivered that had both exclusions and extras?

## SQL Code

WITH customer_orders_pizza_names AS (
    SELECT order_id,
           customer_id,
           co.pizza_id,
           exclusions,
           extras,
           order_time,
           pizza_name
    FROM customer_orders co
    LEFT JOIN pizza_names pn ON co.pizza_id = pn.pizza_id
)
SELECT COUNT(order_id) AS pizzas_delivered_with_exclusions_and_extras
FROM customer_orders_pizza_names
WHERE exclusions IS NOT NULL AND extras IS NOT NULL;
