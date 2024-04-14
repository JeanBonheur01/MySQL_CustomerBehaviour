# Customer Behaviour Analysis

## Table of contents

### Problem Statement
### Entity Relationship Diagram

### Questions And Queries

1. What is the total amount spent by each customer?
```sql
select 
	s.customer_id, sum(m.price) as total_spend
from 
	sales s
join menu m	
on 	m.product_id = s.product_id
group by
	s.customer_id;
```

2. How many days does each customer visit the restaurant?
```sql
select 
	customer_id, count(distinct s.order_date) as days_visited
from 
	sales s
group by 
	customer_id;
```
   
3. What was the first item from the menu purchased by each customer?
```sql
with customer_first_purchase as (
	select 
		s.customer_id, min(s.order_date) as first_purchase_date
	from 
		sales s
	group by 
		s.customer_id)
select 
	cfp.customer_id, cfp.first_purchase_date, m.product_name
from 
	customer_first_purchase cfp
join sales s on s.customer_id = cfp.customer_id
and cfp.first_purchase_date = s.order_date
join menu m on m.product_id = s.product_id;
```
 
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
select 
	m.product_name, count(*) as total_purchased
from 
	sales s
join menu m on s.product_id = m.product_id
group by 
	m.product_name
order by 
	total_purchased desc limit 1;
```

5. Which item was the most popular for each customer?
```
with customer_popularity as (
	select 
		s.customer_id, m.product_name, count(*) as purchase_count, 
		dense_rank() over(partition by s.customer_id order by count(*) desc) as rank_I
	from 
		sales s
	join menu m on s.product_id = m.product_id
	group by 
		s.customer_id, m.product_name) 
select
	cp.customer_id, cp.product_name, cp.purchase_count
from
	customer_popularity cp
where rank_I = 1;
```
NB! Since customer B bought several products the same amount of time, we use the function DENSE_RANK instead of ROW_NUMBER to avoid getting only one ranked purchased product.

6. Which item was purchased first by the customer after they became a member?


8. Which item was purchased just before the customer became a member?


9. What are the total items and amount spent for each member before they became a member?


10. If each $1 spent equates to 10 points and sushi has a 2x points multiplier, how many points would each customer have?


11. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customers A and B have at the end of April?
The company also requires the ranking of products.

<img width="1440" alt="numberofpoints" src="https://github.com/JeanBonheur01/MySQL_CustomerBehaviour/assets/131664311/a9e0fd08-bd2b-448f-aafd-3d8bbc889b73">

11. The ranking of non-member purchases is not needed; it should be NULL for customers who are not yet part of the loyalty program.

<img width="1440" alt="NotNullRanking" src="https://github.com/JeanBonheur01/MySQL_CustomerBehaviour/assets/131664311/b2e98c77-1c0b-406a-929e-eaef011dffcc">




    
