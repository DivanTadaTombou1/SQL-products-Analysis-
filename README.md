# SQL-products-Analysis

  My goal is to :

- Calculates total sales, average order value, and total orders per customer.
- Identifies the top 10 products by sales and average rating.
- Analyses monthly sales trends by region.
- Determines customer segments based on purchase frequency and order value.

```sql

WITH CustomerSales AS (
    SELECT
        c.customer_id,
        c.name,
        c.region,
        COUNT(o.order_id) AS total_orders,
        SUM(oi.quantity * oi.item_price) AS total_sales,
        AVG(oi.quantity * oi.item_price) AS average_order_value
    FROM
        customers c
    JOIN
        orders o ON c.customer_id = o.customer_id
    JOIN
        order_items oi ON o.order_id = oi.order_id
    GROUP BY
        c.customer_id, c.name, c.region
),

ProductPerformance AS (
    SELECT
        p.product_id,
        p.product_name,
        p.category,
        SUM(oi.quantity * oi.item_price) AS total_sales,
        AVG(r.rating) AS average_rating,
        COUNT(DISTINCT o.order_id) AS total_orders
    FROM
        products p
    JOIN
        order_items oi ON p.product_id = oi.product_id
    JOIN
        orders o ON oi.order_id = o.order_id
    LEFT JOIN
        reviews r ON p.product_id = r.product_id
    GROUP BY
        p.product_id, p.product_name, p.category
),

MonthlySales AS (
    SELECT
        DATE_TRUNC('month', o.order_date) AS month,
        c.region,
        SUM(oi.quantity * oi.item_price) AS total_sales
    FROM
        orders o
    JOIN
        customers c ON o.customer_id = c.customer_id
    JOIN
        order_items oi ON o.order_id = oi.order_id
    GROUP BY
        DATE_TRUNC('month', o.order_date), c.region
),

CustomerSegments AS (
    SELECT
        c.customer_id,
        c.name,
        c.region,
        CASE
            WHEN COUNT(o.order_id) > 10 THEN 'High Frequency'
            WHEN COUNT(o.order_id) BETWEEN 5 AND 10 THEN 'Medium Frequency'
            ELSE 'Low Frequency'
        END AS frequency_segment,
        CASE
            WHEN SUM(oi.quantity * oi.item_price) > 1000 THEN 'High Value'
            WHEN SUM(oi.quantity * oi.item_price) BETWEEN 500 AND 1000 THEN 'Medium Value'
            ELSE 'Low Value'
        END AS value_segment
    FROM
        customers c
    JOIN
        orders o ON c.customer_id = o.customer_id
    JOIN
        order_items oi ON o.order_id = oi.order_id
    GROUP BY
        c.customer_id, c.name, c.region
)

SELECT
    cs.customer_id,
    cs.name,
    cs.region,
    cs.total_orders,
    cs.total_sales,
    cs.average_order_value,
    ps.product_name,
    ps.category,
    ps.total_sales AS product_total_sales,
    ps.average_rating,
    ms.month,
    ms.total_sales AS monthly_total_sales,
    seg.frequency_segment,
    seg.value_segment
FROM
    CustomerSales cs
LEFT JOIN
    ProductPerformance ps ON cs.customer_id = ps.product_id
LEFT JOIN
    MonthlySales ms ON cs.region = ms.region
LEFT JOIN
    CustomerSegments seg ON cs.customer_id = seg.customer_id
ORDER BY
    cs.total_sales DESC, ps.total_sales DESC, ms.month ASC;


```
