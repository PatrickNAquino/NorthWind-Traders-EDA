# NorthWind-Traders-EDA
Exploratory data analysis of Sales &amp; order data for Northwind Traders, a fictitious gourmet food supplier.

## OVERVIEW
The Northwind database contains the sales data for a fictitious company called “Northwind Traders,” which imports and exports specialty foods from around the world.
The dataset includes sample data for:

- **Suppliers:** Suppliers and vendors of Northwind
- **Customers:** Customers who buy products from Northwind
- **Employees:** Employee details of Northwind traders
- **Products:** Product information
- **Shippers:** The details of the shippers who ship the products from the traders to the end-customers
- **Orders and Order_Details:** Sales Order transactions taking place between the customers & the company

Link to the dataset: [NorthWind Traders Dataset](https://app.mavenanalytics.io/datasets?search=northwind)

Below, I'm doing an exploratory data analysis using SQL in order to understand the data and how the company works, 'whats their main product', 'from where the customers are?', 'how is the revenue trending over time ?'

## EXPLORING PRODUCTS
- The company works with 77 products distributed in 8 categories;
- 'Confections' category has the highest number of products with 13 items
- Dairy products is the top selling category with 'Raclette Courdavault' being the most ordered product with 54 orders representing 6.5% of total orders;
- 'Beverages' category has 21% of market share and the top revenue product 'Côte de Blaye' with 11.% ($ 149.1K);

```sql
-- How many products we have ? 
SELECT COUNT(*) FROM products;

-- How many categories ? 
SELECT COUNT(*) FROM categories;

-- How many products in each category ? 
SELECT 
  categoryName,
  COUNT(DISTINCT productID) AS product_qty
FROM categories
LEFT JOIN products USING(categoryID)
GROUP BY 1 WITH ROLLUP
ORDER BY 2 
;

-- Top selling Product ?
SELECT
  productName,
  categoryName,
  COUNT(DISTINCT orderID) AS qty_orders,
  CONCAT(ROUND((COUNT(DISTINCT orderID)/(SELECT COUNT(DISTINCT orderID) FROM orders))*100,1),' %') AS pct_orders
FROM order_details
LEFT JOIN products USING(productID)
LEFT JOIN categories USING(categoryID)
GROUP BY 1,2
ORDER BY 3 DESC
;

-- Total revenue by product 
SELECT
  productName,
  categoryName,
  ROUND(SUM(order_details.unitPrice * order_details.quantity),2) AS revenue,
  CONCAT(ROUND((SUM(order_details.unitPrice * order_details.quantity)/(SELECT SUM(order_details.unitPrice * order_details.quantity) FROM order_details))*100,1), ' %') AS pct_revenue
FROM order_details
LEFT JOIN products USING(productID)
LEFT JOIN categories USING(categoryID)
GROUP BY 1,2
ORDER BY 3 DESC
;

-- Total revenue by category
SELECT
  categoryName,
  ROUND(SUM(order_details.unitPrice * order_details.quantity),2) AS revenue,
  CONCAT(ROUND((SUM(order_details.unitPrice * order_details.quantity)/(SELECT SUM(order_details.unitPrice * order_details.quantity) FROM order_details))*100,1), ' %') AS pct_revenue
FROM order_details
LEFT JOIN products USING(productID)
LEFT JOIN categories USING(categoryID)
GROUP BY 1
ORDER BY 2 DESC
;
```

## EXPLORING EMPLOYEES
- The top 3 employees in sales are Margaret Peacock (156 un), Janet Leverling (127 un) and Nancy Davolio (123 un)
- Margaret Peacock is the top employee with 18.8% of sales with 156 orders related; her best-sellers products are Gnocchi di nonna Alice and Pâté chinois;  
- She is also the employee with the highest total revenue, representing 18.5% of the market share;

```sql
-- order per employee
SELECT
  employeeID,
  employeeName,
  COUNT(DISTINCT orderID) AS orders,
  CONCAT(ROUND((COUNT(DISTINCT orderID)/(SELECT COUNT(DISTINCT orderID) FROM orders))*100,1),' %') AS pct_orders
FROM orders
LEFT JOIN employees USING(employeeID)
GROUP BY 1,2
ORDER by 3 DESC
;

-- top selling product per employee
SELECT
  employeeID,
  employeeName,	
  productName,
  COUNT(DISTINCT orderID) AS product_qtd,
  CONCAT(ROUND((COUNT(DISTINCT orderID)/(SELECT COUNT(DISTINCT orderID) FROM orders))*100,1),' %') AS pct_orders
FROM order_details
LEFT JOIN products USING(productID)
LEFT JOIN orders USING(orderID)
LEFT JOIN employees USING(employeeID)
GROUP BY 1,2,3
ORDER BY 4 DESC
;

-- revenue per employee
SELECT
  employeeID,
  employeeName,
  ROUND(SUM(order_details.unitPrice * order_details.quantity),2) AS revenue,
  CONCAT(ROUND((SUM(order_details.unitPrice * order_details.quantity)/(SELECT SUM(order_details.unitPrice * order_details.quantity) FROM order_details))*100,1), ' %') AS pct_revenue
FROM orders
LEFT JOIN employees USING(employeeID)
LEFT JOIN order_details USING(orderID)
GROUP BY 1,2
ORDER by 3 DESC
;
```

## EXPLORING COUNTRIES
- USA, Germany and Austria are the most important countries with 47.9% of revenue and 39.4% of orders;
- With $ 263.5K of revenue, USA is selling mostly 'Camembert Pierrot' and 'Tarte au sucre';

```sql
-- revenue by country
SELECT 
  customers.country,
  ROUND(SUM(order_details.unitPrice * order_details.quantity),2) AS revenue,
  CONCAT(ROUND((SUM(order_details.unitPrice * order_details.quantity)/(SELECT SUM(order_details.unitPrice * order_details.quantity) FROM order_details))*100,1), ' %') AS pct_revenue
FROM order_details
LEFT JOIN products USING(productID)
LEFT JOIN orders USING(orderID)
LEFT JOIN customers USING(customerID)
GROUP BY 1
ORDER BY 2 DESC
;

-- orders by country
SELECT 
  customers.country,
  COUNT(DISTINCT orderID) AS orders,
  CONCAT(ROUND((COUNT(DISTINCT orderID)/(SELECT COUNT(DISTINCT orderID) FROM orders))*100,1),' %') AS pct_orders
FROM order_details
LEFT JOIN orders USING(orderID)
LEFT JOIN customers USING(customerID)
GROUP BY 1 WITH ROLLUP
ORDER BY 2 DESC
;

-- top selling product per country
SELECT
  country,
  productName,
  COUNT(DISTINCT orderID) AS product_qtd,
  CONCAT(ROUND((COUNT(DISTINCT orderID)/(SELECT COUNT(DISTINCT orderID) FROM orders))*100,1),' %') AS pct_orders
FROM order_details
LEFT JOIN products USING(productID)
LEFT JOIN orders USING(orderID)
LEFT JOIN employees USING(employeeID)
GROUP BY 1,2
ORDER BY 3 DESC
;
```

## EXPLORING CUSTOMERS
- From the total 91 customers USA, Germany and France have combined 38.47%;
- QUICK-Stop from Germany is the top customer with $ 117K in revenue;

```sql
-- total customers
SELECT
DISTINCT COUNT(*)
FROM customers;

-- customers by country
SELECT
  country,
  COUNT(*) AS customers,
  CONCAT(ROUND((COUNT(*)/(SELECT COUNT(*) FROM customers))*100,2), ' %') AS pct_customers
FROM customers
GROUP BY 1
ORDER BY 2 DESC
;

 -- revenue by customer
SELECT 
  customers.companyName,
  customers.country,
  ROUND(SUM(order_details.unitPrice * order_details.quantity),2) AS revenue
FROM order_details
LEFT JOIN products USING(productID)
LEFT JOIN orders USING(orderID)
LEFT JOIN customers USING(customerID)
GROUP BY 1,2
ORDER BY 3 DESC
;
```

## EXPLORING REVENUE
- The revenue increases over time from dec/2014 to apr/2015
- 'Guarana Fantastica' and 'Camembert Pierrot' are the top sellers from this period both with 8 orders representing 1% of total;
- 'Raclette Courdavault' from category 'Dairy Products' had the highest revenue in apr/2015 with $ 17.7K (13.2%);
- in Apr/2015 USA lead the market with 25.4% ($ 34.2K) in revenue with the best-seller product being 'Thüringer Rostbratwurst' from category 'Meat & Poultry' and $ 7.7K (5.7%) in revenue;

```sql
-- how is the revenue over time ?
SELECT
  YEAR(orderDate) AS yr,
  MONTH(orderDate) AS mo,
  DATE_FORMAT(orderDate,'%b/%y') AS mo_yr,
  ROUND(SUM(order_details.unitPrice * order_details.quantity),2) AS revenue
 FROM orders
 LEFT JOIN order_details USING(orderID)
 GROUP BY 3,1,2
 ORDER BY 1,2 ASC
 ;

-- Top selling Product apr/2015 ?
SELECT
  DATE_FORMAT(orderDate,'%b/%y') AS mo_yr,
  productName,
  categoryName,
  COUNT(DISTINCT orderID) AS qty_orders,
  CONCAT(ROUND((COUNT(DISTINCT orderID)/(SELECT COUNT(DISTINCT orderID) FROM orders))*100,1),' %') AS pct_orders
FROM order_details
LEFT JOIN products USING(productID)
LEFT JOIN categories USING(categoryID)
LEFT JOIN orders USING(orderID)
WHERE DATE_FORMAT(orderDate,'%b/%y') = 'Apr/15'
GROUP BY 1,2,3
ORDER BY 4 DESC
;

-- Total revenue by product in apr/2015
SELECT
  DATE_FORMAT(orderDate,'%b/%y') AS mo_yr,
  productName,
  categoryName,
  ROUND(SUM(order_details.unitPrice * order_details.quantity),2) AS revenue,
  CONCAT(ROUND((SUM(order_details.unitPrice * order_details.quantity)/(SELECT SUM(order_details.unitPrice * order_details.quantity) FROM order_details LEFT JOIN orders USING(orderID) WHERE DATE_FORMAT(orderDate,'%b/%y') = 'Apr/15' ))*100,1), ' %') AS pct_revenue
FROM order_details
LEFT JOIN products USING(productID)
LEFT JOIN categories USING(categoryID)
LEFT JOIN orders USING(orderID)
WHERE DATE_FORMAT(orderDate,'%b/%y') = 'Apr/15'
GROUP BY 1,2,3
ORDER BY 4 DESC
;

-- revenue by country in apr/2015
SELECT
  DATE_FORMAT(orderDate,'%b/%y') AS mo_yr,
  customers.country,
  ROUND(SUM(order_details.unitPrice * order_details.quantity),2) AS revenue,
  CONCAT(ROUND((SUM(order_details.unitPrice * order_details.quantity)/(SELECT SUM(order_details.unitPrice * order_details.quantity) FROM order_details LEFT JOIN orders USING(orderID) WHERE DATE_FORMAT(orderDate,'%b/%y') = 'Apr/15' ))*100,1), ' %') AS pct_revenue
FROM order_details
LEFT JOIN products USING(productID)
LEFT JOIN orders USING(orderID)
LEFT JOIN customers USING(customerID)
WHERE DATE_FORMAT(orderDate,'%b/%y') = 'Apr/15'
GROUP BY 1,2 WITH ROLLUP
ORDER BY 3 DESC
;

-- Total revenue by product in apr/2015 in USA
SELECT
  DATE_FORMAT(orderDate,'%b/%y') AS mo_yr,
  productName,
  categoryName,
  ROUND(SUM(order_details.unitPrice * order_details.quantity),2) AS revenue,
  CONCAT(ROUND((SUM(order_details.unitPrice * order_details.quantity)/(SELECT SUM(order_details.unitPrice * order_details.quantity) FROM order_details LEFT JOIN orders USING(orderID) WHERE DATE_FORMAT(orderDate,'%b/%y') = 'Apr/15' ))*100,1), ' %') AS pct_revenue
FROM order_details
LEFT JOIN products USING(productID)
LEFT JOIN categories USING(categoryID)
LEFT JOIN orders USING(orderID)
LEFT JOIN customers USING(customerID)
WHERE DATE_FORMAT(orderDate,'%b/%y') = 'Apr/15' AND country = 'USA'
GROUP BY 1,2,3
ORDER BY 4 
;
```

## EXPLORING SHIPPING
- NorthWind works with 3 shipping companys (United Package, Speedy Express and Federal Shipping)
- United Package is the company with most number of orders shipped 326 (39.28 %) and they operate most delivering to Germany and USA. 94% of orders they ship on time, with an average of $ 86.64 of freight.

```sql
-- how many sjhipping companys ?
SELECT * FROM shippers;

-- orders by shipper
SELECT
  companyName,
  COUNT(DISTINCT orderID) AS shipped_orders,
  CONCAT(ROUND((COUNT(DISTINCT orderID)/(SELECT COUNT(DISTINCT orderID) FROM orders))*100,2), ' %') AS pct_shipped_orders
FROM shippers
LEFT JOIN orders USING(shipperID)
GROUP BY 1
ORDER BY 2 DESC
;

-- orders shipped on time and delayed 
WITH CTE_orders_on_time 
AS (
	SELECT
    companyName,
    COUNT(DISTINCT orderID) AS orders_on_time
	FROM shippers
	LEFT JOIN orders USING(shipperID)
	WHERE shippedDate <= requiredDate
	GROUP BY 1 WITH ROLLUP
	ORDER BY 2 DESC
),
CTE_orders_delayed
AS (
	SELECT
    companyName,
    COUNT(DISTINCT orderID) AS orders_delayed
	FROM shippers
	LEFT JOIN orders USING(shipperID)
	WHERE shippedDate > requiredDate
	GROUP BY 1 WITH ROLLUP
	ORDER BY 2 DESC
)
SELECT 
  companyName,
  orders_on_time,
  orders_delayed,
  CONCAT(ROUND(100-(orders_delayed/orders_on_time)*100,2),' %') as pct_shipped_on_time
FROM CTE_orders_on_time
LEFT JOIN CTE_orders_delayed USING(companyName)
;
-- another way
WITH CTE_orders_delayed
AS (
SELECT
  companyName,
  COUNT(CASE WHEN (shippedDate < requiredDate) THEN 1 ELSE NULL END) AS orders_on_time,
  COUNT(CASE WHEN (shippedDate > requiredDate) THEN 1 ELSE NULL END) AS orders_delayed
FROM shippers
LEFT JOIN orders USING(shipperID)
GROUP BY 1
)
SELECT
  companyName,
  orders_on_time,
  orders_delayed,
  CONCAT(ROUND(100-(orders_delayed/orders_on_time)*100,2),' %') as pct_shipped_on_time
FROM CTE_orders_delayed
ORDER BY 2 DESC
;

-- orders by shipper per country
SELECT
  shippers.companyName,
  country,
  COUNT(DISTINCT orderID) AS shipped_orders,
  CONCAT(ROUND((COUNT(DISTINCT orderID)/(SELECT COUNT(DISTINCT orderID) FROM orders))*100,2), ' %') AS pct_shipped_orders
FROM shippers
LEFT JOIN orders USING(shipperID)
LEFT JOIN order_details USING(orderID)
LEFT JOIN customers USING(customerID)
GROUP BY 1,2
ORDER BY 3 DESC
;

-- Are shipping costs consistent across providers?
SELECT
  companyName,
  CONCAT('$ ',ROUND(AVG(freight),2)) AS avg_shipping_cost
FROM shippers
LEFT JOIN orders USING(shipperID)
GROUP BY 1
ORDER BY 2 DESC
```
;
