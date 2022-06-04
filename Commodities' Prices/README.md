# Commodities' Prices case study

![image](https://user-images.githubusercontent.com/77529445/171989172-57e16cbc-e7ee-4c0a-aa33-a302f780d8cc.png)
- Id acts as a unique identifier in all three tables. 
- Commodity_id in the price_details table and id in commodities_info table will be used to apply to join between these tables. 
- Region_Id in the price_details table and id in region_info table will be used to apply to join between these tables. 



- Note that there is only one price for a particular commodity variety for a month. This means that the price does not change within the month for a commodity corresponding to a commodity_id.
***

## Case study questions
- Determine the common commodities between the Top 10 costliest commodities of 2019 and 2020.
- What is the maximum difference between the prices of a commodity at one place vs the other for the month of June 2021? Which commodity was it for?
- Arrange the commodities in an order based on the number of variants in which they are available, with the highest one shown at the top, which is the third commodity in the list.
- In a state with the least number of data points available, which commodity has the highest number of data points available?
- What is the price variation of commodities for each city from January 2019 to December 2020? Which commodity has seen the highest price variation and in which city?

***

### Query 1
**Determine the common commodities between the Top 10 costliest commodities of 2019 and 2020.**

```sql
Input: price_details: Id, Commodity_Id, Date, Retail_Price, commodities_info: Id, Commodity
Expected output: Commodity; Take distinct to remove duplicates
```

- Step 1: Filter price details for year 2019 and get the top 10 costliest entries based on price
- Step 2: Filter price details for year 2020 and get the top 10 costliest entries based on price
- Step 3: Perform inner join and remove duplicates to get unique commodity ids from both datasets
- Step 4: Join the table from step 4 with commodities_info table to get the final result.


```sql
USE commodity_db;

WITH 2019_summary AS
  (SELECT commodity_id,
          MAX(retail_price) AS price
   FROM price_details
   WHERE YEAR(date)=2019
   GROUP BY commodity_id
   ORDER BY price DESC
   LIMIT 10),
     2020_summary AS
  (SELECT commodity_id,
          MAX(retail_price) AS price
   FROM price_details
   WHERE YEAR(date)=2020
   GROUP BY commodity_id
   ORDER BY price DESC
   LIMIT 10),
     common_commodities AS
  (SELECT commodity_id
   FROM 2019_summary
   INNER JOIN 2020_summary USING(commodity_id))
SELECT DISTINCT commodity AS common_commodity_list
FROM common_commodities AS cc
JOIN commodities_info AS ci ON cc.commodity_id=ci.id;
```
![image](https://user-images.githubusercontent.com/77529445/171987937-3b64c1a4-830e-4b5e-a7f9-ced0c2cd6205.png)

***

### Query 2
**What is the maximum difference between the prices of a commodity at one place vs the other for the month of June 2021? Which commodity was it for?**

```sql
Input: price_details: Id, Region_Id, Commodity_Id, Date and Retail_Price;  commodities_info: Id and Commodity
Expected output: Commodity | price difference;  Retain the info for the highest difference.
```

- Step 1: Filter price details for June 2020 in Date column
- Step 2: Aggreagtion - MIN(retail_price), MAX(retail_price) group by commodity
- Step 3: Compute the difference between max and min retail price
- Step 4: Sort in descending order of price difference, retain the top most row.


```sql
WITH june_prices AS
  (SELECT commodity_id,
          MIN(retail_price) AS Min_price,
          MAX(retail_price) AS Max_price
   FROM price_details
   WHERE date BETWEEN '2020-06-01' AND '2020-06-30' -- WHERE month(date) = 6 and year(date) = 2020
GROUP BY commodity_id)
SELECT ci.commodity,
       Max_price-Min_price AS price_difference
FROM june_prices AS jp
JOIN commodities_info AS ci ON jp.commodity_id=ci.id
ORDER BY price_difference DESC
LIMIT 1;
```
![image](https://user-images.githubusercontent.com/77529445/171988246-13b5f182-d4ca-4fd2-8e40-df8845b6e170.png)

***

### Query 3
**Arrange the commodities in an order based on the number of variants in which they are available, with the highest one shown at the top, which is the third commodity in the list.**

```sql
Input: commodities_info: Commodity and Variety
Expected output: Commodity | Variety count; Sort in descending order of Variety count
```

- Step 1: Aggregation - COUNT(DISTINCT variety), group by Commodity
- Step 2: Sort the final table in descending order of variety count


```sql
SELECT commodity,
       count(DISTINCT variety) AS variety_count
FROM commodities_info
GROUP BY Commodity
ORDER BY variety_count DESC ;
```
![image](https://user-images.githubusercontent.com/77529445/171988577-94d74af4-1d4d-4de1-a771-7d424ba53a09.png)


***

### Query 4
**In a state with the least number of data points available, which commodity has the highest number of data points available?**

```sql
Input: price_details: Id, region_id, commodity_id region_info: Id and State commodities_info: Id and Commodity
Expected output: commodity;  Expecting only one value as output
```

- Step 1: Join region information and price details by using the Region_Id from price_details with Id from region_info.
- Step 2: From the result of Step 1, perform aggregation – COUNT(Id), group by State.
- Step 3: Sort the result based on the record count computed in Step 2 in ascending order; Filter for the top State.
- Step 4: Filter for the state identified from Step 3 from the price_details table.


```sql
WITH raw_data AS
  (SELECT pd.id,
          pd.commodity_id,
          ri.state
   FROM price_details AS pd
   LEFT JOIN region_info AS ri ON pd.region_id = ri.id), -- state with lowest data points
state_record_count AS
  (SELECT state,
          COUNT(id) AS state_wise_datapoints
   FROM raw_data
   GROUP BY state
   ORDER BY state_wise_datapoints
   LIMIT 1), -- commodities with the highest number of data points available from the state with lowest data points
-- All data points has 3 record_count
commodity_list AS
  (SELECT commodity_id,
          COUNT(id) AS record_count
   FROM raw_data
   WHERE state IN
       (SELECT DISTINCT state
        FROM state_record_count)
   GROUP BY commodity_id
   ORDER BY record_count DESC) -- Record count irrespective of variety, group by commodity and sum(record_count)

SELECT commodity,
       SUM(record_count) AS record_count
FROM commodity_list AS cl
LEFT JOIN commodities_info AS ci ON cl.commodity_id = ci.id
GROUP BY commodity
ORDER BY record_count DESC
LIMIT 1;
```
![image](https://user-images.githubusercontent.com/77529445/171988834-c6273f35-80e7-4284-af87-f49fbb1c03b8.png)

***

### Query 5
****

```sql

```



```sql

```

***

