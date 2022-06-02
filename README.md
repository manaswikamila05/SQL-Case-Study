# Structured problem solving in SQL

The **issue tree framework** is an easy-to-go method to structure the problems into breakout problems and pseudo-defining the links to avoid any confusion.

- Breaking down the problem to smaller chunks by using an issue tree
- Solving the minor problems with a step-by-step algorithm approach
  - Identifying the tables that are required
  - Understanding the type of data that they contain and their link (relationships - PK and FK) to the other tables
  - Determining the type of operation to perform (use of right functions or keywords)
  - Writing a query to solve the problem
- Stitching the smaller solutions to solve the overall problem


The following steps summarise the manner in which we should process our thinking and visualisation while writing a query:
- Visualising the required output
  - Which columns will be there?
  - Which rows will be selected? Defined by the usage of filters (WHERE and HAVING)
  - What would be the granularity of the data? Defined by aggregations (GROUP BY)
- Using CTEs or subqueries for better organisation of the query
- Validating individual sections by running them independently and validating the results
- Combining or stitching the smaller pieces together, to obtain the entire solution
- Checking for rows explosion while performing JOINs
