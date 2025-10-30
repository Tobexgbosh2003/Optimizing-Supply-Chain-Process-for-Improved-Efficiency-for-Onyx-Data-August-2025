CREATE TABLE supplychain_sample (
    Product_type varchar(20),    
    SKU varchar(10),
    Price float,
    Availability int,    
    Number_of_products_sold int,
    Revenue_generated float,    
    Customer_demographics varchar(20),
    Stock_levels int,    
    Lead_times int,
    Order_quantities int,    
    Shipping_times int,    
    Shipping_carriers varchar(20),
    Shipping_costs float,
    Supplier_name varchar(20),
    Location varchar(20),
    Latitide float,
    Longitude float,
    Lead_time int,
    Production_volumes int,
    Manufacturing_lead_time int,
    Manufacturing_costs float,
    Inspection_results varchar(20),    
    Defect_rates float,    
    Transportation_modes varchar(10),    
    Routes varchar(20),
    Costs float
);

SELECT * FROM supplychain_sample;

/* Q1. Most frequently ordered products */
SELECT product_type, COUNT(Number_of_products_sold) AS Total_prd
FROM supplychain_sample
GROUP BY product_type
ORDER BY 2 DESC;

/* Q2. Total value of inventory on hand */
SELECT product_type, CEIL(SUM(stock_levels * price)) AS inventory_value
FROM supplychain_sample
GROUP BY product_type
ORDER BY inventory_value DESC;

/* Q3. Supplier with the highest average delivery time */
SELECT supplier_name, ROUND(AVG(Lead_times), 3) AS AV_DT
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY AV_DT DESC
LIMIT 2;

/* Q4. Average cost per unit for each supplier */
SELECT supplier_name, CEIL(AVG(manufacturing_costs)) AS avg_cost
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY avg_cost DESC;

/* Q5. Supplier with the highest defect rate */
SELECT supplier_name, CEIL(SUM(defect_rates)) AS Total_defects
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY 2 DESC;

/* Q6. Supplier with the highest defect rate (using CTE) */
WITH Cte_example AS (
    SELECT supplier_name, SUM(defect_rates) AS Defects, 
           RANK() OVER (ORDER BY SUM(defect_rates) DESC) AS Supplier_rank
    FROM supplychain_sample
    GROUP BY 1
)
SELECT supplier_name, CEIL(Defects) AS total_defect 
FROM Cte_example
WHERE Supplier_rank = 1;

/* Q7. Carrier with the highest on-time delivery rate */
SELECT * 
FROM (
    SELECT shipping_carriers, SUM(shipping_times) AS highest,
           ROW_NUMBER() OVER (ORDER BY SUM(shipping_times) DESC) AS best_carrier
    FROM supplychain_sample
    GROUP BY 1
) AS Tot
WHERE best_carrier = 1;

/* Q8. Total number of shipments by carrier */
SELECT Shipping_carriers, COUNT(*) AS total_shipments
FROM supplychain_sample
GROUP BY 1;

/* Q9. Total revenue generated */
SELECT CEIL(SUM(Revenue_generated)) AS total
FROM supplychain_sample;

/* Q10. Product category and customer generating highest revenue */
SELECT *
FROM (
    SELECT product_type, customer_demographics, CEIL(SUM(revenue_generated)) AS revenue,
           ROW_NUMBER() OVER (PARTITION BY customer_demographics 
                              ORDER BY SUM(revenue_generated) DESC) AS ranking
    FROM supplychain_sample
    GROUP BY product_type, customer_demographics
) AS derived_table
WHERE ranking = 1;

/* Q11. Average order value */
SELECT ROUND(AVG(Order_quantities), 3) AS avg_order_value
FROM supplychain_sample;

/* Q12. Top suppliers by revenue */
WITH Calculation AS (
    SELECT supplier_name, SUM(revenue_generated) AS total_revenue,
           RANK() OVER (ORDER BY SUM(revenue_generated) DESC) AS ranking
    FROM supplychain_sample
    GROUP BY 1
)
SELECT supplier_name, CEIL(total_revenue), ranking
FROM Calculation
LIMIT 3;

/* Q13. Average order frequency per customer */
SELECT customer_demographics, ROUND(AVG(order_quantities), 0) AS order_freq
FROM supplychain_sample
GROUP BY 1
ORDER BY 1;

/* Q14. Most profitable location */
SELECT location, CEIL(SUM(revenue_generated)) AS most_profitable
FROM supplychain_sample
GROUP BY 1
ORDER BY 2 DESC;

/* Q15. Top 10 performing SKUs by revenue and products sold */
SELECT sku, CEIL(SUM(revenue_generated)) AS total_revenue,
       SUM(number_of_products_sold) AS total_products
FROM supplychain_sample
GROUP BY 1
ORDER BY 2 DESC, 3 DESC
LIMIT 10;

/* Q16. Suppliers with highest on-time delivery and lowest defect rates */
WITH OnTime AS (
    SELECT supplier_name, ROUND(AVG(lead_times), 3) AS delivery_rate
    FROM supplychain_sample
    GROUP BY 1
),
Defects AS (
    SELECT supplier_name, AVG(defect_rates) AS defects
    FROM supplychain_sample
    GROUP BY 1
)
SELECT o.supplier_name, o.delivery_rate, d.defects
FROM OnTime o
JOIN Defects d ON o.supplier_name = d.supplier_name
ORDER BY delivery_rate DESC, defects ASC;

/* Q17. Most cost-effective transportation mode and route */
WITH CostEffective AS (
    SELECT transportation_modes, routes, SUM(shipping_costs) AS total_costs,
           ROW_NUMBER() OVER (PARTITION BY routes ORDER BY SUM(shipping_costs) ASC) AS ranking
    FROM supplychain_sample
    GROUP BY transportation_modes, routes
)
SELECT transportation_modes, routes, ROUND(total_costs, 0) AS costs
FROM CostEffective
WHERE ranking = 1;

/* Q18. Lead times for all suppliers */
SELECT supplier_name, SUM(lead_times) AS total_lead_time
FROM supplychain_sample
GROUP BY 1
ORDER BY 2 DESC;

/* Q19. Transport mode with the most customers */
SELECT transportation_modes, COUNT(customer_demographics) AS total_customers
FROM supplychain_sample
GROUP BY 1
ORDER BY 2 DESC;

/* Q20. Product type with the highest defect rate */
SELECT product_type, CEIL(SUM(defect_rates)) AS total_defects
FROM supplychain_sample 
GROUP BY 1
ORDER BY 2 DESC;

/* Q21. Most cost-effective transportation mode and carrier */
SELECT Transportation_modes, shipping_carriers, ROUND(SUM(shipping_costs), 0) AS total_costs,
       ROW_NUMBER() OVER (PARTITION BY Transportation_modes ORDER BY SUM(shipping_costs) ASC) AS ranking
FROM supplychain_sample
GROUP BY Transportation_modes, shipping_carriers;

/* Q22. Product type generating highest average revenue per unit sold */
SELECT product_type,
       ROUND(SUM(revenue_generated) / SUM(number_of_products_sold), 2) AS avg_revenue_per_unit
FROM supplychain_sample
GROUP BY product_type
ORDER BY avg_revenue_per_unit DESC;

/* Q23. Average revenue per SKU across all product types */
SELECT sku, ROUND(AVG(revenue_generated), 2) AS avg_revenue
FROM supplychain_sample
GROUP BY sku
ORDER BY avg_revenue DESC
LIMIT 10;

/* Q24. Suppliers serving the widest range of locations */
SELECT supplier_name, COUNT(DISTINCT location) AS total_locations
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY total_locations DESC;

/* Q25. Supplier with best cost-to-revenue ratio */
SELECT supplier_name, ROUND(SUM(revenue_generated) / SUM(manufacturing_costs + shipping_costs), 2) AS revenue_to_cost_ratio
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY revenue_to_cost_ratio DESC;

/* Q26. Product type with highest defect-adjusted cost */
SELECT product_type, ROUND(SUM((manufacturing_costs + shipping_costs) * defect_rates), 2) AS defect_adjusted_cost
FROM supplychain_sample
GROUP BY product_type
ORDER BY defect_adjusted_cost DESC;

/* Q27. Most expensive routes on average */
SELECT routes, ROUND(AVG(shipping_costs), 2) AS avg_route_cost
FROM supplychain_sample
GROUP BY routes
ORDER BY avg_route_cost DESC;

/* Q28. Supplier generating highest revenue per shipment */
SELECT supplier_name, ROUND(SUM(revenue_generated) / COUNT(*), 2) AS revenue_per_shipment
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY revenue_per_shipment DESC;

/* Q29. Shipping carrier with lowest average shipping cost per order */
SELECT shipping_carriers, ROUND(AVG(shipping_costs), 2) AS avg_shipping_cost
FROM supplychain_sample
GROUP BY shipping_carriers
ORDER BY avg_shipping_cost ASC;

/* Q30. Suppliers with lowest manufacturing cost per unit */
SELECT supplier_name, ROUND(AVG(manufacturing_costs), 2) AS avg_cost_per_unit
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY avg_cost_per_unit ASC
LIMIT 5;

/* Q31. Customer demographic contributing most revenue per order */
SELECT customer_demographics, ROUND(SUM(revenue_generated) / SUM(order_quantities), 2) AS revenue_per_order
FROM supplychain_sample
GROUP BY customer_demographics
ORDER BY revenue_per_order DESC;
