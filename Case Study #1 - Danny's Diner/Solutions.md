# :ramen: :curry: :sushi: Case Study #1: Danny's Diner

## Case Study Questions

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
***

###  1. What is the total amount each customer spent at the restaurant?

```sql
SELECT 
  sales.customer_id, 
  SUM(menu.price) as "total_amount" 
FROM 
  dannys_diner.sales 
  INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id 
GROUP BY 
  sales.customer_id 
ORDER BY 
  sales.customer_id;
``` 
	
#### Result:
| customer_id | total_amount |
| ----------- | ----------- |
| A           | 76         |
| B           | 74         |
| C           | 36         |

***

###  2. How many days has each customer visited the restaurant?

```sql
SELECT 
  sales.customer_id, 
  COUNT(DISTINCT sales.order_date) as "days_visited" 
FROM 
  dannys_diner.sales 
GROUP BY 
  sales.customer_id 
ORDER BY 
  sales.customer_id;
``` 
	
#### Result:
| customer_id | day_visited |
| ----------- | ----------- |
| A           | 4           |
| B           | 6           |
| C           | 2           |

***

###  3. What was the first item from the menu purchased by each customer?

```sql
SELECT 
  DISTINCT ON (sales.customer_id) sales.customer_id, 
  menu.product_name 
FROM 
  dannys_diner.sales 
  JOIN dannys_diner.menu ON sales.product_id = menu.product_id 
ORDER BY 
  sales.customer_id, 
  sales.order_date, 
  sales.product_id;
``` 
	
#### Result:
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

***
 
###  4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT 
  menu.product_name AS "most_purchased_item", 
  COUNT(*) AS "total_orders" 
FROM 
  dannys_diner.sales 
  JOIN dannys_diner.menu ON sales.product_id = menu.product_id 
GROUP BY 
  menu.product_name 
ORDER BY 
  "total_orders" DESC 
LIMIT 
  1;
``` 
	
#### Result:
| most_purchased_item | total_orders |
| ------------------- | ----------- |
| ramen               | 8           |

***

###  5. Which item was the most popular for each customer?

```sql
SELECT 
  menu.product_name AS "most_purchased_item", 
  COUNT(*) AS "total_orders" 
FROM 
  dannys_diner.sales 
  JOIN dannys_diner.menu ON sales.product_id = menu.product_id 
GROUP BY 
  menu.product_name 
ORDER BY 
  "total_orders" DESC 
LIMIT 
  1;
``` 
	
#### Result:
| customer_id | product_name | most_popular_orders |
| ----------- | ------------ | ----------- |
| A           | ramen        | 3           |
| B           | sushi        | 2           |
| B           | curry        | 2           |
| B           | ramen        | 2           |
| C           | ramen        | 3           |

***

###  6. Which item was purchased first by the customer after they became a member?

```sql
SELECT 
  customer_id, 
  first_order_date, 
  product_name 
FROM 
  (
    SELECT 
      s.customer_id, 
      s.order_date AS first_order_date, 
      m.product_name, 
      ROW_NUMBER() OVER (
        PARTITION BY s.customer_id 
        ORDER BY 
          s.order_date ASC
      ) AS order_number 
    FROM 
      dannys_diner.sales AS s 
      JOIN dannys_diner.menu AS m ON s.product_id = m.product_id 
      JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id 
      AND s.order_date >= mem.join_date
  ) as sub 
WHERE 
  order_number = 1;
``` 
	
#### Result:
| customer_id | first_order_date | order_date|
| ------------ | ------------ | ------------
| A           | 2021-01-07 | curry |
| B           | 2021-01-11 | sushi |

***

###  7. Which item was purchased just before the customer became a member?

```sql
WITH orders as (
  SELECT 
    s.customer_id, 
    M.product_name, 
    s.order_date, 
    mem.join_date, 
    DENSE_RANK() OVER (
      PARTITION BY s.customer_id 
      ORDER BY 
        s.order_date
    ) as rank_num 
  FROM 
    dannys_diner.sales AS s 
    JOIN dannys_diner.menu AS M ON m.product_id = s.product_id 
    JOIN dannys_diner.members AS mem ON mem.customer_id = s.customer_id 
  WHERE 
    s.order_date < mem.join_date
) 
SELECT 
  customer_id, 
  product_name, 
  order_date, 
  join_date 
FROM 
  orders 
WHERE 
  rank_num = 1;
``` 
	
#### Result:
| customer_id | product_name | order_date               | join_date                |
| ----------- | ------------ | ------------------------ | ------------------------ |
| A           | sushi        | 2021-01-01 | 2021-01-07|
| A           | curry        | 2021-01-01 | 2021-01-09 |
| B           | curry        | 2021-01-01 | 2021-01-09|

***

###  8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT 
  s.customer_id, 
  COUNT(s.product_id) AS total_items, 
  SUM(m.price) AS total_amount 
FROM 
  dannys_diner.sales s 
  JOIN dannys_diner.menu m ON s.product_id = m.product_id 
  LEFT JOIN dannys_diner.members mem ON s.customer_id = mem.customer_id 
  AND s.order_date >= mem.join_date 
WHERE 
  mem.customer_id IS NULL 
GROUP BY 
  s.customer_id 
ORDER BY 
  s.customer_id;
``` 
	
#### Result:
| customer_id | total_items | total_amount |
| ----------- | ----------- | ------------ |
| A           | 2           | 25          |
| B           | 3           | 40          |
| C           | 3           | 36          |

***

###  9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
SELECT 
  s.customer_id, 
  SUM(
    m.price * CASE WHEN m.product_name = 'sushi' THEN 2 ELSE 1 END
  ) AS total_amount_spent, 
  SUM(
    m.price * CASE WHEN m.product_name = 'sushi' THEN 2 ELSE 1 END
  ) * 10 AS total_points 
FROM 
  dannys_diner.sales s 
  JOIN dannys_diner.menu m ON s.product_id = m.product_id 
GROUP BY 
  s.customer_id 
ORDER BY 
  s.customer_id;
``` 
	
#### Result:
| customer_id | total_amount_spent | total_points |
| ----------- | ----------- | ------------ |
| A           | 86           | 860          |
| B           | 94          | 940          |
| C           | 36           | 360          |

***

###  10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January

```sql
WITH members AS (
  SELECT 
    s.customer_id, 
    s.product_id, 
    order_date, 
    join_date 
  FROM 
    dannys_diner.sales s 
    INNER JOIN dannys_diner.members m ON s.customer_id = m.customer_id 
    AND s.order_date >= m.join_date
) 
SELECT 
  customer_id, 
  SUM(
    CASE WHEN order_date < join_date + INTERVAL '7 day' THEN price * 20 WHEN product_name = 'sushi' THEN price * 20 ELSE price * 10 END
  ) AS points 
FROM 
  members 
  INNER JOIN dannys_diner.menu USING (product_id) 
WHERE 
  order_date <= '2021-01-31' 
GROUP BY 
  customer_id 
ORDER BY 
  customer_id;
``` 

#### Result:
| customer_id | points |
| ----------- | --------------- |
| A           | 1020            |
| B           | 320             |

***

###  Bonus Questions

#### Join All The Things

```sql
SELECT 
  s.customer_id, 
  s.order_date, 
  M.product_name, 
  M.price, 
  CASE WHEN s.order_date >= mem.join_date THEN 'Y' ELSE 'N' END AS member 
FROM 
  dannys_diner.members AS mem 
  RIGHT JOIN dannys_diner.sales AS s ON s.customer_id = mem.customer_id 
  INNER JOIN dannys_diner.menu AS M ON M.product_id = s.product_id 
ORDER BY 
  s.customer_id, 
  s.order_date, 
  M.product_name;
``` 
	
#### Result:
![image](https://user-images.githubusercontent.com/77529445/167406964-25276db9-fe1c-4608-8b77-b0970b156888.png)

***

#### Rank All The Things

```sql
WITH member_check AS(
  SELECT 
    s.customer_id, 
    s.order_date, 
    product_name, 
    price, 
    CASE WHEN s.order_date >= join_date THEN 'Y' ELSE 'N' END AS member 
  FROM 
    dannys_diner.sales s 
    LEFT JOIN dannys_diner.menu as m ON s.product_id = m.product_id 
    LEFT JOIN dannys_diner.members AS mem ON s.customer_id = mem.customer_id
) 
SELECT 
  *, 
  CASE WHEN member = 'N' THEN NULL ELSE RANK() OVER(
    PARTITION BY customer_id, 
    member 
    ORDER BY 
      order_date
  ) END AS ranking 
FROM 
  member_check 
ORDER BY 
  customer_id, 
  order_date;
``` 

#### Result:
![image](https://user-images.githubusercontent.com/77529445/167407504-41d02dd0-0bd1-4a3c-8f41-00ae07daefad.png)

***


