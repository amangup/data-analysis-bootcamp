# Problems

## A: Drinking water availability

1. Write a query which outputs the **total** drinking water availability for the most recent year in data, for each region. Sort the output by the value of availability in descending order.

```SQL
SELECT region_name, 
  sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END) AS safe_total
FROM public.mytable
GROUP BY region_name
HAVING sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END) IS NOT NULL
ORDER BY safe_total DESC;
```

2. **Rural/Urban ratio**: For regions where both rural and urban data are available, for most recent year, find the ratio of rural/urban drinking water availability. Sort the table by this ratio, in ascending order.

```SQL
SELECT region_name, 
  sum(CASE WHEN year = 2015 THEN safe_drinking_water_rural::float/safe_drinking_water_urban ELSE NULL END) AS ru_ratio
FROM public.mytable
WHERE safe_drinking_water_rural IS NOT NULL AND safe_drinking_water_urban IS NOT NULL
GROUP BY region_name
ORDER BY ru_ratio;
```

3. **Growth**: For regions where **total** drinking water availability value is available, find the growth in drinking water availability. Growth is defined as the ratio of latest value to the oldest value.

```SQL
SELECT region_name, 
  sum(CASE WHEN year = 2005 THEN safe_drinking_water_total ELSE NULL END) AS old_year_2005,
  sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END) AS latest_year_2015,
  sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END)/sum(CASE WHEN year = 2005 THEN safe_drinking_water_total ELSE NULL END) AS growth_ratio 
FROM public.mytable
GROUP BY region_name
HAVING sum(CASE WHEN year = 2005 THEN safe_drinking_water_total ELSE NULL END) IS NOT NULL 
AND sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END) IS NOT NULL
ORDER BY growth_ratio;
```

## B: GDP

1. Write a query which outputs the GDP per capita for the most recent year in data, for each region. Sort the output by the value of GDP per capita in descending order.

```SQL
SELECT region_name,
  sum(CASE WHEN year = 2016 THEN gdp_per_capita ELSE NULL END) AS gdp_per_cap_2016
FROM public.gdp
GROUP BY region_name
HAVING sum(CASE WHEN year = 2016 THEN gdp_per_capita ELSE NULL END) IS NOT NULL
ORDER BY gdp_per_cap_2016 DESC;
```

2. **Comparison with average growth**: The _average growth_ of GDP per capita can be calculated by comparing the oldest and the most recent value for the "Total, all countries" region. We want to know which countries have grown faster than average, and which have grown slower than average. Write a query that outputs two rows - one row contains the list of all regions where growth is lower than average, and another which contains the regions where growth is higher than average.

```SQL
SELECT 
  CASE WHEN avg_growth_per_region < 3.6518 THEN 'Low Growth' ELSE 'High Growth' END AS growth_category,
  string_agg(region_name,', ') AS countries
FROM 
  (SELECT 
  region_name,
  sum(CASE WHEN year = 1985 THEN gdp_per_capita ELSE NULL END) AS old,
  sum(CASE WHEN year = 2016 THEN gdp_per_capita ELSE NULL END) AS recent,
  sum(CASE WHEN year = 2016 THEN gdp_per_capita ELSE NULL END)::float/sum(CASE WHEN year = 1985 THEN gdp_per_capita ELSE NULL END) AS avg_growth_per_region
  FROM gdp 
  WHERE region_name != 'Total, all countries or areas'
  GROUP BY region_name
  HAVING sum(CASE WHEN year = 2016 THEN gdp_per_capita ELSE NULL END)::float/sum(CASE WHEN year = 1985 THEN gdp_per_capita ELSE NULL END) IS NOT NULL) AS growth_avg
GROUP BY growth_category;
```



# C: Internet penetration

1. Write a query which outputs the internet penetration for the most recent year in data, for each region. Sort the output by the value of availability in descending order.

```SQL
SELECT region_name, 
  sum(CASE WHEN year = 2016 THEN percent_internet_penetration ELSE NULL END) AS net_pen_2016 
FROM public.internet_penetration
GROUP BY region_name
HAVING sum(CASE WHEN year = 2016 THEN percent_internet_penetration ELSE NULL END) IS NOT NULL
ORDER BY net_pen_2016 DESC;
```

2. **Growth**: Find the growth in internet penetration for each region. Growth is defined as the ratio of latest value to the oldest value.

```SQL
SELECT region_name, 
  sum(CASE WHEN year = 2000 THEN percent_internet_penetration ELSE NULL END) AS old_net_pen,
  sum(CASE WHEN year = 2016 THEN percent_internet_penetration ELSE NULL END) AS recent_net_pen,
  sum(CASE WHEN year = 2016 THEN percent_internet_penetration ELSE NULL END)/sum(CASE WHEN year = 2000 THEN percent_internet_penetration ELSE NULL END)::float AS growth_net_pen
FROM public.internet_penetration
GROUP BY region_name
HAVING sum(CASE WHEN year = 2016 THEN percent_internet_penetration ELSE NULL END)/sum(CASE WHEN year = 2000 THEN percent_internet_penetration ELSE NULL END)::float IS NOT NULL 
AND sum(CASE WHEN year = 2000 THEN percent_internet_penetration ELSE NULL END) != 0
ORDER BY growth_net_pen DESC;
```


## Problems

1. Is there any region where internet connectivity is higher than total drinking availability?

```SQL
SELECT i.region_name, i.year, i.percent_internet_penetration, w.safe_drinking_water_total
FROM public.internet_penetration as i
JOIN public.mytable as w
ON i.year = w.year AND i.region_name = w.region_name
WHERE w.safe_drinking_water_total IS NOT NULL
AND percent_internet_penetration > safe_drinking_water_total;
```


2. List all countries where growth in internet connectivity is lower than growth in drinkable water availability. 

```SQL
SELECT water.region_name, growth_net_pen_ratio, growth_water_ratio
FROM
  (SELECT region_name, 
    sum(CASE WHEN year = 2005 THEN percent_internet_penetration ELSE NULL END) AS old_net_pen_2005,
    sum(CASE WHEN year = 2015 THEN percent_internet_penetration ELSE NULL END) AS recent_net_pen_2015,
    sum(CASE WHEN year = 2015 THEN percent_internet_penetration ELSE NULL END)/sum(CASE WHEN year = 2005 THEN percent_internet_penetration ELSE NULL END)::float AS growth_net_pen_ratio
  FROM public.internet_penetration
  GROUP BY region_name
  HAVING sum(CASE WHEN year = 2015 THEN percent_internet_penetration ELSE NULL END)/sum(CASE WHEN year = 2005 THEN percent_internet_penetration ELSE NULL END)::float IS NOT NULL 
  AND sum(CASE WHEN year = 2005 THEN percent_internet_penetration ELSE NULL END) != 0
  ORDER BY growth_net_pen_ratio DESC) AS internet,

  (SELECT region_name, 
    sum(CASE WHEN year = 2005 THEN safe_drinking_water_total ELSE NULL END) AS old_year_2005,
    sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END) AS latest_year_2015,
    sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END)/sum(CASE WHEN year = 2005 THEN safe_drinking_water_total ELSE NULL END) AS growth_water_ratio 
  FROM public.mytable
  GROUP BY region_name
  HAVING sum(CASE WHEN year = 2005 THEN safe_drinking_water_total ELSE NULL END) IS NOT NULL 
  AND sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END) IS NOT NULL
  ORDER BY growth_water_ratio) AS water
WHERE water.region_name = internet.region_name
AND growth_net_pen_ratio < growth_water_ratio
ORDER BY water.region_name;
```

3. Find the correlation between GDP per capita and GDP in current prices. Does the value look strange? What explains the value?

```SQL
SELECT corr(gdp_per_capita,gdp_in_current_prices_millions) 
FROM public.gdp;
```
The correlation coefficient is .078. This value doesn't look strange because countries have vastly different population sizes in relation to their total GDP.

For example, you could have a relatively small country, with a small population but have a high gdp per capita, but with a small population, e.g. Monaco.

On the other hand, you could have a country with a large amount of people, but have a low gdp per capita, but due to the sheer size of the population, the total gdp of the country is quite large e.g.China in the 1990s  

Two countries could have similar GDP's, but their GDP per capita may be very different, which further explains no correlation between the two.

Let's look at two countries with a similar gdp: South Africa and Denmark. According to the gdp table, in 2014, both these countries had a gdp of roughly 350,000 million. However, their gdp per capita was 6,433 for South Africa and 62,323 for Denmark, a difference by a factor of 10. 

What explains this difference? A quick google search of these countries' populations reveals that South Africa has a population of 56 million people, roughly 10 times that of Denmark with a population of 5.7 million.

Put simply, the correlation coefficient between `gdp_per_capita` and `gdp_in_current_prices_millions` is close to zero because there is virtually no correlation between population size and GDP.



4. Find the correlation between growth of internet connectivity and total drinkable water availability.

```SQL
SELECT corr(internet.growth_net_pen_ratio, water.growth_water_ratio)
FROM
  (SELECT region_name, 
    sum(CASE WHEN year = 2005 THEN percent_internet_penetration ELSE NULL END) AS old_net_pen_2005,
    sum(CASE WHEN year = 2015 THEN percent_internet_penetration ELSE NULL END) AS recent_net_pen_2015,
    sum(CASE WHEN year = 2015 THEN percent_internet_penetration ELSE NULL END)/sum(CASE WHEN year = 2005 THEN percent_internet_penetration ELSE NULL END)::float AS growth_net_pen_ratio
  FROM public.internet_penetration
  GROUP BY region_name
  HAVING sum(CASE WHEN year = 2015 THEN percent_internet_penetration ELSE NULL END)/sum(CASE WHEN year = 2005 THEN percent_internet_penetration ELSE NULL END)::float IS NOT NULL 
  AND sum(CASE WHEN year = 2005 THEN percent_internet_penetration ELSE NULL END) != 0
  ORDER BY growth_net_pen_ratio DESC) AS internet,

  (SELECT region_name, 
    sum(CASE WHEN year = 2005 THEN safe_drinking_water_total ELSE NULL END) AS old_year_2005,
    sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END) AS latest_year_2015,
    sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END)/sum(CASE WHEN year = 2005 THEN safe_drinking_water_total ELSE NULL END) AS growth_water_ratio 
  FROM public.mytable
  GROUP BY region_name
  HAVING sum(CASE WHEN year = 2005 THEN safe_drinking_water_total ELSE NULL END) IS NOT NULL 
  AND sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END) IS NOT NULL
  ORDER BY growth_water_ratio) AS water
WHERE internet.region_name = water.region_name;
```

5. Find the correlation between growth of internet connectivity and growth of GDP per capita

```SQL
SELECT corr(internet.growth_net_pen_ratio, gpc.growth_gdp_per_cap)
FROM
  (SELECT region_name, 
    sum(CASE WHEN year = 2005 THEN percent_internet_penetration ELSE NULL END) AS old_net_pen_2005,
    sum(CASE WHEN year = 2016 THEN percent_internet_penetration ELSE NULL END) AS recent_net_pen_2016,
    sum(CASE WHEN year = 2016 THEN percent_internet_penetration ELSE NULL END)/sum(CASE WHEN year = 2005 THEN percent_internet_penetration ELSE NULL END)::float AS growth_net_pen_ratio
  FROM public.internet_penetration
  GROUP BY region_name
  HAVING sum(CASE WHEN year = 2016 THEN percent_internet_penetration ELSE NULL END)/sum(CASE WHEN year = 2005 THEN percent_internet_penetration ELSE NULL END)::float IS NOT NULL 
  AND sum(CASE WHEN year = 2005 THEN percent_internet_penetration ELSE NULL END) != 0
  ORDER BY growth_net_pen_ratio DESC) AS internet,

  (SELECT  region_name,
    sum(CASE WHEN year = 2005 THEN gdp_per_capita ELSE NULL END) AS old_gdp_per_cap_2005,
    sum(CASE WHEN year = 2016 THEN gdp_per_capita ELSE NULL END) AS new_gdp_per_cap_2016,
    sum(CASE WHEN year = 2016 THEN gdp_per_capita ELSE NULL END)::float/sum(CASE WHEN year = 2005 THEN gdp_per_capita ELSE NULL END) AS growth_gdp_per_cap
  FROM public.gdp
  GROUP BY region_name
  HAVING sum(CASE WHEN year = 2016 THEN gdp_per_capita ELSE NULL END)::float/sum(CASE WHEN year = 2005 THEN gdp_per_capita ELSE NULL END) IS NOT NULL
  ORDER BY growth_gdp_per_cap DESC) AS  gpc
WHERE internet.region_name = gpc.region_name;

```

6. Find the correlation between growth of GDP per capita and drinking water availability.

```SQL
SELECT corr(gpc.growth_gdp_per_cap, water.growth_water_ratio)
FROM
  (SELECT  region_name,
    sum(CASE WHEN year = 2005 THEN gdp_per_capita ELSE NULL END) AS old_gdp_per_cap_2005,
    sum(CASE WHEN year = 2015 THEN gdp_per_capita ELSE NULL END) AS new_gdp_per_cap_2016,
    sum(CASE WHEN year = 2015 THEN gdp_per_capita ELSE NULL END)::float/sum(CASE WHEN year = 2005 THEN gdp_per_capita ELSE NULL END) AS growth_gdp_per_cap
  FROM public.gdp
  GROUP BY region_name
  HAVING sum(CASE WHEN year = 2015 THEN gdp_per_capita ELSE NULL END)::float/sum(CASE WHEN year = 2005 THEN gdp_per_capita ELSE NULL END) IS NOT NULL
  ORDER BY growth_gdp_per_cap DESC) AS gpc,

  (SELECT region_name, 
    sum(CASE WHEN year = 2005 THEN safe_drinking_water_total ELSE NULL END) AS old_year_2005,
    sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END) AS latest_year_2015,
    sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END)/sum(CASE WHEN year = 2005 THEN safe_drinking_water_total ELSE NULL END) AS growth_water_ratio 
  FROM public.mytable
  GROUP BY region_name
  HAVING sum(CASE WHEN year = 2005 THEN safe_drinking_water_total ELSE NULL END) IS NOT NULL 
  AND sum(CASE WHEN year = 2015 THEN safe_drinking_water_total ELSE NULL END) IS NOT NULL
  ORDER BY growth_water_ratio DESC) AS water
WHERE water.region_name = gpc.region_name;
```
