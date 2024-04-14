# Customer Behaviour Analysis

## Table of contents

- [Problem Statement](#problem-statement)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Questions And SQL Queries](#questions-and-queries)
- [Results/Findings](#resultsfindings)
- [Recommendations](#recommendations)

### Problem Statement

They company is looking to use its historical orders, customers and membership data to answer some questions, which will provide more insight into the business. The comapany wants to gather information that will help them understand their customers better and thereby improve their customer encounters and offerings. The company is also considering investing in customer membership; therefore, this task is a critical project for the company. 

### Data Sources

Since the main purpose was to execute differents SQL queries, the datesets used was not complex and were manually coded in the query when creating database and tables.  

### Tools

- QuickDBD
- MySQL
  
### Entity Relationship Diagram

### Questions And SQL Queries

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
```sql
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
```sql
with first_purchase_after_membership as (
	select
		s.customer_id, min(s.order_date) as first_purchase_date
	from
		sales s
	join customers cu on s.customer_id = cu.customer_id
	where s.order_date >= cu.join_date
	group by s.customer_id
)
select
	fpam.customer_id, m.product_name
from
	first_purchase_after_membership fpam
join sales s on s.customer_id = fpam.customer_id
and fpam.first_purchase_date = s.order_date
join menu m on s.product_id = m.product_id;
```

7. Which item was purchased just before the customer became a member?
```sql
with last_purchase_before_membership as (
select 
	s.customer_id, max(s.order_date) as last_purchase_date
from
	sales s
join customers cu on s.customer_id = cu.customer_id
where s.order_date < cu.join_date
group by s.customer_id
)
select
	lpbm.customer_id, m.product_name 
from
	last_purchase_before_membership lpbm
join sales s on s.customer_id = lpbm.customer_id
and lpbm.last_purchase_date = s.order_date
join menu m on m.product_id = s.product_id;
```

8. What are the total items and amount spent for each member before they became a member?
```sql
select 
	s.customer_id, count(*) as total_items, sum(m.price) as total_spent
from
	sales s
join menu m on s.product_id = m.product_id
join customers cu on s.customer_id = cu.customer_id
where s.order_date < cu.join_date
group by s.customer_id;
```

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier, how many points would each customer have?
```sql
select s.customer_id, sum(
	case 
		when m.product_name = 'sushi' then m.price*20
        else m.price*10 end) as total_points
from sales s
join menu m on s.product_id = m.product_id
group by s.customer_id;
```

10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customers A and B have at the end of April?
The company also requires the ranking of products.

<img width="1440" alt="numberofpoints" src="https://github.com/JeanBonheur01/MySQL_CustomerBehaviour/assets/131664311/a9e0fd08-bd2b-448f-aafd-3d8bbc889b73">

11. The ranking of non-member purchases is not needed; it should be NULL for customers who are not yet part of the loyalty program.

<img width="1440" alt="NotNullRanking" src="https://github.com/JeanBonheur01/MySQL_CustomerBehaviour/assets/131664311/b2e98c77-1c0b-406a-929e-eaef011dffcc">

### Results/Findings
### Recommendations

💻🖱️💻🤖💻🖱️💻🤖💻



    
