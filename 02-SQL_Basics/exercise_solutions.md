### Problem 1
#####How many survey responses are in the table?

```SQL
SELECT count(*) FROM public.av_survey WHERE status = 'COMPLETE';
```


### Problem 2
#####How many survey responses were gotten online vs in-person?

```SQL
SELECT source_type, count(source_type) FROM public.av_survey GROUP BY source_type;
```


### Problem 3
#####How many survey responses were collected on each day in the duration of the survey?

```SQL
SELECT DISTINCT (date_trunc('day', end_date)) AS survey_days, COUNT(end_date) AS surveys_collected  
FROM public.av_survey 
WHERE status = 'COMPLETE' 
GROUP BY date_trunc('day', end_date) 
ORDER BY survey_days DESC;
```


### Problem 4
#####What is the average time taken to complete the survey?

If we take a simple average of completed surveys we can use the following:

```SQL
SELECT avg(end_date - start_date) FROM public.av_survey WHERE status = 'COMPLETE';
```

If we order by the time taken to complete the survey we see that there are several values that are outliers. For example, the longest time it took to complete the survey was 11 days. These outliers are skewing our average time calculation. 

```SQL
SELECT end_date, start_date, (end_date - start_date) as difference FROM public.av_survey WHERE status = 'COMPLETE' ORDER BY difference DESC;
```

Let's select a reasonable time difference like anything less than an hour and recalculate the average again.

```SQL
SELECT avg(end_date - start_date) FROM public.av_survey WHERE status = 'COMPLETE' AND (end_date - start_date) < '1:00';
```

This returns an average time of 6 minutes and 29 seconds. 


### Problem 5
#####How many distinct zip codes were included in the survey?

```SQL
SELECT count(distinct(zipcode)) FROM public.av_survey WHERE zipcode <> 0;
```


### Problem 6
#####How many people in total have interacted with an Autonomous Vehicle?

```SQL
SELECT count(*) FROM public.av_survey WHERE interactbicycle = 'Yes' OR interactpedestrian = 'Yes';
```

### Problem 7
#####For each type of response to `FeelingsProvingGround`, count how many people gave that response?

```SQL
SELECT feelingsprovingground, count(feelingsprovingground) 
FROM public.av_survey 
WHERE feelingsprovingground IS NOT NULL 
GROUP BY feelingsprovingground;
```


### Problem 8
#####Same question as Q7, but only from people who have interacted with an Autonomous Vehicle.

```SQL
SELECT feelingsprovingground, count(feelingsprovingground) FROM public.av_survey WHERE interactbicycle = 'Yes' OR interactpedestrian = 'Yes' GROUP BY feelingsprovingground;
```


### Problem 9
#####For each type of response to `AVSafetyPotential`, count how many people gave that response?

```SQL
SELECT avsafetypotential, count(avsafetypotential) FROM public.av_survey WHERE avsafetypotential IS NOT NULL GROUP BY avsafetypotential ORDER BY count(avsafetypotential) DESC;
```


### Problem 10
#####Same question as Q9, but only from people who are **Mostly familiar** or **Extremely familiar** with the AV technology (`FamiliarityTechnology` column)

```SQL
SELECT avsafetypotential, count(avsafetypotential) FROM public.av_survey WHERE familiaritytechnology = 'Extremely familiar' OR familiaritytechnology = 'Mostly familiar' GROUP BY avsafetypotential;
```


### Problem 11
#####How many people are in favor of any type of regulation on Autonomous Vehicles?

```SQL
SELECT COUNT(*) FROM public.av_survey WHERE 'Yes' IN (regulationsharedata, regulationschoolzone, regulationspeed, regulationtesting);
```


### Problem 12
#####How many people who are following AVs in the news (`PayingAttentionAV` response is **To a moderate extent** or **To a large extent**) are not familiar with the Technology (`FamiliarityTechnology` is _lesser_ than **Somewhat familiar** and _below_)?

```SQL
SELECT COUNT(payingattentionav) FROM public.av_survey 
WHERE (payingattentionav = 'To a moderate extent' OR payingattentionav = 'To a large extent') 
AND (familiaritytechnology = 'Mostly Unfamiliar' OR familiaritytechnology = 'Not familiar at all');
```
