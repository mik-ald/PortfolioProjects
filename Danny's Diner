/* --------------------
   Case Study Schema
   --------------------*/

CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
VALUES
  ('A', '2021-01-01', '1'),
  ('A', '2021-01-01', '2'),
  ('A', '2021-01-07', '2'),
  ('A', '2021-01-10', '3'),
  ('A', '2021-01-11', '3'),
  ('A', '2021-01-11', '3'),
  ('B', '2021-01-01', '2'),
  ('B', '2021-01-02', '2'),
  ('B', '2021-01-04', '1'),
  ('B', '2021-01-11', '1'),
  ('B', '2021-01-16', '3'),
  ('B', '2021-02-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-01', '3'),
  ('C', '2021-01-07', '3');
 

CREATE TABLE menu (
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');


/* --------------------
   Case Study Questions
   --------------------*/

-- 1. What is the total amount each customer spent at the restaurant?

SELECT customer_id, SUM(m.price) AS total_spent
FROM dannys_diner.sales AS s
   JOIN dannys_diner.menu AS m ON s.product_id = m.product_id
GROUP BY customer_id;

-- 2. How many days has each customer visited the restaurant?

SELECT customer_id, COUNT(DISTINCT order_date)
FROM dannys_diner.sales
GROUP BY customer_id;

-- 3. What was the first item from the menu purchased by each customer?

SELECT sub.customer_id, m.product_name 
FROM (
      SELECT 
         customer_id, 
         order_date, 
         product_id,
         ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date) row_number
      FROM dannys_diner.sales
   ) sub
   JOIN dannys_diner.menu m ON sub.product_id = m.product_id
WHERE row_number = 1;

-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

SELECT m.product_name, COUNT(s.product_id) times_purchased
FROM dannys_diner.sales s
   JOIN dannys_diner.menu m ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY times_purchased Desc
Limit 1;

-- 5. Which item was the most popular for each customer?

SELECT * 
FROM (
   SELECT 
      customer_id, product_id, COUNT(*) AS total,
      ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY COUNT(*) DESC) AS item_rank
   FROM dannys_diner.sales
   GROUP BY customer_id, product_id) sub
WHERE item_rank = 1;

-- 6. Which item was purchased first by the customer after they became a member?

SELECT *
FROM (
   SELECT 
      s.customer_id, s.order_date, s.product_id, m.join_date,
      ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY order_date ASC) AS first_items
   FROM dannys_diner.sales s 
      JOIN dannys_diner.members m ON s.customer_id = m.customer_id
   WHERE order_date >= join_date
  ) sub
WHERE first_items = 1;

-- 7. Which item was purchased just before the customer became a member?

SELECT *
FROM (
   SELECT 
      s.customer_id, s.order_date, s.product_id, m.join_date,
      ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY order_date DESC) AS first_items
   FROM dannys_diner.sales s 
      JOIN dannys_diner.members m ON s.customer_id = m.customer_id
   WHERE order_date < join_date
  ) sub
WHERE first_items = 1;

-- 8. What is the total items and amount spent for each member before they became a member?

SELECT s.customer_id, COUNT(*), SUM(menu.price)
FROM dannys_diner.sales s
   JOIN	dannys_diner.menu menu ON s.product_id = menu.product_id
   JOIN dannys_diner.members members ON s.customer_id = members.customer_id
WHERE s.order_date < members.join_date
GROUP BY s.customer_id

-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

SELECT s.customer_id, SUM(sub.points)
FROM dannys_diner.sales s
JOIN (
   SELECT product_id, 
      CASE product_id
         WHEN 1 THEN price * 2
         ELSE price
      END AS points
   FROM dannys_diner.menu) sub ON s.product_id = sub.product_id
 GROUP BY s.customer_id


-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

WITH dates_cte AS 
(
 SELECT *, 
  DATEADD(DAY, 6, join_date) AS valid_date, 
  EOMONTH('2021-01-31') AS last_date
 FROM members AS m
)

SELECT d.customer_id, s.order_date, d.join_date, 
 d.valid_date, d.last_date, m.product_name, m.price,
 SUM(CASE
  WHEN m.product_name = 'sushi' THEN 2 * 10 * m.price
  WHEN s.order_date BETWEEN d.join_date AND d.valid_date THEN 2 * 10 * m.price
  ELSE 10 * m.price
  END) AS points
FROM dates_cte AS d
JOIN sales AS s ON d.customer_id = s.customer_id
JOIN menu AS m ON s.product_id = m.product_id
WHERE s.order_date < d.last_date
GROUP BY d.customer_id, s.order_date, d.join_date, d.valid_date, d.last_date, m.product_name, m.price

