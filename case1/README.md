
Case Study #1 - Danny's Diner
--
This project is based on the "Danny's Diner" case study from the [8-Week SQL Challenge](https://8weeksqlchallenge.com/case-study-1/).
-- 
## Case Guide

- [Tasks](#tasks)
- [Database Schema](#database-schema)
- [Qeustions and Queries](#qeustions-and-queries)

  
## 1. Tasks
The business task of the "Danny's Diner" case study is to analyze sales and customer data to provide insights into customer spending habits, visit frequency, product popularity, and the impact of the loyalty program. The goal is to help Danny's Diner better understand its customers, optimize its menu, and improve customer retention strategies.

## 2. Database Schema
#### ER Diagram
![plot](https://github.com/edHuang11/sqlproject/blob/main/case1/Diner%20Diagram.drawio.png)
#### Entities and Attributes

##### `sales` Table

-   **customer_id**: `CHAR(1)` - A single character representing the unique ID of a customer.
-   **order_date**: `DATE` - The date on which the order was placed.
-   **product_id**: `INT` - An integer representing the unique ID of the product ordered.

##### `menu` Table

-   **product_id**: `INT` - An integer representing the unique ID of the product.
-   **product_name**: `VARCHAR(5)` - A variable-length string up to 5 characters representing the name of the product.
-   **price**: `INT` - An integer representing the price of the product in dollars.

##### `members` Table

-   **customer_id**: `CHAR(1)` - A single character representing the unique ID of a customer who is a member.
-   **join_date**: `DATE` - The date on which the customer joined the membership program.

## 3. Questions and Queries
#### 1. What is the total amount each customer spent at the restaurant?

    SELECT 
      s.customer_id, 
      SUM(m.price) AS total_sales
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m USING (product_id)
    GROUP BY s.customer_id
    ORDER BY s.customer_id ASC; 
| customer_id | total_sales |
| -- | -- |
| A | 76  |
| B | 74  |
| C | 36  |


#### 2. How many days has each customer visited the restaurant?
    SELECT 
      customer_id, 
      COUNT(DISTINCT order_date) AS visit_count
    FROM dannys_diner.sales
    GROUP BY customer_id;
| customer_id | visit_count |
| ----------- | ----------- |
| A | 4 |
| B | 6 |
| C | 2 |

####  3. What was the first item from the menu purchased by each customer?

    WITH ordered_sales AS (
      SELECT 
        s.customer_id, 
        s.order_date, 
        m.product_name,
        DENSE_RANK() OVER (
          PARTITION BY s.customer_id 
          ORDER BY s.order_date) AS rk
      FROM dannys_diner.sales s
      JOIN dannys_diner.menu m USING (product_id)
    )
    
    SELECT 
      customer_id, 
      product_name
    FROM ordered_sales
    WHERE rk = 1
    GROUP BY customer_id, product_name;
| customer_id | product_name |
| ----------- | ------------ |
| A | sushi  |
| A | curry  |
| B | curry  |
| C | ramen  |

#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?


    WITH order_time_item AS (
        SELECT m.product_name, COUNT(m.product_name) AS order_times
        FROM dannys_diner.sales s
        JOIN dannys_diner.menu m USING (product_id)
        GROUP BY m.product_name
        ORDER BY order_times DESC
        LIMIT 1
    )
    
    -- First SELECT statement using the CTE
    SELECT *
    FROM order_time_item;
    
    -- Second SELECT statement using the CTE
    SELECT s.customer_id, m.product_name, COUNT(*) AS purchase_time
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m USING (product_id)
    WHERE m.product_name = (SELECT product_name FROM order_time_item)
    GROUP BY s.customer_id, m.product_name;


| product_name | most_purchased_item |
| ------------ | ------------------- |
| ramen  | 8 |

| customer_id | product_name | purchase_time|
|--|--|--|
| A | sushi |1|
| B | sushi |2|


#### 5. Which item was the most popular for each customer?

    WITH order_counts AS (
      SELECT s.customer_id, s.product_id, COUNT(*) AS order_count
      FROM dannys_diner.sales s
      JOIN dannys_diner.menu m USING (product_id)
      GROUP BY s.customer_id, s.product_id
      ORDER BY s.customer_id, s.product_id
      )
      
    SELECT *
    FROM order_counts
    WHERE (customer_id, order_count) IN (
      SELECT customer_id, MAX(order_count) 
      FROM order_counts
      GROUP BY customer_id);

| customer_id | product_name | order_count |
| ----------- | ------------ | ----------- |
| A | ramen  | 3 |
| B | curry  | 2 |
| B | sushi  | 2 |
| B | ramen  | 2 |
| C | ramen  | 3 |

#### 6. Which item was purchased first by the customer after they became a member?

    WITH order_after AS(
    SELECT
      s.customer_id, 
      s.order_date, 
      m.product_name
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m USING (product_id)
    JOIN dannys_diner.members mb ON mb.customer_id = s.customer_id AND s.order_date >= mb.join_date
    ORDER BY s.customer_id,s.order_date)
    
    SELECT *
    FROM order_after
    WHERE (customer_id, order_date) IN (SELECT customer_id, MIN(order_date) FROM order_after GROUP BY customer_id);

| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A | 2021-01-07 | curry  |
| B | 2021-01-11 | sushi  |

#### 7. Which item was purchased just before the customer became a member?

    WITH order_before AS(
    SELECT
      s.customer_id, 
      s.order_date, 
      m.product_name
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m USING (product_id)
    JOIN dannys_diner.members mb ON mb.customer_id = s.customer_id AND s.order_date < mb.join_date
    ORDER BY s.customer_id,s.order_date)
    
    SELECT *
    FROM order_before
    WHERE (customer_id, order_date) IN (SELECT customer_id, MAX(order_date) FROM order_before GROUP BY customer_id);

| customer_id | order_date | product_name |
| ----------- | ---------- | ------------ |
| A | 2021-01-01 | sushi  |
| A | 2021-01-01 | curry  |
| B | 2021-01-04 | sushi  |

#### 8. What is the total items and amount spent for each member before they became a member?  

    WITH order_before AS(
    SELECT
      s.customer_id, 
      s.order_date, 
      m.price
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m USING (product_id)
    JOIN dannys_diner.members mb ON mb.customer_id = s.customer_id AND s.order_date < mb.join_date
    ORDER BY s.customer_id,s.order_date)
    
    SELECT customer_id, COUNT(*) AS total_item, SUM(price) AS total_expense
    FROM order_before
    GROUP BY customer_id
    ORDER BY customer_id;

| customer_id | total_item | total_expense |
| ----------- | ---------- | ------------- |
| A | 2  | 25  |
| B | 3  | 40  |

#### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?  

    SELECT s.customer_id, 
    	SUM(
          CASE 
          WHEN s.product_id =1 THEN m.price * 20
          ELSE m.price * 10 END
        ) AS points
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m USING (product_id)
    GROUP BY s.customer_id;

| customer_id | points |
| ----------- | ------ |
| A | 860  |
| B | 940  |
| C | 360  |

#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January? 

    SELECT
      s.customer_id, 
      SUM(
        CASE 
        WHEN s.order_date BETWEEN mb.join_date AND DATE_ADD(mb.join_date, INTERVAL 6 DAY) THEN m.price * 20
        WHEN s.product_id = 1 THEN m.price * 20
        ELSE m.price * 10 END
        ) AS Jan_points
    FROM dannys_diner.sales s
    JOIN dannys_diner.menu m USING (product_id)
    LEFT JOIN dannys_diner.members mb ON mb.customer_id = s.customer_id 
    WHERE MONTH(order_date) = 1
    GROUP BY s.customer_id;

| customer_id | Jan_points |
| ----------- | ---------- |
| A | 1370 |
| B | 820  |
| C | 360  |

