 Customer Table (Top 5 Rows)

| Customer_ID | First   | Last    | Age | Country |
| ----------- | ------- | ------- | --- | ------- |
| 1           | Joseph  | Rice    | 43  | USA     |
| 2           | Gary    | Moore   | 71  | USA     |
| 3           | John    | Walker  | 44  | UK      |
| 4           | Eric    | Carter  | 38  | UK      |
| 5           | William | Jackson | 58  | UAE     |

 Orders Table (Top 5 Rows)

| Order_ID | Item     | Amount | Customer_ID |
| -------- | -------- | ------ | ----------- |
| 1        | Keyboard | 400    | 139         |
| 2        | Mouse    | 300    | 250         |
| 3        | Monitor  | 12000  | 239         |
| 4        | Keyboard | 400    | 153         |
| 5        | Mousepad | 250    | 153         |

 Shipping Table (Top 5 Rows)

| Shipping_ID | Status    | Customer_ID |
| ----------- | --------- | ----------- |
| 1           | Pending   | 173         |
| 2           | Pending   | 155         |
| 3           | Delivered | 242         |
| 4           | Pending   | 223         |
| 5           | Delivered | 72          |



# 1. Data Validation (Accuracy, Completeness, Reliability)

Key Findings from Source Data are : 

A. Shipping Data Issues (from Shipping.json)

- Duplicate Customer_IDs → one customer has multiple shipments 
- No Order_ID linkage → cannot directly map shipment to order (major data gap)
- Status values are consistent: Pending / Delivered
- No missing values observed in sample
- Total rows count : 250


B. Customer Table Checks (from Customer.csv)
```SQL
--Total Rows
SELECT count(*) from Customer
--250

-- Check nulls
SELECT COUNT(*) 
FROM Customer
WHERE customer_id IS NULL OR country IS NULL OR age IS NULL;


-- Check duplicates
SELECT customer_id, COUNT(*)
FROM Customer
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

C. Order Table Checks (from Order.csv)
```SQL
SELECT count(*) from Order
--250

-- Check no matching customer
SELECT o.customer_id
FROM Orders o
LEFT JOIN Customer c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;

-- Check negative or invalid values
SELECT *
FROM Orders
WHERE amount <= 0;
```

# 2. Proposed Data Model 
 Recommended : Star Schema
 
Fact Table: 

fact_sales(order_id,
customer_id,
product_id,
shipping_id,
country_id,
quantity,
amount,
delivery_status,
order_date)

Dimension Tables:

dim_customer(
customer_id,
customer_name,
age,
age_group (<30, 30+),
country_id)
 
dim_product (To be Created) 
(product_id,
product_name,
category,
price)

dim_country(
country_id,
country_name)

dim_date(
date_id,
day,
month,
year)

 
 dim_shipping(
 shipping_id ,
 customer_id,
 order_id (to be added),
 status)

# Relationships
fact_sales → dim_customer (customer_id)
fact_sales → dim_product (product_id)
fact_sales → dim_country (country_id)
fact_sales → dim_shipping (shipping_id)


# 3. Data Engineer Story (Technical Specification)

- Fix Shipping Table & Build Fact Table

# Source Tables:
- raw_sales
- raw_customers
- raw_products
- raw_shipping

Transformations:

1. Data Cleaning
- Remove duplicates based on order_id
- Filter invalid records: price > 0
  
2. Standardization
- Convert delivery_status to uppercase : UPPER(delivery_status)
   
3. Age Group Logic :
```SQL
CASE 
    WHEN age < 30 THEN '<30'
    ELSE '30+'
END AS age_group
```

4. Final Table Creation :
```SQL
CREATE TABLE fact_sales AS
SELECT 
    s.order_id,
    c.customer_id,
    p.product_id,
    h.shipping_id,
    c.country_id,
    s.quantity,
    s.quantity * p.price AS amount,
    UPPER(s.delivery_status) AS delivery_status,
    s.order_date
FROM raw_sales s
JOIN raw_customers c ON s.customer_id = c.customer_id
JOIN raw_products p ON s.product_id = p.product_id;
JOIN shipping h ON h.order_id = s.order_id;
```

# QA Test Cases
- No duplicate order_id
- No null customer_id/product_id
- amount = quantity * price
- delivery_status only contains valid values (PENDING, DELIVERED)
- Age group correctly assigned


# 1. Total Amount + Country for Pending Delivery
```SQL
SELECT country_id, SUM(amount) AS total_amount
FROM fact_sales
WHERE upper(delivery_status) = 'PENDING'
GROUP BY country_id;
```


# 2. Customer Transactions Summary
```SQL
SELECT 
    customer_id,
    COUNT(order_id) AS total_transactions,
    SUM(quantity) AS total_quantity,
    SUM(amount) AS total_spent
FROM fact_sales
GROUP BY customer_id;
```

# 3. Max Product per Country
```SQL
SELECT country_id, product_id, SUM(quantity) AS total_qty
FROM fact_sales
GROUP BY country_id, product_id
QUALIFY RANK() OVER (PARTITION BY country_id ORDER BY SUM(quantity) DESC) = 1;
```

# 4. Most Purchased Product by Age Group
```SQL
SELECT age_group, product_id, SUM(quantity) AS total_qty
FROM fact_sales fs
JOIN dim_customer dc ON fs.customer_id = dc.customer_id
GROUP BY age_group, product_id
QUALIFY RANK() OVER (PARTITION BY age_group ORDER BY SUM(quantity) DESC) = 1;
```

# 5. Country with Minimum Transactions
```SQL
SELECT country_id,
       COUNT(order_id) AS transactions,
       SUM(amount) AS total_sales
FROM fact_sales
GROUP BY country_id
ORDER BY transactions ASC, total_sales ASC
LIMIT 1;
```
