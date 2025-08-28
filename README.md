-- ==========================================================
-- Optimizing Supply Chain Process for Improved Efficiency
-- Onyx Data | August 2025
-- Title: Supply Chain Analysis
-- ==========================================================

-- ================================
-- SCHEMA DEFINITION
-- ================================

CREATE TABLE supplychain_sample (
    product_type             VARCHAR(20),
    sku                      VARCHAR(10),
    price                    FLOAT,
    availability             INT,
    number_of_products_sold  INT,
    revenue_generated        FLOAT,
    customer_demographics    VARCHAR(20),
    stock_levels             INT,
    lead_times               INT,
    order_quantities         INT,
    shipping_times           INT,
    shipping_carriers        VARCHAR(20),
    shipping_costs           FLOAT,
    supplier_name            VARCHAR(20),
    location                 VARCHAR(20),
    latitude                 FLOAT,   -- fixed typo (was 'Latitide')
    longitude                FLOAT,
    lead_time                INT,
    production_volumes       INT,
    manufacturing_lead_time  INT,
    manufacturing_costs      FLOAT,
    inspection_results       VARCHAR(20),
    defect_rates             FLOAT,
    transportation_modes     VARCHAR(10),
    routes                   VARCHAR(20),
    costs                    FLOAT
);

-- Preview the table
SELECT * 
FROM supplychain_sample;


-- ==========================================================
-- EXPLORATORY DATA ANALYSIS (EDA) QUERIES
-- ==========================================================

-- 1. Most frequently ordered products
SELECT 
    product_type, 
    COUNT(number_of_products_sold) AS total_prd
FROM supplychain_sample
GROUP BY product_type
ORDER BY total_prd DESC;


-- 2. Total value of inventory on hand
SELECT 
    product_type, 
    CEIL(SUM(stock_levels * price)) AS inventory_value
FROM supplychain_sample
GROUP BY product_type
ORDER BY inventory_value DESC;


-- 3. Suppliers with the highest average delivery time
SELECT  
    supplier_name, 
    ROUND(AVG(lead_times), 3) AS avg_delivery_time
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY avg_delivery_time DESC
LIMIT 2;


-- 4. Average cost per unit for each supplier
SELECT 
    supplier_name, 
    CEIL(AVG(manufacturing_costs)) AS avg_cost
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY avg_cost DESC;


-- 5. Supplier with the highest defect rate
SELECT 
    supplier_name, 
    CEIL(SUM(defect_rates)) AS total_defects
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY total_defects DESC;


-- 6. Highest defect supplier (as CTE example)
WITH Cte_example AS (
    SELECT 
        supplier_name, 
        SUM(defect_rates) AS defects, 
        RANK() OVER (ORDER BY SUM(defect_rates) DESC) AS supplier_rank
    FROM supplychain_sample
    GROUP BY 1
)
SELECT 
    supplier_name, 
    CEIL(defects) AS total_defect
FROM Cte_example
WHERE supplier_rank = 1;


-- 7. Carrier with the highest on-time delivery rate
SELECT * 
FROM (
    SELECT 
        shipping_carriers, 
        SUM(shipping_times) AS highest,
        ROW_NUMBER () OVER (ORDER BY SUM(shipping_times) DESC) AS best_carrier
    FROM supplychain_sample
    GROUP BY 1
) AS Tot
WHERE best_carrier = 1;


-- 8. Total number of shipments by carrier
SELECT 
    shipping_carriers, 
    COUNT(*) AS total_shipments
FROM supplychain_sample
GROUP BY 1;


-- 9. Total revenue generated
SELECT 
    CEIL(SUM(revenue_generated)) AS total_revenue
FROM supplychain_sample;


-- 10. Highest revenue by product category and customer segment
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


-- 11. Average order value
SELECT 
    ROUND(AVG(order_quantities), 3) AS avg_order_value
FROM supplychain_sample;


-- 12. Top 3 suppliers by revenue
WITH Calculation AS (
    SELECT 
        supplier_name, 
        SUM(revenue_generated) AS total_revenue,
        RANK() OVER (ORDER BY SUM(revenue_generated) DESC) AS ranking
    FROM supplychain_sample
    GROUP BY 1
)
SELECT 
    supplier_name, 
    CEIL(total_revenue) AS total_revenue,
    ranking
FROM Calculation
LIMIT 3;


-- 13. Average order frequency per customer
SELECT 
    customer_demographics, 
    ROUND(AVG(order_quantities), 0) AS order_freq
FROM supplychain_sample
GROUP BY 1
ORDER BY 1;


-- 14. Most profitable location
SELECT 
    location, 
    CEIL(SUM(revenue_generated)) AS most_profitable
FROM supplychain_sample
GROUP BY 1
ORDER BY most_profitable DESC;


-- 15. Top 10 performing SKUs by revenue and products sold
SELECT 
    sku, 
    CEIL(SUM(revenue_generated)) AS total_revenue,
    SUM(number_of_products_sold) AS total_products
FROM supplychain_sample
GROUP BY 1
ORDER BY total_revenue DESC, total_products DESC
LIMIT 10;


-- 16. Suppliers with highest on-time delivery rates & lowest defect rates
WITH OnTime AS (
    SELECT 
        supplier_name, 
        ROUND(AVG(lead_times),3) AS delivery_rate
    FROM supplychain_sample
    GROUP BY 1
),
Defects AS (
    SELECT 
        supplier_name, 
        AVG(defect_rates) AS defects
    FROM supplychain_sample
    GROUP BY 1
)
SELECT 
    o.supplier_name, 
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


-- 18. Lead times for all suppliers
SELECT 
    supplier_name, 
    SUM(lead_times) AS total_lead_time
FROM supplychain_sample
GROUP BY 1
ORDER BY total_lead_time DESC;


-- 19. Transport mode with most customers
SELECT 
    transportation_modes, 
    COUNT(customer_demographics) AS total_customers
FROM supplychain_sample
GROUP BY 1
ORDER BY total_customers DESC;


-- 20. Product type with the most defects
SELECT 
    product_type, 
    CEIL(SUM(defect_rates)) AS total_defects
FROM supplychain_sample 
GROUP BY 1
ORDER BY total_defects DESC;


-- 21. Most cost-effective transportation mode and carrier per shipment
SELECT 
    transportation_modes, 
    shipping_carriers, 
    ROUND(SUM(shipping_costs), 0) AS total_costs,
    ROW_NUMBER() OVER (
        PARTITION BY transportation_modes 
        ORDER BY SUM(shipping_costs) DESC
    ) AS ranking
FROM supplychain_sample
GROUP BY transportation_modes, shipping_carriers;


-- 22. Product type with highest average revenue per unit sold
SELECT 
    product_type,
    ROUND(SUM(revenue_generated) / SUM(number_of_products_sold), 2) AS avg_revenue_per_unit
FROM supplychain_sample
GROUP BY product_type
ORDER BY avg_revenue_per_unit DESC;


-- 23. Average revenue per SKU (top 10)
SELECT 
    sku,
    ROUND(AVG(revenue_generated), 2) AS avg_revenue
FROM supplychain_sample
GROUP BY sku
ORDER BY avg_revenue DESC
LIMIT 10;


-- 24. Suppliers serving the widest range of locations
SELECT 
    supplier_name, 
    COUNT(DISTINCT location) AS total_locations
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY total_locations DESC;


-- 25. Supplier with best cost-to-revenue ratio
SELECT 
    supplier_name, 
    ROUND(SUM(revenue_generated) / SUM(manufacturing_costs + shipping_costs), 2) AS revenue_to_cost_ratio
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY revenue_to_cost_ratio DESC;


-- 26. Product type with highest defect-adjusted cost
SELECT 
    product_type, 
    ROUND(SUM((manufacturing_costs + shipping_costs) * defect_rates), 2) AS defect_adjusted_cost
FROM supplychain_sample
GROUP BY product_type
ORDER BY defect_adjusted_cost DESC;


-- 27. Most expensive routes (average)
SELECT 
    routes, 
    ROUND(AVG(shipping_costs), 2) AS avg_route_cost
FROM supplychain_sample
GROUP BY routes
ORDER BY avg_route_cost DESC;


-- 28. Supplier generating highest revenue per shipment
SELECT 
    supplier_name, 
    ROUND(SUM(revenue_generated) / COUNT(*), 2) AS revenue_per_shipment
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY revenue_per_shipment DESC;


-- 29. Carrier with lowest average shipping cost
SELECT 
    shipping_carriers, 
    ROUND(AVG(shipping_costs), 2) AS avg_shipping_cost
FROM supplychain_sample
GROUP BY shipping_carriers
ORDER BY avg_shipping_cost ASC;


-- 30. Suppliers with lowest manufacturing cost per unit (Top 5)
SELECT 
    supplier_name, 
    ROUND(AVG(manufacturing_costs), 2) AS avg_cost_per_unit
FROM supplychain_sample
GROUP BY supplier_name
ORDER BY avg_cost_per_unit ASC
LIMIT 5;


-- 31. Customer demographic with highest revenue per order
SELECT 
    customer_demographics, 
    ROUND(SUM(revenue_generated) / SUM(order_quantities), 2) AS revenue_per_order
FROM supplychain_sample
GROUP BY customer_demographics
ORDER BY revenue_per_order DESC;
