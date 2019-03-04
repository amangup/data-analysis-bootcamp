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

# D: Correlations

The main reason I chose these datasets was to see how these different markers correlate with each other. Does Internet penetration increase with GDP? Are there countries where more people are getting access to internet while drinking water availability is not improving?

- To quantify correlation, we will use the [Pearson correlation cofficient (wikipedia link)](https://en.wikipedia.org/wiki/Pearson_correlation_coefficient). 
- This is a value which ranges from -1 to +1 : 
   - -1 signifies that the series whose correlation is being measured have opposite growth patterns, 
   - 0 means one is growing independent of the other, and 
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
