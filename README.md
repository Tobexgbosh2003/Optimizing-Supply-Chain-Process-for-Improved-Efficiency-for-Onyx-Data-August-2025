CREATE TABLE supplychain_sample (
    Product_type              VARCHAR(50),
    SKU                       VARCHAR(20),
    Price                     DECIMAL(10,2),
    Availability              INT,
    Number_of_products_sold   INT,
    Revenue_generated         DECIMAL(12,2),
    Customer_demographics     VARCHAR(50),
    Stock_levels              INT,
    Lead_time                 INT,
    Order_quantities          INT,
    Shipping_times            INT,
    Shipping_carriers         VARCHAR(50),
    Shipping_costs            DECIMAL(10,2),
    Supplier_name             VARCHAR(50),
    Location                  VARCHAR(50),
    Latitude                  DECIMAL(9,6),
    Longitude                 DECIMAL(9,6),
    Production_volumes        INT,
    Manufacturing_lead_time   INT,
    Manufacturing_costs       DECIMAL(12,2),
    Inspection_results        VARCHAR(50),
    Defect_rates              DECIMAL(5,2),
    Transportation_modes      VARCHAR(20),
    Routes                    VARCHAR(100),
    Costs                     DECIMAL(12,2)
);

    
select * 
from supplychain_sample;
 -- EDA  

-- Problem statement  

-- 1. What are the most frequently ordered products?  
SELECT 
    product_type, 
    COUNT(Number_of_products_sold) AS Total_prd
FROM supplychain_sample
GROUP BY product_type
ORDER BY 2 DESC;

-- 2. What is the total value of inventory on hand?  
SELECT 
    product_type, 
    CEIL(SUM(stock_levels * price)) AS inventory_value
FROM supplychain_sample
GROUP BY product_type
ORDER BY inventory_value DESC;

-- 3. Which supplier has the highest average delivery time?  
SELECT DISTINCT 
    supplier_name, 
    ROUND(AVG(lead_times), 3) AS AV_DT
FROM supplychain_sample
GROUP BY 1
ORDER BY 2 DESC 
LIMIT 2;

-- 4. What is the average cost per unit for each supplier?  
SELECT 
    supplier_name, 
    CEIL(AVG(manufacturing_costs)) AS avg_cost
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY avg_cost DESC;

-- 5. Which supplier has the highest defect rate?  
SELECT 
    supplier_name, 
    CEIL(SUM(defect_rates)) AS Total_defects
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY 2 DESC;

-- 6. Using a CTE to find supplier with highest defect rate  
WITH Cte_example AS (
    SELECT 
        supplier_name, 
        SUM(defect_rates) AS Defects, 
        RANK() OVER (ORDER BY SUM(defect_rates) DESC) AS Supplier_rank
    FROM supplychain_sample
    GROUP BY 1 
)
SELECT 
    supplier_name, 
    CEIL(Defects) AS total_defect 
FROM Cte_example
WHERE Supplier_rank = 1;

-- Alternative form  
SELECT * 
FROM (
    SELECT 
        supplier_name, 
        CEIL(SUM(defect_rates)) AS total_defects,
        RANK() OVER (ORDER BY SUM(defect_rates) DESC) AS Supplier_rank
    FROM supplychain_sample
    GROUP BY 1
) AS Pop
WHERE Supplier_rank = 1;

-- 7. Which carrier has the highest on-time delivery rate?  
SELECT * 
FROM (
    SELECT 
        shipping_carriers, 
        SUM(shipping_times) AS highest,
        ROW_NUMBER() OVER (ORDER BY SUM(shipping_times) DESC) AS best_carrier
    FROM supplychain_sample
    GROUP BY 1
) AS Tot
WHERE best_carrier = 1;

-- 8. What is the total number of shipments by carrier?  
SELECT 
    shipping_carriers, 
    COUNT(*) 
FROM supplychain_sample
GROUP BY 1;

-- 9. What is the total revenue generated?  
SELECT 
    CEIL(SUM(revenue_generated)) AS total
FROM supplychain_sample;

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
-- 1. Total revenue generated
SELECT 
    CEIL(SUM(revenue_generated)) AS total_revenue
FROM supplychain_sample;

-- 2. Top 5 products by revenue
SELECT 
    product_type, 
    CEIL(SUM(revenue_generated)) AS revenue
FROM supplychain_sample
GROUP BY product_type
ORDER BY revenue DESC
LIMIT 5;

-- 3. Average lead time per supplier
SELECT 
    supplier_name, 
    ROUND(AVG(lead_times), 2) AS avg_lead_time
FROM supplychain_sample
GROUP BY supplier_name;

-- 4. Monthly sales trend
SELECT 
    DATE_FORMAT(order_date, '%Y-%m') AS month, 
    SUM(revenue_generated) AS revenue
FROM supplychain_sample
GROUP BY month
ORDER BY month;

-- 5. Defect rate by supplier
SELECT 
    supplier_name, 
    ROUND(AVG(defect_rates), 2) AS avg_defect_rate
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY avg_defect_rate DESC;

-- 6. Revenue by region
SELECT 
    regions, 
    CEIL(SUM(revenue_generated)) AS revenue
FROM supplychain_sample
GROUP BY regions
ORDER BY revenue DESC;

-- 7. Highest revenue customer segment
SELECT 
    customer_demographics, 
    CEIL(SUM(revenue_generated)) AS revenue
FROM supplychain_sample
GROUP BY customer_demographics
ORDER BY revenue DESC
LIMIT 1;

-- 8. Stock levels by product
SELECT 
    product_type, 
    SUM(stock_levels) AS stock_available
FROM supplychain_sample
GROUP BY product_type
ORDER BY stock_available ASC;

-- 9. Transportation cost per mode
SELECT 
    transportation_modes, 
    ROUND(AVG(shipping_costs), 2) AS avg_cost
FROM supplychain_sample
GROUP BY transportation_modes
ORDER BY avg_cost ASC;

-- 10. Orders by region and route
SELECT 
    regions, 
    routes, 
    COUNT(order_id) AS total_orders
FROM supplychain_sample
GROUP BY regions, routes
ORDER BY total_orders DESC;

-- 11. Top supplier by defect rate
WITH CTE AS (
    SELECT 
        supplier_name, 
        SUM(defect_rates) AS defects,
        RANK() OVER (ORDER BY SUM(defect_rates) DESC) AS supplier_rank
    FROM supplychain_sample
    GROUP BY supplier_name
)
SELECT 
    supplier_name, 
    CEIL(defects) AS total_defect
FROM CTE
WHERE supplier_rank = 1;

-- 12. Customer segment revenue by product type
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
) AS derived_table;

-- 13. Orders with delayed shipping
SELECT 
    order_id, 
    product_type, 
    shipping_times, 
    lead_times
FROM supplychain_sample
WHERE shipping_times > lead_times;

-- 14. Most used transportation route
SELECT 
    routes, 
    COUNT(order_id) AS total_orders
FROM supplychain_sample
GROUP BY routes
ORDER BY total_orders DESC
LIMIT 1;

-- 15. Average revenue per customer demographic
SELECT 
    customer_demographics, 
    ROUND(AVG(revenue_generated), 2) AS avg_revenue
FROM supplychain_sample
GROUP BY customer_demographics
ORDER BY avg_revenue DESC;

-- 16. Product with highest stock-out risk
SELECT 
    product_type, 
    SUM(stock_levels) AS stock_remaining, 
    SUM(order_quantities) AS total_orders
FROM supplychain_sample
GROUP BY product_type
ORDER BY (total_orders - stock_remaining) DESC
LIMIT 1;

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

-- 18. Yearly revenue growth
SELECT 
    YEAR(order_date) AS year, 
    SUM(revenue_generated) AS revenue,
    LAG(SUM(revenue_generated)) OVER (ORDER BY YEAR(order_date)) AS prev_year_revenue,
    (SUM(revenue_generated) - LAG(SUM(revenue_generated)) OVER (ORDER BY YEAR(order_date))) / 
    LAG(SUM(revenue_generated)) OVER (ORDER BY YEAR(order_date)) * 100 AS growth_percent
FROM supplychain_sample
GROUP BY YEAR(order_date)
ORDER BY year;

-- 19. Top 3 suppliers by revenue contribution
SELECT 
    supplier_name, 
    CEIL(SUM(revenue_generated)) AS revenue
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY revenue DESC
LIMIT 3;

-- 20. Shipping performance (on-time vs delayed)
SELECT 
    CASE 
        WHEN shipping_times <= lead_times THEN 'On-Time'
        ELSE 'Delayed'
    END AS shipping_status,
    COUNT(order_id) AS total_orders
FROM supplychain_sample
GROUP BY shipping_status;

GROUP BY customer_demographics
ORDER BY revenue_per_order DESC;
