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

### Query 3
**Get the order count by the Shipping Mode and the Department Name. Consider departments with at least 40 closed/completed orders.**

```sql
Input: orders (order_id, Shipping_Mode and Order_Status) ordered_items, product_info, department (name)
Expected output: Shipping Mode | Department Name | Orders (Retain departments with at least 40 closed/completed orders)
```
- Step 1: Join orders, ordered_items, product_info and department to get all the departments and orders associated with them
- Step 2: Filter out ‘COMPLETE’ and ‘CLOSED’ from the order_status column of the orders table.
- Step 3: Apply Aggregation – COUNT(order_id), GROUP BY department name
- Step 4: In the table mentioned in Step 3, filter out COUNT(order_id)>=40
- Step 5: From Step 1, perform aggregation – COUNT(order_id), GROUP BY Shipping mode and department name. Retain only those department names that were left over after the filter was applied in Step 4.


```sql
-- Step 1: Perform join on required tables
WITH ord_dept_summary AS
  (SELECT ord.order_id,
          ord.shipping_mode,
          d.name AS department_name,
          order_status
   FROM orders AS ord
   JOIN ordered_items AS ord_itm USING(order_id)
   JOIN product_info AS p ON ord_itm.item_id=p.product_id
   JOIN department AS d ON p.department_id=d.id),
-- Step 2 : Retrieve number of orders 'Completed/Closed' per department
     dept_summary AS
  (SELECT department_name,
          count(order_id) AS order_count
   FROM ord_dept_summary
   WHERE order_status IN ('COMPLETE',
                          'CLOSED')
   GROUP BY department_name),
-- Step 3: Filter the departments with at least 40 closed/completed orders. 
     dept_list AS
  (SELECT department_name
   FROM dept_summary
   WHERE order_count>=40)
-- Step 4: Get the order count by the Shipping Mode and the Department Name and consider the deaprtments from step 3
SELECT *,
       count(order_id) AS Orders
FROM ord_dept_summary
WHERE department_name IN
    (SELECT *
     FROM dept_list)
GROUP BY shipping_mode,
         department_name
ORDER BY Orders DESC;
```
![image](https://user-images.githubusercontent.com/77529445/171829310-c8da5135-a903-4081-a3e6-f6cde2ba3952.png)

***
### Query 4
**Create a new field as shipment compliance based on Real_Shipping_Days and Scheduled_Shipping_Days.**
It should have the following values:
- Cancelled shipment: If the Order Status is SUSPECTED_FRAUD or CANCELED
- Within schedule: If shipped within the scheduled number of days 
- On time: If shipped exactly as per schedule
- Up to 2 days of delay: If shipped beyond schedule but delayed by 2 days
- Beyond 2 days of delay: If shipped beyond schedule with a delay of more than 2 days

```sql
Input: orders (order_id, Real_Shipping_Days, Scheduled_Shipping_Days and Shipping_Mode)
Expected output: order_id | shipment_compliance | shipping_mode | Number of delayed orders
```

- Step 1: Create a shipment compliance column based on the given criteria.
- Step 2: Test and confirm if all the cases are handled. Check for null values too.
- Step 3: Filter out the delayed orders only.
- Step 4: Apply Aggregation – COUNT(order_id), GROUP BY shipping mode and sort in descending order of order count and retain the top-most row.

```sql
WITH shipment_compliance_summary AS
  (SELECT order_id,
          Real_Shipping_Days,
          Scheduled_Shipping_Days,
          Shipping_Mode,
          order_status,
          CASE
              WHEN order_status = 'SUSPECTED_FRAUD'
                   OR order_status = 'CANCELED' THEN 'Cancelled shipment'
              WHEN Real_Shipping_Days<Scheduled_Shipping_Days THEN 'Within schedule'
              WHEN Real_Shipping_Days=Scheduled_Shipping_Days THEN 'On Time'
              WHEN Real_Shipping_Days<=Scheduled_Shipping_Days+2 THEN 'Upto 2 days of delay'
              WHEN Real_Shipping_Days>Scheduled_Shipping_Days+2 THEN 'Beyond 2 days of delay'
              ELSE 'Others'
          END AS shipment_compliance
   FROM orders)
SELECT shipping_mode,
       count(order_id) AS Orders
FROM shipment_compliance_summary
WHERE shipment_compliance in ('Upto 2 days of delay',
                              'Beyond 2 days of delay')
GROUP BY shipping_mode
ORDER BY Orders DESC
LIMIT 1;
```
![image](https://user-images.githubusercontent.com/77529445/171833095-35a91154-fc66-47fc-ba21-21f98ba93e47.png)

***

### Query 5
**An order is canceled when the status of the order is either CANCELED or SUSPECTED_FRAUD. Obtain the list of states by the order cancellation% and sort them in the descending order of the cancellation%.**
Definition: Cancellation% = Cancelled order / Total orders”

```sql
Input: Orders (Order_Id, Order_State and Order_Status)
Expected output: Order State | Cancellation % (Sort in the descending order of cancellation%)
```

- Step 1: Filter out ‘CANCELED’ and ‘SUSPECTED_FRAUD’ from the order_status column of the orders table.
- Step 2: From the result of Step 1, perform aggregation – COUNT(order_id), GROUP BY Order_State.
- Step 3: Create separate aggregation of the orders table to get the total orders - COUNT(order_id), GROUP BY Order_State.
- Step 4: Join the results of Step 2 and Step 3 on Order_State.
- Step 5: Create a new column with the calculation of Cancellation percentage = Cancelled Orders / Total Orders.
- Step 6: Sort the final table in the descending order of Cancellation percentage.


```sql
WITH cancelled_orders_summary AS
  (SELECT Order_State,
          COUNT(order_id) AS cancelled_orders
   FROM Orders
   WHERE order_status='CANCELED'
     OR order_status='SUSPECTED_FRAUD'
   GROUP BY Order_State),
     total_orders_summary AS
  (SELECT Order_State,
          COUNT(order_id) AS total_orders
   FROM Orders
   GROUP BY Order_State)
SELECT Order_state,
       cancelled_orders,
       total_orders,
       ROUND(cancelled_orders/total_orders*100, 2) AS CANCELLATION_PERCANTAGE
FROM total_orders_summary AS t
LEFT JOIN cancelled_orders_summary AS c USING(Order_state)
ORDER BY CANCELLATION_PERCANTAGE DESC;
```

***
![image](https://user-images.githubusercontent.com/77529445/171834653-24c6c51f-45bb-47da-aaf4-94558d88dadd.png)



