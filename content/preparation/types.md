+++
title = 'Types'
date = 2024-04-14T23:53:52-07:00
draft = false
weight = 3
+++

### Data dictionary

A document that outlines all the database data columns with defined types, estimated values, source, and overall relationships. 

### Types of data

- **Strings**: A series of letters. CHAR for 2 characters, VARCHAR for variable length, and TEXT for unlimited length. It's best to restrict length when feasible.
- **Numeric**: Whole numbers or floats.
- **Booleans**: True or false.
- **Date time**: Can be more or less specific for the exact time.

### 1st, 2nd, and 3rd party data

- **1st**: collected by the organization.
- **2nd**: collected on behalf of the organization like from a vendor.
- **3rd**: collected from an unaffiliated organization so no control over data format.

### Limits and sampling

To reduce computation costs, it is a good idea to add `LIMIT` to the end of your query. You can also use a sampling method to grab a random portion of the data. If you have randomly assigned indices, you can take every 100th entry with this command (about 1% of data):

    WHERE mod(integer_order_id, 100) = 6 -- Takes entry # 106, 206, 306, etc...

### Profiling distributions

It's a good idea to check the distribution of each column before starting your analysis:

    SELECT example, count(*) 
    AS quantity 
    FROM Table
    GROUP BY 1
    ;

Note that `count` takes duplicate values so for unique views, use `count distinct`.

    SELECT orders, count(*) as num_customers
    FROM
    (
        SELECT customer_id, count(order_id) as orders
        FROM orders
        GROUP BY 1
    ) a
    GROUP BY 1
    ;

###### Binning

You can separate data into a histogram using different bin widths for different visualizations. One way to do this is with the `CASE` command which is analogous to if/then statements.

    SELECT
    case when order_amount <= 100 then 'up to 100'
        when order_amount <= 500 then '100 - 500'
        else '500+' end as amount_bin
    ,case when order_amount <= 100 then 'small'
        when order_amount <= 500 then 'medium'
        else 'large' end as amount_category
    ,count(customer_id) as customers
    FROM orders
    GROUP BY 1,2
    ;

Here we are binning customer_id based on their order amount.

You can also bin by quartile (4 subsets), decile (10 subsets), or n-tile (defined subset number).

    SELECT ntile
    ,min(order_amount) as lower_bound
    ,max(order_amount) as upper_bound
    ,count(order_id) as orders
    FROM
    (
        SELECT customer_id, order_id, order_amount
        ,ntile(10) over (order by order_amount) as ntile
        FROM orders
    ) a
    GROUP BY 1
    ;

### Window functions

Window functions apply some function to multiple rows that are partitioned by some rule. For example, if you want to partition rows by some categorical value and then take the sum of each category separately.

For example:

    SELECT
        office_id,
        sales,
        RANK() OVER (PARTITION BY office_id ORDER BY sales DESC) AS rank
    FROM sales_data;

Or this example:

    SELECT
        date,
        sales,
        AVG(sales) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg
    FROM sales_data;


