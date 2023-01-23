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
