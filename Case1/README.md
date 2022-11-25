1. What is the total amount each customer spent at the restaurant?
```sql
SELECT dannys_diner.sales.customer_id as "Customer", SUM(dannys_diner.menu.price) as "Total Spent"
FROM dannys_diner.sales
JOIN dannys_diner.menu ON dannys_diner.menu.product_id = dannys_diner.sales.product_id
GROUP BY dannys_diner.sales.customer_id
ORDER BY dannys_diner.sales.customer_id
```


| Customer | Total Spent |
| -------- | ----------- |
| A        | 76          |
| B        | 74          |
| C        | 36          |


---

2. How many days has each customer visited the restaurant?

```sql
SELECT dannys_diner.sales.customer_id as "Customer", COUNT(DISTINCT(dannys_diner.sales.order_date)) as "Number of days visited"
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu ON dannys_diner.menu.product_id = dannys_diner.sales.product_id
GROUP BY dannys_diner.sales.customer_id
```


| Customer | Number of days visited |
| -------- | ---------------------- |
| A        | 4                      |
| B        | 6                      |
| C        | 2                      |

---

3. What was the first item from the menu purchased by each customer?

```sql
WITH TABLE2 AS (SELECT dannys_diner.sales.customer_id, dannys_diner.sales.product_id
FROM dannys_diner.sales
INNER JOIN
(
	SELECT dannys_diner.sales.customer_id, MIN(dannys_diner.sales.order_date)
	FROM dannys_diner.sales
	INNER JOIN dannys_diner.menu ON dannys_diner.menu.product_id = dannys_diner.sales.product_id
	GROUP BY dannys_diner.sales.customer_id
) 
TABLE1
ON dannys_diner.sales.order_date = TABLE1.min AND dannys_diner.sales.customer_id = TABLE1.customer_id
)

SELECT TABLE2.customer_id, menu.product_name
FROM TABLE2
INNER JOIN dannys_diner.menu ON dannys_diner.menu.product_id = TABLE2.product_id
ORDER BY customer_id
```


| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | curry        |
| C           | ramen        |
| C           | ramen        |

---


4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT dannys_diner.menu.product_name, COUNT(dannys_diner.sales.order_date) as most_purchased
FROM dannys_diner.sales
JOIN dannys_diner.menu ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY dannys_diner.menu.product_name
ORDER BY most_purchased DESC
LIMIT 1
```


| product_name | most_purchased |
| ------------ | -------------- |
| ramen        | 8              |

---


5. Which item was the most popular for each customer?


```sql
WITH TEMP AS (SELECT dannys_diner.sales.customer_id, dannys_diner.menu.product_name, COUNT(dannys_diner.sales.order_date) as most_purchased, DENSE_RANK() OVER   
    (PARTITION BY dannys_diner.sales.customer_id ORDER BY COUNT(sales.customer_id) DESC) AS Rank  
FROM dannys_diner.sales
JOIN dannys_diner.menu ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY dannys_diner.sales.customer_id, dannys_diner.menu.product_name
ORDER BY most_purchased DESC)


SELECT customer_id, product_name, most_purchased
FROM TEMP
WHERE rank = 1
ORDER BY customer_id
```



| customer_id | product_name | most_purchased |
| ----------- | ------------ | -------------- |
| A           | ramen        | 3              |
| B           | curry        | 2              |
| B           | sushi        | 2              |
| B           | ramen        | 2              |
| C           | ramen        | 3              |

---
