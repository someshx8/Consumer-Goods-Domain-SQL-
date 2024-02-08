1. Provide the list of markets in which customer "Atliq Exclusive" operates its
business in the APAC region.

ANS: select market from dim_customer 
where customer='Atliq Exclusive' and region='APAC'

2. What is the percentage of unique product increase in 2021 vs. 2020? The
final output contains these fields,
unique_products_2020
unique_products_2021
percentage_chg

ANS:  with cte as(
select count(distinct case when fiscal_year=2020 then product_code end) as unique_products_2020,
count(distinct case when fiscal_year=2021 then product_code end) as unique_products_2021
from fact_sales_monthly
)
SELECT unique_products_2020,
	   unique_products_2021,
	   ROUND(((unique_products_2021-unique_products_2020)/unique_products_2020)*100,2) AS percentage_chg
FROM cte;


3.Provide a report with all the unique product counts for each segment and
sort them in descending order of product counts. The final output contains 2 fields,segment product_count

ANS select max(segment) as segment,count(distinct product_code) as product_code from dim_product
group by segment
order by product_code desc;

4.Follow-up: Which segment had the most increase in unique products in 2021 vs 2020? The final output contains these fields,
segment
product_count_2020
product_count_2021
difference


ANS: 
WITH unique_product AS
(
 SELECT
      b.segment AS segment,
      COUNT(DISTINCT
          (CASE 
              WHEN fiscal_year = 2020 THEN a.product_code END)) AS product_count_2020,
       COUNT(DISTINCT
          (CASE 
              WHEN fiscal_year = 2021 THEN a.product_code END)) AS product_count_2021        
 FROM fact_sales_monthly AS a
 INNER JOIN dim_product AS b
 ON a.product_code = b.product_code
 GROUP BY b.segment
)
SELECT segment, product_count_2020, product_count_2021, (product_count_2021-product_count_2020) AS difference
FROM unique_product
ORDER BY difference DESC;



5.Get the products that have the highest and lowest manufacturing costs. The final output should contain these fields,
product_code ,product ,manufacturing_cost

ANS:SELECT  a.product_code AS product_code,
         a.product AS product,
		 CONCAT('$',ROUND(b.manufacturing_cost,2)) AS manufacturing_cost
FROM
dim_product AS a 
INNER JOIN
fact_manufacturing_cost AS b
ON a.product_code = b.product_code /* joining on key ie. product_code*/
WHERE b.manufacturing_cost = (SELECT MAX(manufacturing_cost) FROM fact_manufacturing_cost) 
OR    b.manufacturing_cost = (SELECT MIN(manufacturing_cost) FROM fact_manufacturing_cost) 
ORDER BY b.manufacturing_cost DESC;




6.Generate a report which contains the top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the
Indian market. The final output contains these fields, customer_code customer average_discount_percentage


ANS:  
select dc.customer_code,dc.customer,round(avg(fd.pre_invoice_discount_pct)*100,2)as avg from
dim_customer as dc join
 fact_pre_invoice_deductions as fd
 on dc.customer_code=fd.customer_code
 where fiscal_year=2021 and market='India'
 group by dc.customer_code,dc.customer
 order by avg desc
 limit 5;
 
 
 
 
 7.Get the complete report of the Gross sales amount for the customer “Atliq
Exclusive” for each month. This analysis helps to get an idea of low and
high-performing months and take strategic decisions.
The final report contains these columns:
Month
Year
Gross sales Amount


ANS: select monthname(fs.date) as month ,year(fs.date) as year ,
round(sum(fs.sold_quantity*fg.gross_price),2) as gross_sales_amount
from dim_customer as dc join 
fact_sales_monthly as fs on (dc.customer_code=fs.customer_code)
join fact_gross_price as fg on (fs.product_code=fg.product_code)
where customer='Atliq Exclusive'
group by month,year
order by gross_sales_amount desc;





8.In which quarter of 2020, got the maximum total_sold_quantity? The final
output contains these fields sorted by the total_sold_quantity,
Quarter
total_sold_quantity


ANS:select(
case when monthname(date) in ('January','February','March') then 'Q1'
     when monthname(date) in ('April','May','June') then 'Q2'
     when monthname(date) in ('July','August','September') then 'Q3'
     when monthname(date) in ('October','November','December') then 'Q4' 
     end) as Quarter,sum(sold_quantity) as Total_sold_quantity
     from fact_sales_monthly
     where fiscal_year='2020'
     group by Quarter
     order by Total_sold_quantity;
     
9.  Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution? The final output contains these fields,
channel
gross_sales_mln
percentage


ANS:with cte as (
select  dc.channel,round(sum(fs.sold_quantity*fg.gross_price)/1000000,2) as gross_sales_mln
from dim_customer as dc join fact_sales_monthly as fs
on (dc.customer_code=fs.customer_code)
join fact_gross_price as fg
on(fs.product_code=fg.product_code)
where fg.fiscal_year='2021'
group by channel)
select cte.channel,cte.gross_sales_mln,ROUND(cte.gross_sales_mln / SUM(cte.gross_sales_mln) over() * 100, 2) AS percentage
FROM cte;


10. Get the Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021? The final output contains these
fields, division product_code product total_sold_quantity rank_order

ANS: WITH top_sold_products AS 
(
	SELECT b.division AS division,
		   b.product_code AS product_code,
		   b.product AS product,
		   SUM(a.sold_quantity) AS total_sold_quantity
	FROM fact_sales_monthly AS a
	INNER JOIN dim_product AS b
	ON a.product_code = b.product_code
	WHERE a.fiscal_year = 2021
	GROUP BY  b.division, b.product_code, b.product
	ORDER BY total_sold_quantity DESC
),
top_sold_per_division AS 
(
 SELECT division,
	    product_code,
        product,
        total_sold_quantity,
        DENSE_RANK() OVER(PARTITION BY division ORDER BY total_sold_quantity DESC) AS rank_order 
 FROM top_sold_products
 )
 SELECT * FROM top_sold_per_division
 WHERE rank_order <= 3;
     


