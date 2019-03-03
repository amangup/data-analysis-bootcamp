# Data description

This data consists of three different markers of progress in countries across the world - the GDP (in table `gdp`), internet penetration (in table `internet_penetration`), and drinking water availability (in table `drinking_water_availability`). These tables are quite simple - spend some time to familiarize yourself with the data before you read on.

# Some new concepts

There are some concepts that you need to understand before these exercises can be attempted.

## CASE statement

`CASE` statement is used for conditional logic in SQL. You can [read about it here](https://github.com/amangup/data-analysis-bootcamp/blob/master/04-SQL_Writing/sql_writing.md#case-statement).

It's quite straightforward, but just be careful to include all the keywords required for the syntax.

## GROUP BY with conditions

In the lecture, we understood how `GROUP BY` works - creating groups using the distinct values of a column (or multiple columns). The `GROUP BY` clause can also be used with a custom expression. For example, here is a query:

```SQL
SELECT CASE WHEN year < 2000 THEN 'OLD' ELSE 'NEW' END, avg(gdp_per_capita) 
FROM gdp 
GROUP BY CASE WHEN year < 2000 THEN 'OLD' ELSE 'NEW' END;
```

- Try it out and see how this works.
- Remember that we said when using the `SELECT` clause along with `GROUP BY`, you can either use the columns or expression mentioned in `GROUP BY`, or an aggregate function. In this case, the whole conditional statement in `GROUP BY` needs to be copied exactly in the `SELECT` clause, otherwise SQL will give you an error.

## Pivot Tables in SQL

- The **Pivot table** transformation is one of the essential data transformations which any data analyst should know. If you've tried the Pivot table in feature in MS Excel or any other spreadsheet software, you know what we are talking about.

- Pivot table is used to take an existing column, and create multiple columns, one for each of its values. 
- For example, let's look at the `internet_penetration` table. It has an year column, and we are interested in knowing how the value has changed over the course of many years. To make it easy to compare, we would like to data for all years in the same row.
- That makes it easier to compare values manually, as the numbers that we want to compare are visible side by side.
- If we want to write a data transformation in SQL to understand that comparison better (as we'll do in the exercises), it's much easier if all the data is present _in the same row_.

Here is a query that gives us the data for 2016, 2010 and 2005 in their own columns (here we have chosen only a few values of year to transform to columns).

```SQL
SELECT 
	region_name, 
	sum(CASE WHEN year=2016 THEN percent_internet_penetration ELSE NULL END) AS y2016,
	sum(CASE WHEN year=2010 THEN percent_internet_penetration ELSE NULL END) AS y2010,  
	sum(CASE WHEN year=2005 THEN percent_internet_penetration ELSE NULL END) AS y2005 
FROM internet_penetration 
GROUP BY region_name;
```

and here are the first few rows of the output:


|          region_name          | y2016 | y2010 | y2005 
| ------------------------------|-------|-------|-------
| Europe                        |  77.5 |  61.4 |  40.2
| Micronesia (Fed. States of)   |  33.4 |  20.0 |  11.9
| Nicaragua                     |  24.6 |  10.0 |   2.6
| United Rep. of Tanzania       |  13.0 |   2.9 |   1.1
| Tonga                         |  40.0 |  16.0 |   4.9
| Nepal                         |  19.7 |   7.9 |   0.8
| United States Virgin Islands  |  59.6 |  31.2 |  27.3
| Greenland                     |  68.5 |  63.0 |  57.7
| Bolivia (Plurin. State of)    |  39.7 |  22.4 |   5.2


- It is an exercise for you to understand why this query works. However, I'll mention a couple of points to keep in mind as you do that.
- The `sum()` function will ignore any null values provided to it, and only add the non-null values. If all values are null, it returns null.
- The `CASE` statement works within `SELECT` clause with `GROUP BY` as long as it's outcome is fed as an input to an aggregate function.

# Problems

## A: Drinking water availability

1. Write a query which outputs the **total** drinking water availability for the most recent year in data, for each region. Sort the output by the value of availability in descending order.

2. **Rural/Urban ratio**: For regions where both rural and urban data are available, for most recent year, find the ratio of rural/urban drinking water availability. Sort the table by this ratio, in ascending order.

3. **Growth**: For regions where **total** drinking water availability value is available, find the growth in drinking water availability. Growth is defined as the ratio of latest value to the oldest value.

## B: GDP

1. Write a query which outputs the GDP per capita for the most recent year in data, for each region. Sort the output by the value of GDP per capita in descending order.

2. **Comparison with average growth**: The _average growth_ of GDP per capita can be calculated by comparing the oldest and the most recent value for the "Total, all countries" region. We want to know which countries have grown faster than average, and which have grown slower than average. Write a query that outputs two rows - one row contains the list of all regions where growth is lower than average, and another which contains the regions where growth is higher than average.

Here is how the output looks like (list of countries is truncated)

| Growth Category | Countries
| --- | ---
 LOW GROWTH  | Brunei Darussalam, Syrian Arab Republic, San Marino, Colombia, Ukraine, Saudi Arabia, Sao Tome and Principe, Latvia, Dem. People's Rep. Korea, Algeria, ...
 HIGH GROWTH | Australia and New Zealand, Bangladesh, Luxembourg, Sweden, Dominican Republic, Ireland, Cambodia, Singapore, Portugal, Malta, Albania, Oceania, Maldives, Israel, Malaysia, State of Palestine, Cyprus, China, ...
 
You will need to know this function in PostgreSQL: [string_agg()](https://www.dbrnd.com/2016/09/postgresql-string_agg-to-concatenate-string-per-each-group-like-sql-server-stuff-string-aggregation-function/). This is an aggregate function that allows you to list out all strings.

# C: Internet penetration

1. Write a query which outputs the internet penetration for the most recent year in data, for each region. Sort the output by the value of availability in descending order.

2. **Growth**: Find the growth in internet penetration for each region. Growth is defined as the ratio of latest value to the oldest value.

#D: Correlations

The main reason I chose these datasets was to see how these different markers correlate with each other. Does Internet penetration increase with GDP? Are there countries where more people are getting access to internet while drinking water availability is not improving?

- To quantify correlation, we will use the [Pearson correlation cofficient (wikipedia link)](https://en.wikipedia.org/wiki/Pearson_correlation_coefficient). 
- This is a value which ranges from -1 to +1 : 
   - -1 signifies that the series whose correlation is being measured have opposite growth pattern, 
   - 0 means one is growing independent of each other, and 
   - 1 means that their growth is perfectly aligned.

- To calculate the Pearson correlation coefficient. The [list of all aggregate functions](https://www.postgresql.org/docs/10/functions-aggregate.html) includes the descriptions of the `corr()` function in PostgreSQL, which we will use.

- As we've done previously in this exercise set, we will define **Growth** as the ratio of most recent value to the oldest value. But, as we are going to compare growth across different markers (gdp, internet connectivity, etc.), we need to make sure we measure growth over the same period of time, for each marker.
  - For these exercises, take the longest overlapping time period to measure growth.


## Problems

1. Is there any region where internet connectivity is higher than total drinking availability?
2. List all countries where growth in internet connectivity is lower than growth in drinkable water availability. 
3. Find the correlation between GDP per capita and GDP in current prices. Does the value look strange? What explains the value?
4. Find the correlation between growth of internet connectivity and total drinkable water availability.
5. Find the correlation between growth of internet connectivity and growth of GDP per capita
6. Find the correlation between growth of GDP per capita and drinking water availability.








 