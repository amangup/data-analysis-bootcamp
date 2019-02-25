# Exercises for this dataset

1. Using the `drug_overdose` table, find how much % of the deaths due to drug overdoses are due to Opiods (for the states for which we have this data).
  ```SQL
SELECT state, sum(opioids_t400t404t406::float)/sum(number_of_drug_overdose_deaths) as drug_ratio 
FROM public.drug_overdose 
WHERE opioids_t400t404t406 is NOT NULL
GROUP BY state 
ORDER BY drug_ratio DESC
  ```
2. Using the `drug_overdose` table, find in which states can you attribute more than 80% of drug overdose deaths to opioids.

```SQL
SELECT state, drug_ratio
FROM (SELECT state, sum(opioids_t400t404t406::float)/sum(number_of_drug_overdose_deaths) as drug_ratio 
      FROM public.drug_overdose 
      GROUP BY state) AS original
WHERE drug_ratio > 0.8 AND drug_ratio IS NOT NULL
```

3. Using both the tables, find if more people died by suicide or by drug overdose in the US in 2015 and 2016.
```SQL
SELECT d.year, sum(suicide)*10 as suicide_deaths, 
      sum(number_of_drug_overdose_deaths) as overdose_deaths, 
      CASE WHEN sum(suicide)*10 > sum(number_of_drug_overdose_deaths) THEN 'suicide'
            ELSE 'overdose' END AS majority_deaths
FROM public.mortality as m 
JOIN public.drug_overdose as d ON d.year = m.year 
GROUP BY d.year
```

4. While looking at the data in `mortality` table, it seems that death numbers are generally increasing over time. We want to know if there are some states where this is not true. 
```SQL
SELECT state,year, sum(all_causes) FROM public.mortality GROUP BY state, year ORDER BY state,year
```
5. Find all states where the year in which maximum deaths occurred is not 2016.
```SQL
SELECT md.state, md.year,md.all_causes
FROM public.mortality md
INNER JOIN
    (SELECT state, MAX(all_causes) AS max_death
    FROM public.mortality
    GROUP BY state) groupedmd 
ON md.state = groupedmd.state
AND md.all_causes = groupedmd.max_death
WHERE year != 2016
```



