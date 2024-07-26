# Customer Behaviour Analysis

## Table of contents

- [Problem Statement](#problem-statement)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- [Questions And SQL Queries](#questions-and-SQL-queries)
- [Results/Findings](#resultsfindings)
- [Recommendations](#recommendations)
- [Conclusion](#conclusion)

### Problem Statement

The company is looking to use its historical orders, customers and membership data to answer some questions, which will provide more insight into the business. The comapany wants to gather information that will help them understand their customers better and thereby improve their customer encounters and offerings. The company is also considering investing in customer membership; therefore, this task is a critical project for the company. 

### Data Sources

Since the main purpose was to execute differents SQL queries, the datesets used was not complex and were manually coded in the query when creating database and tables.  

### Tools

- QuickDBD
- MySQL
  
### Entity Relationship Diagram

![QuickDBD-Resto_schemaDB](https://github.com/JeanBonheur01/MySQL_CustomerBehaviour/assets/131664311/8a34ff3a-941f-4090-9fae-f3d82992e288)

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

<img width="1440" alt="numberofpoints" src="https://github.com/JeanBonheur01/MySQL_CustomerBehaviour/assets/131664311/a9e0fd08-bd2b-448f-aafd-3d8bbc889b73">

11. The ranking of non-member purchases is not needed; it should be NULL for customers who are not yet part of the loyalty program.

<img width="1440" alt="NotNullRanking" src="https://github.com/JeanBonheur01/MySQL_CustomerBehaviour/assets/131664311/b2e98c77-1c0b-406a-929e-eaef011dffcc">

### Results/Findings

- Customer A spends more in the restaurant, while customer B is the most frequent visitor. This does not differ much, as B has made 6 visits while A has made 4 visits.
  
- Customers A and B are also members of the company, where B had more visits and purchases before becomming a menber, while A has been more active after becoming a member.
  
- The restaurant´s most popular item is "Ramen", followed by curry and sushi.
  
- Customer A and C tend to buy more Ramen, while customer B purchases a more equal variety of items.
  
- Customers A has more points in the members club compared to B; customer C is not yet a member.
   
### Recommendations

- Since customers A and B visit and spend more at the restaurant and are already members, the company should invest more on them due to their loyalty and dedication. It's easier and cheaper to retain existing customers than attract new ones. For exmaple, the company could offer them more frequent discounts on their favorit items and improve customer services.
  
- To attracte more customers to its membership group, the company should focus on providing customer-oriented service, e.g to C, to enhance their experience and potentially turn them into future members.
- As "Ramen" is the most popular item, the company should allocate most resources to it to ensure positive cash flow. For instance, always ensuring ingredients for Ramen are available in stock.
- The company should also analyze why other items are not selling as well and determine which strateggies to take to increase the sales of other items on the menu. For exmaple, by improving the recipes and quality of the food, etc.

### Conclusion

In general, the company has to take into consideration these data analystics findings and continue to be data-driven in its processes in order to enhance its decision-making, productivity, and efficiency in the business. This is also crutical for better understanding its customers, gaining competitive advantages, and ultimately being ready to detect new trends and enhance its busniess planning. Utilizing data to understand the business and its customers will enable the restaurant to adapt and offer better service that meets customers' expectations, manage competition in a more suitable and sustainable way, which is crucial for the the company's long-term survival. 

💻🖱️💻🤖💻🖱️💻🤖💻



    
