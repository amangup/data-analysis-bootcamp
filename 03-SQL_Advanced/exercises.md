# Problems

## A: Drinking water availability

1. Write a query which outputs the **total** drinking water availability for the most recent year in data, for each region. Sort the output by the value of availability in descending order.
```SQL
SELECT region_name, year, safe_drinking_water_total as total_drinking_water
FROM public.drinking_water_availability
WHERE year=2015 AND safe_drinking_water_total is NOT NULL
ORDER BY total_drinking_water DESC
```

2. **Rural/Urban ratio**: For regions where both rural and urban data are available, for most recent year, find the ratio of rural/urban drinking water availability. Sort the table by this ratio, in ascending order.
```SQL 
SELECT region_name as region, year, (safe_drinking_water_rural::float/safe_drinking_water_urban) as rural_urban_ratio
FROM public.drinking_water_availability
WHERE safe_drinking_water_rural is NOT NULL AND safe_drinking_water_urban is not NULL AND year=2015
ORDER BY rural_urban_ratio
```

3. **Growth**: For regions where **total** drinking water availability value is available, find the growth in drinking water availability. Growth is defined as the ratio of latest value to the oldest value.
```SQL
SELECT region, y2005/y2015 as growth
FROM (SELECT region_name as region, 
        sum(CASE WHEN year=2005 THEN safe_drinking_water_total ELSE NULL END) AS y2005,
        sum(CASE WHEN year=2015 THEN safe_drinking_water_total ELSE NULL END) AS y2015
      FROM public.drinking_water_availability
      GROUP By region_name) as origin
WHERE y2005 IS NOT NULL AND y2015 IS NOT NULL
ORDER BY growth DESC
```

## B: GDP

1. Write a query which outputs the GDP per capita for the most recent year in data, for each region. Sort the output by the value of GDP per capita in descending order.
```SQL
SELECT region_name, year, gdp_per_capita
FROM public.gdp
WHERE year=2016
ORDER BY gdp_per_capita DESC
```

2. **Comparison with average growth**: The _average growth_ of GDP per capita can be calculated by comparing the oldest and the most recent value for the "Total, all countries" region. We want to know which countries have grown faster than average, and which have grown slower than average. Write a query that outputs two rows - one row contains the list of all regions where growth is lower than average, and another which contains the regions where growth is higher than average.

Here is how the output looks like (list of countries is truncated)

| Growth Category | Countries
| --- | ---
 LOW GROWTH  | Brunei Darussalam, Syrian Arab Republic, San Marino, Colombia, Ukraine, Saudi Arabia, Sao Tome and Principe, Latvia, Dem. People's Rep. Korea, Algeria, ...
 HIGH GROWTH | Australia and New Zealand, Bangladesh, Luxembourg, Sweden, Dominican Republic, Ireland, Cambodia, Singapore, Portugal, Malta, Albania, Oceania, Maldives, Israel, Malaysia, State of Palestine, Cyprus, China, ...
 
You will need to know this function in PostgreSQL: [string_agg()](https://www.dbrnd.com/2016/09/postgresql-string_agg-to-concatenate-string-per-each-group-like-sql-server-stuff-string-aggregation-function/). This is an aggregate function that allows you to list out all strings.

```SQL
SELECT region_name, (y2016::float/y1985) as growth,
      (CASE WHEN (y2016::float/y1985)>=3.652 THEN 'High Growth' ELSE 'Low Growth' END) as comparision_growth
FROM (SELECT region_name, 
      sum(CASE WHEN year=1985 THEN gdp_per_capita ELSE NULL END) AS y1985,
      sum(CASE WHEN year=2016 THEN gdp_per_capita ELSE NULL END) AS y2016
      FROM public.gdp
      GROUP BY region_name) as original
WHERE y2016 IS NOT NULL AND y1985 IS NOT NULL
```
**UGH HOW DO YOU DO THIS PROBLEM!!!!!!**

# C: Internet penetration

1. Write a query which outputs the internet penetration for the most recent year in data, for each region. Sort the output by the value of availability in descending order.
```SQL
SELECT region_name, percent_internet_penetration as percent_internet_penetration_2016
FROM public.internet_penetration
WHERE year=2016
ORDER BY percent_internet_penetration DESC
```

2. **Growth**: Find the growth in internet penetration for each region. Growth is defined as the ratio of latest value to the oldest value.
```SQL
SELECT region_name, 
      SUM(CASE WHEN year = 2016 THEN percent_internet_penetration ELSE NULL END)/
      SUM(CASE WHEN year = 2000 THEN percent_internet_penetration ELSE NULL END)::float as growth
FROM public.internet_penetration
GROUP BY region_name
HAVING SUM(CASE WHEN year = 2000 THEN percent_internet_penetration ELSE NULL END) != 0 AND
      SUM(CASE WHEN year = 2016 THEN percent_internet_penetration ELSE NULL END)/
      SUM(CASE WHEN year = 2000 THEN percent_internet_penetration ELSE NULL END)::float IS NOT NULL
ORDER BY growth DESC
```

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
```SQL
SELECT region_name, year, 
      (CASE WHEN percent_internet > water THEN 'Internet' ELSE NULL END) as winner
FROM (SELECT i.region_name, i.year, i.percent_internet_penetration as percent_internet, w.safe_drinking_water_total as water
      FROM public.internet_penetration as i
      INNER JOIN public.drinking_water_availability as w
      ON i.year = w.year and i.region_name = w.region_name
      WHERE i.percent_internet_penetration IS NOT NULL AND
            w.safe_drinking_water_total IS NOT NULL) as original
WHERE CASE WHEN percent_internet>water THEN 'Internet' ELSE NULL END IS NOT NULL
```
2. List all countries where growth in internet connectivity is lower than growth in drinkable water availability. 
3. Find the correlation between GDP per capita and GDP in current prices. Does the value look strange? What explains the value?
4. Find the correlation between growth of internet connectivity and total drinkable water availability.
5. Find the correlation between growth of internet connectivity and growth of GDP per capita
6. Find the correlation between growth of GDP per capita and drinking water availability.
