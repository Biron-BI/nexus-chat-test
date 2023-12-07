As a data analyst I should use a custom SQL-like language to query a datawarehouse.
Here is the differences from a standard sql language :
 - aggregate functions like "SUM", "AVG", etc are replaced by a special "metric" function that take a single string argument composed of 2 parts, joined by a dot :
  - the first part is the code of a "view" : the concept of "view" are like datamarts, i.e. groups of metrics that share common business sense. Available "view" codes come from a fixed-list named "the view list" that I would provide further.
  - the second part is the code of a metric : a "metric" is a predefined calculation. Available "metric" codes come from a fixed-list named "the metrics list" that I would provide further.
  - Example `metric('transactions_order_date.billing_with_vat')` represents the metric with the code "billing_with_vat" within the view with the code "transactions_order_date"
- the FROM clause must always be `FROM datamodel`, there is no other table-like than "datamodel".
- when a dimension/slicer should be used to segment the result, this should be done by calling a function named "dimension" that take a single string argument : the code of a dimension. Available dimension codes come from a fixed-list named "the dimensions list" that I would provide further. Example `dimension('delivery_country_code')`.
the WHERE clause must have a filter on the refDate column with a mandatory lower and upper boundary. Example `WHERE refDate BETWEEN '2023-01-01' AND '2023-02-28'`


A complete example :
```
SELECT dimension('transactions_order_date.delivery_country_code') as deliveryCountry, metric('billing_with_vat'), metric('no_orders')
FROM datamodel
WHERE refDate BETWEEN '2023-01-01' AND '2023-02-28'
GROUP BY deliveryCountry
```
This corresponds to the billing with VAT and the number of orders, grouped by the delivery country for Jan. & Feb 2023.

The "the metrics list" can be found in a TSV formated file in [metricList.tsv](metricList.tsv), composed of the following columns in order :
- viewCode: the related view code
- metricCode: the metric code
- name: a human readable name
- description: a human readable business description

Find “the views list” can be found in a TSV formated file in [viewList.tsv](viewList.tsv), composed of the following columns in order :
- code: the view code, that would be used in the query
- name: a human readable name,
- description: a human readable business description
- priority: a priority score where the highest value is the highest priority

Find “the dimension list”can be found in a TSV formated file in [dimensionList.tsv](dimensionList.tsv), composed of the following columns in order :
- code: the dimension code, that would be used in the query
- name: a human readable name
- description: a human readable business description

When we have to anwser some questions we have to generate an SQL statement that follow these rules :
- First try to find the corresponding metrics: use the name and the description to try and find the most likely. We can also use the code of the metric though it should be considered as a third option. If multiple options are possible, always choose the metric from the view with the highest priority.
- To find the right dimension, use the name and the description to try and find the most likely. We can also use the code of the dimension though it should be considered as a third option.
- If you can't find any valid metric, view or dimension, tell me and do not generate a query ;
- never ever use a "SUM", "AVG" or "COUNT" aggregate function, the "metric" function will do it automatically
- never use an unlisted metric, view or dimension code, we are not allowed to invent a code ;
- add an order clause that sorts by the selected dimensions if any ;
