# Case Study #1: Danny's Diner 🍜

You can view the full case study [HERE](https://8weeksqlchallenge.com/case-study-1/)! In all honesty, I do refer to some online resources when I was working on this case study. However, I tried my best to limit my reference and to write my own code as much as possible. This is my first SQL project/case study that I worked on, hopefully on the next one I won't have to depend on any reference as much.

Now, let's jump directly into the questions, ```coding``` and solutions 💻📝

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

- For this question, the ```SUM``` funtion was used to add up the total amount spent for each customer

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

- We use the ```COUNT``` funtion on ```order_date``` to count the number of days each customer visited.
- It's important to include ```DISTINCT``` in this case to avoid multiple count of the same date.

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

- For this question, we use ```DENSE_RANK``` to identify which product was purchased first based on each customer's ```order_date```
- ```PARTION BY``` over ```customer_id``` so the ranks will be organized for each customer
- The table will only show the first item purchased when we set ```item_rank=1```

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

- Use ```COUNT``` function to identify how many times each product was purchased
- ```ORDER BY ... DESC``` and ```LIMIT 1``` were set to only show the most purchased product

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

- ```DENSE_RANK``` were used again in combination with ```COUNT``` funtion to rank up the most purchased item for each customer
- We set ```rank_num = 1``` to only show the most purchased item
- Since customer B have purchased the same amount of times for all product, hence all product will be ranked as 1 and listed in the table.

### 6. Which item was purchased first by the customer after they became a member?
```sql
SELECT 	customer_id,
	product_name
        
FROM 
	(SELECT sales.customer_id,
         	sales.order_date,
         	sales.product_id,
         	DENSE_RANK () OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date) AS order_date_rank
         
         FROM dannys_diner.sales
         JOIN dannys_diner.members ON sales.customer_id = members.customer_id
         WHERE sales.order_date >= members.join_date) AS member_rank_table
         
JOIN dannys_diner.menu ON member_rank_table.product_id = menu.product_id
WHERE order_date_rank = 1
ORDER BY customer_id;
```

#### SOLUTION
| Customer_ID | Product_Name |
|-------------|--------------|
|A            |Curry         |
|B            |Sushi         |

- The ```WHERE``` funtion was used to set a condition of ```order_date >= join_date```, this is because we only want to see the first item purchased AFTER they have registered as a member.

### 7. Which item was purchased just before the customer became a member?
```sql
SELECT 	customer_id,
	product_name
        
FROM 
	(SELECT sales.customer_id,
         	sales.order_date,
         	sales.product_id,
         	DENSE_RANK () OVER (PARTITION BY sales.customer_id ORDER BY sales.order_date DESC) AS order_date_rank
         
         FROM dannys_diner.sales
         JOIN dannys_diner.members ON sales.customer_id = members.customer_id
         WHERE sales.order_date < members.join_date) AS member_rank_table
         
JOIN dannys_diner.menu ON member_rank_table.product_id = menu.product_id
WHERE order_date_rank = 1
ORDER BY customer_id;
```

#### SOLUTION
| Customer_ID | Product_Name |
|-------------|--------------|
|A            |Sushi         |
|A            |Curry         |
|B            |Sushi         |

-Similar to previous question, but instead the ```WHERE``` condition were set as ```order_date < join_date``` and ```DESC``` were added in the ```ORDER BY``` call in order to only view the item purchased BEFORE the customer registered for membership.

### 8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT 	sales.customer_id,
	COUNT (menu.product_id) AS total_item,
        SUM (menu.price) AS total_amount
        
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu ON sales.product_id = menu.product_id
INNER JOIN dannys_diner.members ON members.customer_id = sales.customer_id
WHERE sales.order_date < members.join_date
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

#### SOLUTION
| Customer_ID | Total_Item | Total_Amount | 
|-------------|------------|--------------|
|A            |2           |25            |
|A            |3           |40            |

- We use ```COUNT``` function to count how many times a product has been purchased by each customer
- ```SUM``` to add up the total amount spent for each customer
- What's important here is the ```WHERE``` function which I used to set a condition of ```order_date < join_date``` in order to only view the items purchased BEFORE membership.

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
SELECT 	customer_id,
	SUM ( CASE 	WHEN product_name = 'sushi' 
             		THEN 20*price 
             		ELSE 10*price END) AS total_point
                    
FROM dannys_diner.sales
JOIN dannys_diner.menu ON sales.product_id = menu.product_id
GROUP BY customer_id
ORDER BY customer_id;
```

#### SOLUTION
| Customer_ID | Total_Point |
|-------------|-------------|
|A            |860          |
|B            |940          |
|C            |360          |

- Since we have a condition of 10 points for all items except of sushi which have a 2x multiplier, we us ```CASE``` to set the condition.

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
SELECT 	sales.customer_id,
	SUM (CASE 	
             	WHEN (sales.order_date >= members.join_date) AND (sales.order_date < members.join_date+7) THEN 20*menu.price
               	WHEN menu.product_name = 'sushi' THEN 20*menu.price
             	ELSE 10*menu.price
             END) AS total_points
             
FROM dannys_diner.sales
JOIN dannys_diner.menu ON sales.product_id = menu.product_id
JOIN dannys_diner.members ON sales.customer_id = members.customer_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```
#### SOLUTION
| Customer_ID | Total_Point |
|-------------|-------------|
|A            |1370         |
|B            |940          |

- This is where it gets tricky, we add another condition ```AND``` into our ```CASE``` as we only allow the 2x point multiplier freebies happening tduring the first week of membership.
- But you have to remember, when ```AND``` is used, both condition has to be met or else the code will proceed to the next condition. 
