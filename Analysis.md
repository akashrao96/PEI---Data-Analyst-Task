# Customer Table (Top 5 Rows)

| Customer_ID | First   | Last    | Age | Country |
| ----------- | ------- | ------- | --- | ------- |
| 1           | Joseph  | Rice    | 43  | USA     |
| 2           | Gary    | Moore   | 71  | USA     |
| 3           | John    | Walker  | 44  | UK      |
| 4           | Eric    | Carter  | 38  | UK      |
| 5           | William | Jackson | 58  | UAE     |

# Orders Table (Top 5 Rows)

| Order_ID | Item     | Amount | Customer_ID |
| -------- | -------- | ------ | ----------- |
| 1        | Keyboard | 400    | 139         |
| 2        | Mouse    | 300    | 250         |
| 3        | Monitor  | 12000  | 239         |
| 4        | Keyboard | 400    | 153         |
| 5        | Mousepad | 250    | 153         |

# Shipping Table (Top 5 Rows)

| Shipping_ID | Status    | Customer_ID |
| ----------- | --------- | ----------- |
| 1           | Pending   | 173         |
| 2           | Pending   | 155         |
| 3           | Delivered | 242         |
| 4           | Pending   | 223         |
| 5           | Delivered | 72          |



1. Data Validation (Accuracy, Completeness, Reliability)

Key Findings from Source Data are : 

A. Shipping Data Issues (from Shipping.json)

- Duplicate Customer_IDs → one customer has multiple shipments 
- No Order_ID linkage → cannot directly map shipment to order (major data gap)
- Status values are consistent: Pending / Delivered
- No missing values observed in sample
- Total rows count : 250


B. Customer Table Checks (from Customer.csv)

-- Check nulls
SELECT COUNT(*) 
FROM Customer
WHERE customer_id IS NULL OR country IS NULL OR age IS NULL;

-- Check duplicates
SELECT customer_id, COUNT(*)
FROM Customer
GROUP BY customer_id
HAVING COUNT(*) > 1;




