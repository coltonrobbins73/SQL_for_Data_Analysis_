+++
title = 'Data Quality Control'
date = 2024-04-15T02:00:37-07:00
draft = false
weight = 4
+++

The best models will fail if the initial data is not of good quality so data QC is one of the most important tasks of a data analyst.

### Identify duplicates

One easy way to check for duplicates is to use the count function and then check if there are any counts greater than 1 indicating a duplicate row. Define a subset of the columns by using `SELECT` and check if that combination of columns has any repeated values between any of the rows.

    SELECT records, count(*)
    FROM
    (
        SELECT column_a, column_b, column_c..., count(*) AS records
        FROM...
        GROUP BY 1,2,3...
    ) a
    WHERE records > 1
    GROUP BY 1
    ;

It is a good idea to revisit this function several times during your analysis, especially after something like a cartesian `JOIN` which has a higher risk of incorporating duplicates.

### Avoiding duplicates

When joining tables, there can be duplicate entries that are formed. An example would be customer_ids that have made a transaction. If a customer makes more than one transaction their id, will show up multiple times in the join table. These artifacts are due more to the query dynamics rather than bad data. There are two methods to eliminate this error on joins.

Using `distinct` commands:

    SELECT distinct a.customer_id, a.customer_name, a.customer_email
    FROM customers a
    JOIN transactions b on a.customer_id = b.customer_id
    ;

Or using the `GROUP BY` command which natively orders items distinctly:

    SELECT a.customer_id, a.customer_name, a.customer_email
    FROM customers a
    JOIN transactions b on a.customer_id = b.customer_id
    GROUP BY 1,2,3
    ;

### Data cleaning

###### Standardizing data

Data in a single column might exist as several variations which all mean the same thing. To standardize several variations, you can use `CASE`:

    CASE when gender = 'F' then 'Female'
        when gender = 'female' then 'Female'
        when gender = 'femme' then 'Female'
        else gender
        end as gender_cleaned

When you have many data variants that need to be updated regularly, a lookup table is more efficient than `CASE`. Use the lookup table to map variants to a standardized value and then use a `JOIN` to the main data table.

Note that sometimes, it is easier to correct inconsistencies upstream of data collection to make sure that all incoming data is already standardized.

###### Enriching data

Sometimes data can be enhanced through categorization or transformation. For instance, in a 10-scale scoring system, we can codify different scoring ranges for each row in a table:

    SELECT response_id
    ,likelihood
    ,case when likelihood <= 6 then 'Detractor'
        when likelihood <= 8 then 'Passive'
        else 'Promoter'
        end as response_type
    FROM nps_responses
    ;

If you need to categorize specific values that are not adjacent, you can specify specific values for your case statement:

    case when likelihood in (0,1,2,3,4,5,6) then 'Detractor'
        when likelihood in (7,8) then 'Passive'
        when likelihood in (9,10) then 'Promoter'
        end as response_type

###### Dummy variables

For categorical binary data, it is sometimes useful to set up dummy variables that are either 1 or 0 to better track frequency distributions of the data.

###### Flagging data

Sometimes you might want to derive values from columns using functions or thresholds to better annotate rows. This can be done with `CASE`:

    SELECT customer_id
    ,max(case when fruit = 'apple' then 1
        else 0
        end) as bought_apples
    ,max(case when fruit = 'orange' then 1
        else 0
        end) as bought_oranges
    FROM ...
    GROUP BY 1
    ;

Or a more complicated thresholding scheme:

    SELECT customer_id
    ,max(case when fruit = 'apple' and quantity > 5 then 1
        else 0
        end) as loves_apples
    ,max(case when fruit = 'orange' and quantity > 5 then 1
        else 0
        end) as loves_oranges
    FROM ...
    GROUP BY 1
    ;

### Changing types

When importing data sometimes it is of an inappropriate format like string representation of numeric data. In these cases, we can recast the data type with `::` as in `price::float`. Some data types cannot be cast to other data types like VARCHAR to BOOL.

### Missing data

The term **null** represents missing data. The null placeholder can be present from importing a sparse dataset or even making a join without complete correspondence among columns.

One major challenge with null values is that they do cause most functions to fail if they are included in the process. You can transform null values using either `CASE` or even `COALESCE` which takes in two or more arguments and returns the first argument that is not null.

    coalesce(num_orders,0)
    coalesce(address,'Unknown')
    coalesce(column_a,column_b)
    coalesce(column_a,column_b,column_c)

Note that null will not be retrieved using conditional statements unless explicitly asked for. For example, if you use `CASE` to find fields that do not contain a certain value, any value with null will not be considered and thus be omitted.

###### Imputation

Missing data can be approximated using the existing data on hand. Defining the average data value with median or mean can be a good replacement when there are missing entries. Another method is to use a fill-forward or fill-backward technique which uses values from adjacent entries. Not that adjacent filling only really works when the entry order has some logical relationship.

### Adding data dimensions

Data dimensions are essentially look-up tables that can be used to calculate new data entries. One of the most common data dimensions is a date dimension table that contains standard information for each date for 10s to hundreds of years in the future or past. A date dimension can thus be used to quickly retrieve attributes about a specified date.

### Analyzing data shape

The shape of your data refers to the column and row dimensions of your various.

###### Pivot tables

A pivot table is a way to summerize a data table using values from one of your tables. Typically, the chosen pivot columns will have duplicate entries accross multiple rows so that summary functions can be used against multiple rows (think groupby for python). For example, a table that records sales can be pivotted using a sum aggregation on a column that records quarters. The resulting pivot table will view selected columns with additional columns for each quarter that show the sum of sales for that quarter. Multiple columns can be pivotted on in a single table to show different combinations of values.
