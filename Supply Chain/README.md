# Supply chain case study


The tables present in the supply chain data set are as follows:
- Orders: Order_Id is the primary key.
- Ordered_items: Order_Item_Id is the primary key, and Customer_Id is the unique identifier for the customers who placed the order and acts as a foreign key to the customer_info table. Category_Id is the unique identifier of the category of the product and acts as a foreign key to the category table.
- Product_info: Product_Id is the primary key.
- customer_info: Id is the primary key.
- category: Id is the primary key.
- department: Id is the primary key.


In the .csv file provided, the Data Dictionary sheet contains all the relevant information, describing each column for a particular table. 


***

### Query 1
**Get the number of orders by the Type of Transaction excluding the orders shipped from Sangli and Srinagar. Also, exclude the SUSPECTED_FRAUD cases based on the Order Status, and sort the result in the descending order based on the number of orders.**

```sql
Input: Orders table (Order_Id, Type, Order_City, Order_Status)
Expected output: Type of Transaction | Orders (Sorted in the descending order of Orders)
```

- Step 1: Filter out ‘Sangli’ and ‘Srinagar’ from the city column of the data.
- Step 2: Filter out ‘SUSPECTED_FRAUD ’ from the order_status column of the data.
- Step 3: Aggregation – COUNT(order_id), GROUP BY Transaction_type
- Step 4: Sort the result in the descending order of Orders

```sql
SELECT TYPE AS transaction_type,
               COUNT(order_id) AS orders
FROM orders
WHERE Order_City NOT IN ('Sangli',
                         'Srinagar')
  AND Order_Status <> 'SUSPECTED_FRAUD'
GROUP BY Transaction_type
ORDER BY Orders DESC;
```
![image](https://user-images.githubusercontent.com/77529445/171622205-db813890-9a93-4747-bfb9-8c9941b0e125.png)

***

### Query 2
**Get the list of the Top 3 customers based on the completed orders along with the following details:**
- Customer Id
- Customer First Name
- Customer City
- Customer State
- Number of completed orders
- Total Sales**


```sql
Input: Orders table (Order_Id and Order_Status), Ordered_items table (Sales), Customer_info table (Id, First_Name, City, State)
Expected output: Customer Id | Customer First Name | Customer City | Customer State | Completed orders | Total Sales
```
- Step 1: Join orders and order_items to get order_id level sales.
- Step 2: Filter out ‘COMPLETE’ orders from the order_status column of the orders table.
- Step 3: Join the result from Step 2 with the Customers table and create a customer id level summary.
- Step 4: Apply Aggregation – COUNT(order_id), SUM(Sales)  and GROUP BY Customer Id, Customer First Name, Customer City and Customer State.

```sql
WITH order_summary AS
  (SELECT ord.order_id,
          ord.customer_id,
          SUM(sales) AS ord_sales
   FROM orders AS ord
   JOIN ordered_items AS itm USING(order_id)
   WHERE ord.order_status='COMPLETE'
   GROUP BY ord.order_id)
SELECT Id AS Customer_id,
       First_Name AS Customer_First_Name,
       City AS Customer_City,
       State AS Customer_State,
       COUNT(DISTINCT order_id) AS Completed_Orders,
       SUM(ord_sales) AS Total_Sales
FROM order_summary AS ord
INNER JOIN customer_info AS cust ON ord.customer_id=cust.id
GROUP BY Customer_id,
         Customer_First_Name,
         Customer_City
ORDER BY Completed_Orders DESC,
         Total_Sales DESC
LIMIT 3;
```

***


