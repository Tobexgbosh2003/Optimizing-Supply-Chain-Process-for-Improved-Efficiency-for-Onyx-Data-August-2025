CREATE TABLE supplychain_sample (
	Product_type varchar(20),	
	SKU	varchar(10),
	Price float,
	Availability int,	
	Number_of_products_sold	 int,
	Revenue_generated float,	
	Customer_demographics varchar(20),
	Stock_levels int,	
	Lead_times	int,
	Order_quantities int,	
	Shipping_times int,	
	Shipping_carriers varchar(20),
	Shipping_costs	float,
	Supplier_name	varchar(20),
	Location	varchar(20),
	Latitide	float,
	Longitude	float,
	Lead_time	int,
	Production_volumes	int,
	Manufacturing_lead_time	int,
	Manufacturing_costs	float,
	Inspection_results varchar(20),	
	Defect_rates float,	
	Transportation_modes varchar(10),	
	Routes	varchar(20),
	Costs float
	);
    
select * 
from supplychain_sample;
  
  
  -- EDA
  
  -- problem statement
  -- 1. what are the most frequently ordered products?
  
  select product_type, count(Number_of_products_sold) as Total_prd
  from supplychain_sample
  group by product_type
  order by 2 desc;
  
  -- 2. what is the total value of inventory on hand 
  
  SELECT 
    product_type, 
    CEIL(SUM(stock_levels * price)) AS inventory_value
FROM supplychain_sample
GROUP BY product_type
ORDER BY inventory_value DESC;

  
  
  -- 3. which supplier has the highest average delivery time
  
  select  distinct Supplier_name, round(avg(Lead_times),3) as AV_DT
  from supplychain_sample
  group by 1
  order by 2 desc 
  limit 2
  ;
  
  -- 4. what is the average cost per unit for each supplier?
  
SELECT 
    supplier_name, 
    CEIL(AVG(manufacturing_costs)) AS avg_cost
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY avg_cost DESC;


-- which supplier has the highest defect rate?

select 
	supplier_name, 
    ceil(SUM(defect_rates)) as Total_defects
    FROM supplychain_sample
GROUP BY supplier_name
ORDER BY 2 DESC;


-- As a CTE

with Cte_example as 
(
select supplier_name, sum(defect_rates) as Defects, 
	RANK() OVER (ORDER BY sum(defect_rates) desc) as Supplier_rank
		FROM supplychain_sample
GROUP BY 1 
)

select supplier_name, ceil(Defects) as total_defect 
from Cte_example
where Supplier_rank  = 1
;


SELECT * FROM
		(SELECT supplier_name, CEIL(SUM(defect_rates)) AS total_defects,
		RANK() OVER (ORDER BY SUM(defect_rates) DESC) AS Supplier_rank
		FROM supplychain_sample
		GROUP BY 1)AS Pop
	WHERE Supplier_rank= 1;



-- 7. Which carrier has the highest on-time delivery rate?

SELECT * FROM 
		(SELECT shipping_carriers, SUM(shipping_times) AS highest,
		ROW_NUMBER () OVER (ORDER BY SUM(shipping_times) DESC) AS best_carrier
		FROM supplychain_sample
		GROUP BY 1) AS Tot
	WHERE best_carrier = 1;

select *
 from supplychain_sample;

-- what is the total number of shipment by carrier?

select Shipping_carriers, count(*)
from supplychain_sample
group by 1
;

-- what is the total revenue generated 

select ceil(sum(Revenue_generated)) as total
from supplychain_sample
;

-- 10. Which product category and customer generates the highest revenue?

SELECT *
FROM (
    SELECT 
        product_type, 
        customer_demographics, 
        CEIL(SUM(revenue_generated)) AS revenue,
        ROW_NUMBER() OVER (
            PARTITION BY customer_demographics 
            ORDER BY SUM(revenue_generated) DESC
        ) AS ranking
    FROM supplychain_sample
    GROUP BY product_type, customer_demographics
) AS derived_table
WHERE ranking = 1;

select * from supplychain_sample;

-- 11. What is the average order value?

	SELECT ROUND(AVG(Order_quantities), 3)
	FROM supplychain_sample;
	
-- 12. Who are the top suppliers by revenue?
	
	WITH Calculation AS 
	(
		SELECT supplier_name, SUM(revenue_generated) AS total_revenue,
		RANK() OVER (ORDER BY SUM(revenue_generated) DESC) AS ranking
		FROM supplychain_sample
		GROUP BY 1
	)
	SELECT supplier_name, CEIL(total_revenue), ranking
	FROM Calculation
	LIMIT 3;
	
-- 13. What is the average order frequency per customer?

	SELECT customer_demographics, ROUND(AVG(order_quantities), 0) as order_freq
	FROM supplychain_sample
	GROUP BY 1
	ORDER BY 1;
	
-- 14. Which location is most profitable?

	SELECT location, CEIL(SUM(revenue_generated)) AS most_profitable
	FROM supplychain_sample
	GROUP BY 1
	ORDER BY 2 DESC;
	
-- 15. Who are the top 10 performing SKU by revenue and product sold?
	
	SELECT sku, 
	       CEIL(SUM(revenue_generated)) AS total_revenue,
		   SUM(number_of_products_sold) AS total_products
	FROM supplychain_sample
	GROUP BY 1
	ORDER BY 2 DESC, 3 DESC
	LIMIT 10;

	
-- 16. Which suppliers have the highest on-time delivery rates and the lowest defect rate

	WITH OnTime AS (
	    SELECT supplier_name, 
	    ROUND(AVG(lead_times),3) AS delivery_rate
	    FROM supplychain_sample
	    GROUP BY 1
	),
	Defects AS (
	    SELECT supplier_name, 
		AVG(defect_rates) AS defects
	    FROM supplychain_sample
	    GROUP BY 1
	)
	SELECT o.supplier_name, 
	       o.delivery_rate, 
	       d.defects
	FROM OnTime o
	JOIN Defects d 
		ON o.supplier_name = d.supplier_name
	ORDER BY delivery_rate DESC, defects ASC;

-- 17. What is the most cost-effective transportation mode and route for each shipment?


	-- 17. Most cost-effective transportation mode and route per shipment
WITH CostEffective AS (
    SELECT 
        transportation_modes, 
        routes, 
        SUM(shipping_costs) AS total_costs,
        ROW_NUMBER() OVER (
            PARTITION BY routes 
            ORDER BY SUM(shipping_costs) ASC
        ) AS ranking
    FROM supplychain_sample
    GROUP BY transportation_modes, routes
)
SELECT 
    transportation_modes, 
    routes, 
    ROUND(total_costs, 0) AS costs
FROM CostEffective
WHERE ranking = 1;


-- 18. What are the lead times for all suppliers?

	SELECT supplier_name, SUM(lead_times)
	FROM supplychain_sample
	GROUP BY 1
	ORDER BY 2 DESC;

-- 19. What transport mode has more populated customers?

	SELECT transportation_modes, COUNT(customer_demographics)
	FROM supplychain_sample
	GROUP BY 1
	ORDER BY 2 DESC;

-- 20. What product type has the most defect rate?
	SELECT product_type, CEIL(SUM(defect_rates))
	FROM supplychain_sample 
	GROUP BY 1
	ORDER BY 2 DESC;

select *
from supplychain_sample;

-- 21. What is the most cost-effective transportation mode and carrier for each shipment?
SELECT 
    Transportation_modes, 
    shipping_carriers, 
    ROUND(SUM(shipping_costs), 0) AS total_costs,
    ROW_NUMBER() OVER (
        PARTITION BY Transportation_modes 
        ORDER BY SUM(shipping_costs) DESC
    ) AS ranking
FROM supplychain_sample
GROUP BY Transportation_modes, shipping_carriers;

    -- 22. Which product type generates the highest average revenue per unit sold?
    
    SELECT 
    product_type,
    ROUND(SUM(revenue_generated) / SUM(number_of_products_sold), 2) AS avg_revenue_per_unit
FROM supplychain_sample
GROUP BY product_type
ORDER BY avg_revenue_per_unit DESC;

    -- What is the average revenue per SKU across all product types?
    
    SELECT 
    sku,
    ROUND(AVG(revenue_generated), 2) AS avg_revenue
FROM supplychain_sample
GROUP BY sku
ORDER BY avg_revenue DESC
LIMIT 10;


-- Which suppliers serve the widest range of locations?

SELECT 
    supplier_name,
    COUNT(DISTINCT location) AS total_locations
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY total_locations DESC;

-- Which supplier has the best cost-to-revenue ratio?

SELECT 
    supplier_name,
    ROUND(SUM(revenue_generated) / SUM(manufacturing_costs + shipping_costs), 2) AS revenue_to_cost_ratio
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY revenue_to_cost_ratio DESC;

-- Which product type has the highest defect-adjusted cost?

SELECT 
    product_type,
    ROUND(SUM((manufacturing_costs + shipping_costs) * defect_rates), 2) AS defect_adjusted_cost
FROM supplychain_sample
GROUP BY product_type
ORDER BY defect_adjusted_cost DESC;


-- 27. Which routes are most expensive on average?

SELECT 
    routes,
    ROUND(AVG(shipping_costs), 2) AS avg_route_cost
FROM supplychain_sample
GROUP BY routes
ORDER BY avg_route_cost DESC;

-- Which supplier generates the highest revenue per shipment?

SELECT 
    supplier_name,
    ROUND(SUM(revenue_generated) / COUNT(*), 2) AS revenue_per_shipment
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY revenue_per_shipment DESC;

-- Which shipping carrier has the lowest average shipping cost per order?

SELECT 
    shipping_carriers,
    ROUND(AVG(shipping_costs), 2) AS avg_shipping_cost
FROM supplychain_sample
GROUP BY shipping_carriers
ORDER BY avg_shipping_cost ASC;


-- Which suppliers provide the lowest manufacturing cost per unit?

SELECT 
    supplier_name,
    ROUND(AVG(manufacturing_costs), 2) AS avg_cost_per_unit
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY avg_cost_per_unit ASC
LIMIT 5;


-- Which customer demographic contributes the most revenue per order?

SELECT 
    customer_demographics,
    ROUND(SUM(revenue_generated) / SUM(order_quantities), 2) AS revenue_per_order
FROM supplychain_sample
GROUP BY customer_demographics
ORDER BY revenue_per_order DESC;
