# Case Study #1: Danny's Diner

### 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT 	customer_id,
	CONCAT ('$', SUM(price)) AS total_amount
        
FROM dannys_diner.sales
JOIN dannys_diner.menu ON sales.product_id = menu.product_id
GROUP BY customer_id
ORDER BY customer_id;
```

#### SOLUTION

| Customer_ID | Total_Amount |
|-------------|--------------|
|A            |$76           |
|B            |$74           |
|C            |$36           |

### 2. How many days has each customer visited the restaurant?

```sql
SELECT 	customer_id,
	COUNT (DISTINCT order_date) AS days_vistied
        
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```

#### SOLUTION

| Customer_ID | Days_Visited |
|-------------|--------------|
|A            |4             |
|B            |6             |
|C            |2             |

### 3. What was the first item from the menu purchased by each customer?

```sql
SELECT 	customer_id,
	product_name
        
FROM
	(SELECT customer_id,
         	order_date,
         	product_name,
         	DENSE_RANK () OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date) AS item_rank
         FROM dannys_diner.sales
         JOIN dannys_diner.menu ON sales.product_id = menu.product_id) AS first_item_table
WHERE item_rank=1
ORDER BY customer_id;
```
#### SOLUTION

| Customer_ID | Product_Name |
|-------------|--------------|
|A            |Curry         |
|A            |Sushi         |
|B            |Curry         |
|C            |Ramen         |
|C            |Ramen         |

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT 	product_name,
	COUNT (sales.product_id) AS order_count
      
FROM dannys_diner.menu
JOIN dannys_diner.sales ON menu.product_id = sales.product_id
GROUP BY product_name
ORDER BY order_count DESC
LIMIT 1;
```
#### SOLUTION
| Product_Name | Order_Count |
|--------------|-------------|
|Ramen         |8            |

### 5. Which item was the most popular for each customer?
```sql
SELECT 	customer_id,
	product_name
FROM
	(SELECT customer_id,
         	product_name,
         	COUNT(product_name) AS order_count,
         	DENSE_RANK () OVER (PARTITION BY customer_id ORDER BY COUNT(product_name) DESC) AS rank_num
         
         FROM dannys_diner.sales
         JOIN dannys_diner.menu ON sales.product_id = menu.product_id
         GROUP BY customer_id, product_name) AS order_info
         
WHERE rank_num = 1;
```

#### SOLUTION
| Customer_ID | Product_Name |
|-------------|--------------|
|A            |Ramen         |
|B            |Ramen         |
|B            |Curry         |
|B            |Sushi         |
|C            |Ramen         |

### 6. Which item was purchased first by the customer after they became a member?
