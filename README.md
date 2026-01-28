# SQL Case Study 2: Pizza Runner

## Data Cleaning Customer Orders Table 

```sql

--Cleaning Customer Orders Table Exclustions and Extras Columns - updating blank vlaues to nulls
-- Checking for null string value, blank values and actual null and replacing them all with nulls

UPDATE customer_orders
SET exclusions = null
WHERE exclusions IS null OR TRIM(exclusions) = '' OR exclusions = 'null';
   

UPDATE customer_orders
SET extras = null
WHERe extras IS NULL  OR TRIM(extras)='' OR extras  = 'null';

-- Checking Duplicates 
SELECT order_id,customer_id,pizza_id,exclusions,extras,order_time, COUNT(*)
FROM customer_orders
GROUP BY order_id,customer_id,order_time,pizza_id,exclusions,extras
HAVING COUNT(*) > 1;


--Deleting the duplicate records
--ctid → a unique physical identifier for each row  in PostgresSQL

DELETE FROM customer_orders 
WHERE ctid NOT IN ( SELECT MIN(ctid) FROM customer_orders GROUP BY order_id,customer_id,order_time,pizza_id,exclusions,extras ) ;
            
-- Final cleaned customer_orders Table
SELECT * FROM customer_orders ORDER BY order_id;

```

---

## Data Cleaning Runner Orders Table

```sql
--cancellation column

UPDATE runner_orders 
SET cancellation = null 
WHERe cancellation = 'null' OR  TRIM(cancellation) = '' OR 
cancellation = 'NaN' ;

--pickup_time
UPDATE runner_orders 
SET pickup_time = null 
WHERe pickup_time = 'null' OR  TRIM(pickup_time) = '' OR 
pickup_time = 'NaN' ;

--distance
UPDATE runner_orders 
SET distance = null 
WHERe distance = 'null' OR  TRIM(distance) = '' OR 
distance = 'NaN' ;
--duration 
UPDATE runner_orders 
SET duration = null 
WHERe duration = 'null' OR  TRIM(duration) = '' OR 
duration = 'NaN' ;

-- removing km and mins from distance and duration columns

UPDATE runner_orders
SET distance = REGEXP_REPLACE( distance, '[^0-9.]', '', 'g' ) ;


UPDATE runner_orders
SET duration = REGEXP_REPLACE (duration , '[^0-9.]','','g');


-- Changing the data types of pickup_time, distance and duration columns

ALTER TABLE runner_orders
ALTER COLUMN distance TYPE decimal USING distance :: decimal; 

ALTER TABLE runner_orders
ALTER duration TYPE integer USING duration ::integer;

ALTER TABLE runner_orders
ALTER pickup_time TYPE timestamp USING  pickup_time :: timestamp; 


SELECT * FROM runner_orders ORDER BY order_id

```


## A. Pizza Metrics

**Question 1:** How many pizzas were ordered?

---

## SQL Code

```sql
SELECT COUNT(order_id) AS pizzas_ordered
FROM customer_orders;
```


**Question 2:** How many unique customer orders were made?

---

## SQL Code
```sql
SELECT COUNT(DISTINCT order_id) AS unique_customer_orders
FROM customer_orders;
```

**Question 3:** How many successful orders were delivered by each runner?

---
## SQL Code

```sql
SELECT runner_id, COUNT(pickup_time) as successful_orders FROM runner_orders WHERE 
pickup_time IS NOT NULL AND cancellation IS null 
GROUP BY runner_id

```


**Question 4:** How many of each type of pizza was delivered?

---

## SQL Code

```sql
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
```


**Question 5**: How many Vegetarian and Meatlovers were ordered by each customer? - Join customer_orders and pizza_names tables

---

## SQL Code

```sql
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
```


**Question 8**: How many pizzas were delivered that had both exclusions and extras?

---

## SQL Code

```sql
WITH customer_orders_runner_orders AS 
(
SELECT co.order_id,customer_id,pizza_id,exclusions,extras,order_time,runner_id,pickup_time,distance,duration,cancellation FROM customer_orders co LEFT JOIN runner_orders ro ON
co.order_id = ro.order_id 
ORDER BY co.order_id
)

SELECT COUNT(order_id) as  pizzas_delivered_with_exclusions_and_extras
FROM  customer_orders_runner_orders WHERE exclusions IS NOT NULL AND extras IS NOT NULL AND pickup_time IS NOT NULL
```
