Team member: Mahmoud Adel Mohamed      and   heba Mohamed alsaad 
Story of dashboard
1-Strategic price optimization can significantly improve profit margins without hurting sales  by leveraging product elasticity insights and competitive pricing intelligence.

2-Determine the business questions behind this message to be answered
Page 1: Performance Overview
Theme: Revenue, Profit, Quantity, and Product/Category Performance
Business Questions:
1.	What are our top revenue-generating product categories and individual products?
2.	Which products deliver the highest profit margins?
3.	Which product has the highest total quantity sold?
4.	How does product weight or rating (score) affect sales volume?
5.	Which product categories contribute the most to overall revenue?
________________________________________
Page 2: Price Elasticity & Sales Dynamics
Theme: Demand Sensitivity, Price Impact, Product Clustering
Business Questions:
6.	Which products are price-sensitive (elastic) vs. price-insensitive (inelastic)?
7.	What pricing recommendations can we make based on elasticity analysis?
8.	How does the average price trend over time affect total quantity sold?
9.	Are there clusters of products where small price changes could significantly impact demand?
________________________________________
Page 3: Competitive Pricing Analysis
Theme: Competitor Benchmarking, Pricing Strategy
Business Questions:
10.	Are our product prices higher, lower, or equal to competitors’?
11.	Which products should we consider increasing or decreasing prices for?
12.	What is the relationship between our prices and competitors’ average prices?
13.	How many products are priced competitively, and how many are misaligned?
14.	What factors most influence our unit prices (Key Influencers)?

3-Write at least 10 SQL queries (including analytical functions) to answer these business
Questions
--1
--تصنيف المنتجات الاكثر دخلا 
with catg_sales as (
select product_category_name,product_id , sum(total_price) as total_rev,sum(qty) as QTY_sold
from product_sales
group by product_category_name ,product_id
)
select product_id,total_rev,QTY_sold,
rank() over(order by total_rev desc) as rank_rev,
rank() over(order by QTY_sold desc ) as rank_sold_QTY
from catg_sales

--2
--تصنيف المنتجات الاكثر طلبا 
with order_count as(
select product_category_name,product_id,sum(customers) as Total_orders,sum(total_price) as total_rev
from product_sales
group by product_category_name,product_id
)
select product_category_name,product_id,Total_orders,total_rev,
dense_rank() over(order by Total_orders desc) as rank_most_ordered_item
from order_count

--3
--تاثير السعر علي المييعات 
select product_id,avg(unit_price) as agg_sales,sum(qty) as qty_sold,
corr(unit_price,qty) as price_elasticity,
case
  when corr(unit_price,qty)< -0.3 then 'highly elastic'
  when corr(unit_price,qty)< 0 then 'elastic'
  when corr(unit_price,qty)= 0 then 'Neutral'
  else
  'Inelastic'
  END AS elasticity_category
from product_sales
group by product_id

--4
--تاثير الوزن ع ميبعات الشركة 
select count(*) as prodcut_count,sum(total_price) as total_rev,sum(freight_price)as total_fright,sum(total_price-freight_price) as total_profit,
case 
when product_weight_g < 500 then 'light weight'
when product_weight_g < 2000 then 'medium weight'
else 'heavy wight'
end as weigth_catg
from product_sales
group by weigth_catg

--5
--ترتيب اكثر المنتجات تقييما
select product_id,avg(product_score) as avg_rate,sum(total_price) as total_revnue, dense_rank() over(order by avg(product_score)desc)
from product_sales
group by product_id

--6
-- تحليل المبيعات بالنسين السابقة وتغيرها 
with sal_year as(
SELECT 
  product_category_name,
  year,
  month,
  SUM(total_price-freight_price) AS total_profit
FROM product_sales
GROUP BY 
  product_category_name,
  year,
  month
ORDER BY 
  product_category_name, year, month
)
select product_category_name,year, sum(total_profit) as tota_profi_current , lag(sum(total_profit)) over(partition by product_category_name order by year) as perv_year_profit,
sum(total_profit)- lag(sum(total_profit)) over(partition by product_category_name order by year) as changes
from sal_year
group by product_category_name,year
order by product_category_name,year

--7
-- تغيرات السعر
select product_id,year, month,
unit_price,lead(unit_price) over(partition by product_id order by year)as  next_price,
unit_price-lead(unit_price) over(partition by product_id order by year) as change_price
from product_sales

--8
--مقارنة اسعار تكلفة الشحن 
SELECT product_id, extract(year from to_date(month_year,'DD-MM-YYYY')) as year,extract(month from to_date(month_year,'DD-MM-YYYY')) as month, freight_price,
least(fp1,fp2,fp3)as min_price_competitor ,
freight_price-least(fp1,fp2,fp3)as price_different
FROM product_sales

--9
--مقارنة اسعار المنتجات 
with pric_product as (
SELECT product_id,product_category_name, extract(year from to_date(month_year,'DD-MM-YYYY')) as year,extract(month from to_date(month_year,'DD-MM-YYYY')) as month, unit_price,
least(comp_1,comp_2,comp_3)as min_price_competitor ,
unit_price-least(comp_1,comp_2,comp_3)as price_different
FROM product_sales
)
select product_id,avg(unit_price) as price , avg(min_price_competitor) as avg_min_price_competitor,avg(unit_price)-avg(min_price_competitor) as diff_price
from pric_product
group by product_id

![image](https://github.com/user-attachments/assets/8c94f70d-5f2c-4773-bed5-53b1ea36ad23)


![image](https://github.com/user-attachments/assets/aa7f1e8b-a5ca-451d-8478-36d518985203)


![image](https://github.com/user-attachments/assets/0a6c9c46-fb22-4a04-ab09-5b53a8ea5a45)

