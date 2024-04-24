# Zomato_Dataset_Analysis_with_SQL
#### This is an Practice analysis on food delivery platform Zomato. The data has been put manually with the help of SQL. Here you will find the code for the dataset and analysis.
#### The project demonstrates how to approach and answer various business problems with the help of SQL.

---
### Answering Business Questions with SQL
---
### what is the total amount spent by each customer? 

select sales.userid,sum(product.price) as total_amount
from sales
join product on sales.product_id = product.product_id
group by userid

### How many days each customer visited zomato? 

select userid,count( distinct created_date) as number_of_days
from sales
group by userid

### what was the first product purchased by each customer? 

select * from
(select *, rank() over(partition by userid order by created_date) as rnk from sales) a where rnk=1

### what is the most purchased item on the menu and how many times it was purchased by each customer? 

select userid, count(product_id) cnt from sales where product_id =
(select top 1 product_id from sales group by product_id order by count(product_id) desc)
group by userid

### What item is most popular for each customer? 

select * from
(select *, rank() over(partition by userid order by cnt desc) as rnk from
(select userid, product_id, count(product_id) as cnt from sales group by userid,product_id)a)b
where rnk = 1

### which item was purchased first by the customer after they became a member? 

select * from
(select c.*, rank() over(partition by userid order by created_date) rnk from
(select a.userid,a.created_date,a.product_id,b.gold_signup_date from sales a
inner join goldusers_signup b on a.userid = b.userid and created_date>= gold_signup_date) c)d where rnk = 1

### What item was purchased just before the customer became a member? 

select * from
(select c.*, rank() over(partition by userid order by created_date desc) rnk from
(select a.userid,a.created_date,a.product_id,b.gold_signup_date from sales a
inner join goldusers_signup b on a.userid = b.userid and created_date<= gold_signup_date) c)d where rnk = 1

### what is the total order and amount spent for each member before they become a member ?

select userid, count(created_date) total_order , sum(price) total_amt_spent from
(select c.*, d.price from
(select a.userid,a.created_date,a.product_id,b.gold_signup_date from sales a
inner join goldusers_signup b on a.userid = b.userid and created_date<= gold_signup_date) c inner join product d on c.product_id = d.product_id) e
group by userid

### If buying each product generates points for eg. 5rs = 1 zomato point and each product has different purchasing points 
###for eg for P1 5rs =1 zomato point, P2 10rs = 5 zomato point, P3 5rs = 1 zomato point,
  
 ### calculate points collected by each customers and for which products most points has been given till now ? 

select userid, sum(total_points) as total_points_earned from
(select e.*, amt/points total_points from
(select d.*, case when product_id=1 then 5 when product_id = 2 then 2 when product_id = 3 then 5 else 0 end as points from
(select c.userid, c.product_id, sum(price) amt from
(select a.*, b.price from sales a inner join product b on a.product_id = b.product_id) c
group by userid,product_id) d) e) f group by userid


select * from
(select *, rank() over(order by total_points_earned desc) rnk from
(select product_id, sum(total_points) as total_points_earned from
(select e.*, amt/points total_points from
(select d.*, case when product_id=1 then 5 when product_id = 2 then 2 when product_id = 3 then 5 else 0 end as points from
(select c.userid, c.product_id, sum(price) amt from
(select a.*, b.price from sales a inner join product b on a.product_id = b.product_id) c
group by userid,product_id) d) e) f group by product_id) g) h where rnk = 1

### In the first one year after the customer joins the gold program (including their join date) irrespective of what the customer has purchased
they earn 5 zomato points for every 10 rs spent, who earned more 1 or 3 and what was their point earnings in the first year?

###5 zp = 10 rs
###1 zp = 2 rs
###1 rs = 0.5 zp


select c.userid,d.price*0.5 total_points_earned from 
(select a.userid,a.created_date,a.product_id,b.gold_signup_date from sales a
inner join goldusers_signup b on a.userid = b.userid and created_date>= gold_signup_date and created_date<= dateadd(year,1,gold_signup_date)) c 
inner join product d on c.product_id = d.product_id

### rank all the transcations made by customers 

select *, rank() over( partition by userid order by created_date) rnk from sales

### rank all the transcations whenever they are gold member, for every non member mark as na 

select e.*, case when rnk = 0 then 'na' else rnk end as rrnk from
(select c.*,cast((case when gold_signup_date is null then 0 else rank() over(partition by userid order by created_date desc) end) as varchar) rnk from
(select a.userid,a.created_date,a.product_id,b.gold_signup_date from sales a
left join goldusers_signup b on a.userid = b.userid and created_date>= gold_signup_date) c)e
