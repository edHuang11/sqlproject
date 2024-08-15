``` sql
# create temp table
CREATE TABLE customer_orders_temp AS
SELECT 
  order_id, 
  customer_id, 
  pizza_id, 
  CASE
    WHEN exclusions LIKE '' OR exclusions LIKE 'NULL' THEN NULL
    ELSE exclusions
    END AS exclusions,
  CASE
    WHEN extras LIKE '' OR extras LIKE 'NULL' THEN NULL
    ELSE extras
    END AS extras,
  order_time
FROM pizza_runner.customer_orders;

# create temp table
CREATE TABLE runner_orders_temp AS
SELECT order_id, runner_id,
	CASE
    WHEN pickup_time IS NULL OR pickup_time LIKE 'NULL' THEN NULL
    ELSE pickup_time END AS pickup_time,
    CASE
    WHEN distance IS NULL OR distance LIKE 'NULL' THEN NULL
    WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
    ELSE distance END AS distance,
    CASE
    WHEN duration LIKE '' OR duration LIKE 'NULL' THEN NULL
    WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration) 
    WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
    WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
    ELSE duration END AS duration,
    CASE
    WHEN cancellation LIKE '' OR cancellation LIKE 'NULL' THEN NULL
    ELSE cancellation END AS cancellation
FROM pizza_runner.runner_orders;

# alter datatypes
ALTER TABLE pizza_runner.runner_orders_temp
MODIFY COLUMN pickup_time DATETIME,
MODIFY COLUMN duration INT,
MODIFY COLUMN distance FLOAT;

ALTER TABLE pizza_runner.customer_orders_temp
MODIFY COLUMN order_time DATETIME;

```
---

**Query #1**

    SELECT * FROM pizza_runner.customer_orders_temp;

| order_id | customer_id | pizza_id | exclusions | extras | order_time          |
| -------- | ----------- | -------- | ---------- | ------ | ------------------- |
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02 |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52 |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23 |
| 3        | 102         | 2        |            |        | 2020-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46 |
| 5        | 104         | 1        |            | 1      | 2020-01-08 21:00:29 |
| 6        | 101         | 2        |            |        | 2020-01-08 21:03:13 |
| 7        | 105         | 2        |            | 1      | 2020-01-08 21:20:29 |
| 8        | 102         | 1        |            |        | 2020-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59 |
| 10       | 104         | 1        |            |        | 2020-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11 18:34:49 |

---
**Query #2**

    SELECT * FROM pizza_runner.runner_orders_temp;

| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
| -------- | --------- | ------------------- | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4     | 20       |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |                         |
| 6        | 3         |                     |          |          | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       |                         |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |                         |
| 9        | 2         |                     |          |          | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       |                         |

---

**Schema (MySQL v8.0)**

    CREATE SCHEMA pizza_runner;
    USE pizza_runner;
    
    DROP TABLE IF EXISTS runners;
    CREATE TABLE runners (
      runner_id INT,
      registration_date DATE
    );
    INSERT INTO runners (runner_id, registration_date)
    VALUES
      (1, '2021-01-01'),
      (2, '2021-01-03'),
      (3, '2021-01-08'),
      (4, '2021-01-15');
    
    DROP TABLE IF EXISTS customer_orders;
    CREATE TABLE customer_orders (
      order_id INT,
      customer_id INT,
      pizza_id INT,
      exclusions VARCHAR(4),
      extras VARCHAR(4),
      order_time TIMESTAMP
    );
    
    INSERT INTO customer_orders (order_id, customer_id, pizza_id, exclusions, extras, order_time)
    VALUES
      (1, 101, 1, '', '', '2020-01-01 18:05:02'),
      (2, 101, 1, '', '', '2020-01-01 19:00:52'),
      (3, 102, 1, '', '', '2020-01-02 23:51:23'),
      (3, 102, 2, '', NULL, '2020-01-02 23:51:23'),
      (4, 103, 1, '4', '', '2020-01-04 13:23:46'),
      (4, 103, 1, '4', '', '2020-01-04 13:23:46'),
      (4, 103, 2, '4', '', '2020-01-04 13:23:46'),
      (5, 104, 1, NULL, '1', '2020-01-08 21:00:29'),
      (6, 101, 2, NULL, NULL, '2020-01-08 21:03:13'),
      (7, 105, 2, NULL, '1', '2020-01-08 21:20:29'),
      (8, 102, 1, NULL, NULL, '2020-01-09 23:54:33'),
      (9, 103, 1, '4', '1, 5', '2020-01-10 11:22:59'),
      (10, 104, 1, NULL, NULL, '2020-01-11 18:34:49'),
      (10, 104, 1, '2, 6', '1, 4', '2020-01-11 18:34:49');
    
    DROP TABLE IF EXISTS runner_orders;
    CREATE TABLE runner_orders (
      order_id INT,
      runner_id INT,
      pickup_time VARCHAR(19),
      distance VARCHAR(7),
      duration VARCHAR(10),
      cancellation VARCHAR(23)
    );
    
    INSERT INTO runner_orders (order_id, runner_id, pickup_time, distance, duration, cancellation)
    VALUES
      (1, 1, '2020-01-01 18:15:34', '20km', '32 minutes', ''),
      (2, 1, '2020-01-01 19:10:54', '20km', '27 minutes', ''),
      (3, 1, '2020-01-03 00:12:37', '13.4km', '20 mins', NULL),
      (4, 2, '2020-01-04 13:53:03', '23.4', '40', NULL),
      (5, 3, '2020-01-08 21:10:57', '10', '15', NULL),
      (6, 3, NULL, NULL, NULL, 'Restaurant Cancellation'),
      (7, 2, '2020-01-08 21:30:45', '25km', '25mins', NULL),
      (8, 2, '2020-01-10 00:15:02', '23.4 km', '15 minute', NULL),
      (9, 2, NULL, NULL, NULL, 'Customer Cancellation'),
      (10, 1, '2020-01-11 18:50:20', '10km', '10minutes', NULL);
    
    DROP TABLE IF EXISTS pizza_names;
    CREATE TABLE pizza_names (
      pizza_id INT,
      pizza_name TEXT
    );
    INSERT INTO pizza_names (pizza_id, pizza_name)
    VALUES
      (1, 'Meatlovers'),
      (2, 'Vegetarian');
    
    DROP TABLE IF EXISTS pizza_recipes;
    CREATE TABLE pizza_recipes (
      pizza_id INT,
      toppings TEXT
    );
    INSERT INTO pizza_recipes (pizza_id, toppings)
    VALUES
      (1, '1, 2, 3, 4, 5, 6, 8, 10'),
      (2, '4, 6, 7, 9, 11, 12');
    
    DROP TABLE IF EXISTS pizza_toppings;
    CREATE TABLE pizza_toppings (
      topping_id INT,
      topping_name TEXT
    );
    INSERT INTO pizza_toppings (topping_id, topping_name)
    VALUES
      (1, 'Bacon'),
      (2, 'BBQ Sauce'),
      (3, 'Beef'),
      (4, 'Cheese'),
      (5, 'Chicken'),
      (6, 'Mushrooms'),
      (7, 'Onions'),
      (8, 'Pepperoni'),
      (9, 'Peppers'),
      (10, 'Salami'),
      (11, 'Tomatoes'),
      (12, 'Tomato Sauce');
    
      
    CREATE TABLE customer_orders_temp AS
    SELECT 
      order_id, 
      customer_id, 
      pizza_id, 
      CASE
        WHEN exclusions LIKE '' OR exclusions LIKE 'NULL' THEN NULL
        ELSE exclusions
        END AS exclusions,
      CASE
        WHEN extras LIKE '' OR extras LIKE 'NULL' THEN NULL
        ELSE extras
        END AS extras,
      order_time
    FROM pizza_runner.customer_orders;
    
    CREATE TABLE runner_orders_temp AS
    SELECT order_id, runner_id,
    	CASE
        WHEN pickup_time IS NULL OR pickup_time LIKE 'NULL' THEN NULL
        ELSE pickup_time END AS pickup_time,
        CASE
        WHEN distance IS NULL OR distance LIKE 'NULL' THEN NULL
        WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
        ELSE distance END AS distance,
        CASE
        WHEN duration LIKE '' OR duration LIKE 'NULL' THEN NULL
        WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration) 
        WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
        WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
        ELSE duration END AS duration,
        CASE
        WHEN cancellation LIKE '' OR cancellation LIKE 'NULL' THEN NULL
        ELSE cancellation END AS cancellation
    FROM pizza_runner.runner_orders;
    
    ALTER TABLE pizza_runner.runner_orders_temp
    MODIFY COLUMN pickup_time DATETIME,
    MODIFY COLUMN duration INT,
    MODIFY COLUMN distance FLOAT;
    
    ALTER TABLE pizza_runner.customer_orders_temp
    MODIFY COLUMN order_time DATETIME;

---

**Query #1**
~~~~sql
    SELECT COUNT(*) AS order_count
    FROM pizza_runner.customer_orders_temp;
~~~~

| order_count |
| ----------- |
| 14          |

---
**Query #2**
~~~~sql
    SELECT COUNT(DISTINCT order_id) AS unique_order_count
    FROM pizza_runner.customer_orders_temp;
~~~~
| unique_order_count |
| ------------------ |
| 10                 |

---
**Query #3**
~~~~sql
    SELECT runner_id, COUNT(*) AS delivered_by_runner
    FROM pizza_runner.runner_orders_temp
    WHERE pickup_time IS NOT NULL
    GROUP BY runner_id;
~~~~
| runner_id | delivered_by_runner |
| --------- | ------------------- |
| 1         | 4                   |
| 2         | 3                   |
| 3         | 1                   |

---
**Query #4**
~~~~sql
    SELECT pizza_id, COUNT(*) AS delivered_by_type
    FROM pizza_runner.customer_orders_temp
    GROUP BY pizza_id;
~~~~
| pizza_id | delivered_by_type |
| -------- | ----------------- |
| 1        | 10                |
| 2        | 4                 |

---
**Query #5**
~~~~sql
    SELECT customer_id, pizza_name, COUNT(*) AS order_times
    FROM pizza_runner.customer_orders_temp
    JOIN pizza_runner.pizza_names USING (pizza_id)
    GROUP BY customer_id, pizza_name
    ORDER BY customer_id, pizza_name;
~~~~
| customer_id | pizza_name | order_times |
| ----------- | ---------- | ----------- |
| 101         | Meatlovers | 2           |
| 101         | Vegetarian | 1           |
| 102         | Meatlovers | 2           |
| 102         | Vegetarian | 1           |
| 103         | Meatlovers | 3           |
| 103         | Vegetarian | 1           |
| 104         | Meatlovers | 3           |
| 105         | Vegetarian | 1           |

---
**Query #6**
~~~~sql
    SELECT customer_orders_temp.order_id, COUNT(customer_orders_temp.order_id) AS order_count
    FROM pizza_runner.customer_orders_temp
    JOIN pizza_runner.runner_orders_temp USING (order_id)
    WHERE runner_orders_temp.cancellation IS NULL
    GROUP BY customer_orders_temp.order_id
    ORDER BY order_count DESC
    LIMIT 1;
~~~~
| order_id | order_count |
| -------- | ----------- |
| 4        | 3           |

---
**Query #7**
~~~~sql
    SELECT 
    	customer_orders_temp.customer_id, 
    	SUM(CASE 
    		WHEN exclusions IS NULL AND extras IS NULL THEN 1
            ELSE 0 END) AS no_changes,
        SUM(CASE 
        	WHEN exclusions IS NOT NULL OR extras IS NOT NULL THEN 1
        	ELSE 0 END) AS at_least_one_change
    FROM pizza_runner.customer_orders_temp
    JOIN pizza_runner.runner_orders_temp USING (order_id)
    WHERE runner_orders_temp.cancellation IS NULL 
    GROUP BY customer_orders_temp.customer_id;
~~~~
| customer_id | no_changes | at_least_one_change |
| ----------- | ---------- | ------------------- |
| 101         | 2          | 0                   |
| 102         | 3          | 0                   |
| 103         | 0          | 3                   |
| 104         | 1          | 2                   |
| 105         | 0          | 1                   |

---
**Query #8**
~~~~sql
    SELECT 
    	SUM(CASE 
    		WHEN exclusions IS NOT NULL AND extras IS NOT NULL THEN 1
            ELSE 0 END) AS pizza_count_exclusion_extra
    FROM pizza_runner.customer_orders_temp
    JOIN pizza_runner.runner_orders_temp USING (order_id)
    WHERE runner_orders_temp.cancellation IS NULL;
~~~~
| pizza_count_exclusion_extra |
| --------------------------- |
| 1                           |

---
**Query #9**
~~~~sql
    SELECT HOUR(order_time) AS `hour`, COUNT(order_time) AS pizza_count
    FROM pizza_runner.customer_orders_temp
    GROUP BY HOUR(order_time)
    ORDER BY `hour`;
~~~~
| hour | pizza_count |
| ---- | ----------- |
| 11   | 1           |
| 13   | 3           |
| 18   | 3           |
| 19   | 1           |
| 21   | 3           |
| 23   | 3           |

---
**Query #10**
~~~~sql
    SELECT 
    	DAYNAME(order_time) AS weekday,
        COUNT(order_id) AS order_count
    FROM pizza_runner.customer_orders_temp
    GROUP BY DAYNAME(order_time);
~~~~
| weekday   | order_count |
| --------- | ----------- |
| Wednesday | 5           |
| Thursday  | 3           |
| Saturday  | 5           |
| Friday    | 1           |

---

