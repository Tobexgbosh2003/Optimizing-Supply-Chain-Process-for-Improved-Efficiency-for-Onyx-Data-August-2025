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


