# Introduction

We are looking at some more SQL concepts in this lecture. To introduce some of the concepts, especially joins, we need new datasets. In this case, we are looking at some health data - causes of death in the US, to be precise.

- If you're using Mode Analytics, you should see the tables `drug_overdose` and `mortality`.
- If you're using your own DB, you can use the following sql files to create those tables: [drug_overdose.sql](https://raw.githubusercontent.com/amangup/data-analysis-bootcamp/master/03-SQL_Advanced/drug_overdose.sql) and [mortality_cause.sql](https://raw.githubusercontent.com/amangup/data-analysis-bootcamp/master/03-SQL_Advanced/mortality_cause.sql).

# HAVING

In the last lecture we looked at the `GROUP BY` clause. `HAVING` is a clause which is very useful in many `GROUP BY` queries as it can filter the output of `GROUP BY`. It's like a `WHERE` clause.

Let's look at the following query using `GROUP BY`.

```SQL
SELECT State, sum(Number_of_Drug_Overdose_Deaths) AS overdose_deaths FROM drug_overdose WHERE year=2018 GROUP BY State;
```

- In this query, we have a `WHERE` clause, which applies to the year column. This filters the data _before_ the grouping is done.

Here are some rows from the output:

|state|overdose_deaths|
|-----|---------------|
|CA   |35462          |
|NH   |3186           |
|OR   |3889           |
|US   |480266         |
|TX   |20735          |
|ND   |523            |
|NV   |5129           |
|OH   |31262          |



Now, if we wanted to only get those rows where the number of overdose deaths is more than 10000, than we can use the having clause.

```SQL
SELECT State, sum(Number_of_Drug_Overdose_Deaths) AS overdose_deaths FROM drug_overdose WHERE year=2018 GROUP BY State HAVING sum(Number_of_Drug_Overdose_Deaths) > 10000;
```

This will include only those rows where the column `overdose_deaths` has a value > 10000. Here is the full table:

|state|overdose_deaths|
|-----|---------------|
|CA   |35462          |
|US   |480266         |
|TX   |20735          |
|OH   |31262          |
|KY   |10019          |
|NY   |15985          |
|IN   |11981          |
|MO   |10118          |
|FL   |36910          |
|NC   |15692          |
|GA   |10147          |
|PA   |34524          |
|YC   |10400          |
|MD   |16745          |
|IL   |19667          |
|TN   |12408          |
|NJ   |18269          |
|MI   |18583          |
|MA   |15443          |
|AZ   |11204          |



Note that
- the `HAVING` clause is only applicable to `GROUP BY` queries, and it's condition must use only the aggregate expressions - like `sum()` or `count()` functions.
- You cannot use the alias to refer to the aggregate column mentioned in the `SELECT` clause. In the query above, `HAVING overdose_deaths > 10000` will not work.
- Almost as a counterpoint, using columns which are not included in the `SELECT` clause is perfectly fine. For example, in the query below, the condition in the `HAVING` clause uses the column `Number_of_Deaths` instead of `Number_of_Drug_Overdose_Deaths`.

```SQL
SELECT State, sum(Number_of_Drug_Overdose_Deaths) AS overdose_deaths FROM drug_overdose WHERE year=2018 GROUP BY State HAVING sum(Number_of_Deaths) > 500000;
```

Output:

|state|overdose_deaths|
|-----|---------------|
|CA   |35462          |
|US   |480266         |
|TX   |20735          |
|OH   |31262          |
|NY   |15985          |
|FL   |36910          |
|NC   |15692          |
|GA   |10147          |
|PA   |34524          |
|IL   |19667          |
|TN   |12408          |
|NJ   |18269          |
|MI   |18583          |


# Joins

A Join is a very fundamental concept in SQL and data analysis in general. A join is used to combine two or more datasets into one result. In this lecture, we will use both the tables `drug_overdose` and `mortality` to get a better understanding of joins work.

## Implicit join

In SQL, there are explicit clauses to execute joins, and there are few types of joins. But before we get there, let's see how we can execute a join without learning a new clause.

Both the tables have a column that measures all deaths in each state. Let's compare if those two numbers match.

```SQL
SELECT d.state, d.year, d.Number_of_Deaths, m.All_causes*10 FROM drug_overdose AS d, mortality AS m WHERE m.statecode = d.state AND d.year = m.year ORDER BY d.state;
```

Here is some of the output:

|state|year|number_of_deaths|?column?|
|-----|----|----------------|--------|
|AK   |2015|50086           |43160   |
|AK   |2016|51141           |44940   |
|AL   |2015|607641          |519090  |
|AL   |2016|608611          |524660  |
|AR   |2015|368117          |316170  |
|AR   |2016|366574          |317560  |
|AZ   |2015|642297          |542990  |
|AZ   |2016|679217          |566450  |
|CA   |2015|3056952         |2592060 |
|CA   |2016|3139581         |2622400 |

- Let's look at the `FROM` clause first. 
   - We have listed two tables here separated by a comma. 
   - Secondly, we have given each table an alias, so that we can refer to the columns from these tables.
- When we refer to the columns, we are using the notation `table_name_or_alias.column_name`.
  - This is needed if both tables have a column with the same name, and you want to refer to that column in one of the tables.
  - Recommended approach is to always use this notation if your query deals with multiple tables.
- Joins must have a criteria, and in this query, we have specified that in the `WHERE` clause. We want to get data from both tables which are from the same state and the same year.

- A note on data: the `mortality` table data is represented as a multiple of 10. To get the actual count, we need to multiply by 10.
- It is clear that the data in `mortality` is consistently lower than the data in the `drug_overdose` column.

To illustrate my next point, I need to run a couple of other queries:

```SQL
SELECT count(distinct(year)) FROM drug_overdose;
```

Output:


| count |
|------ |
| 4     |


```SQL
SELECT count(distinct(year)) FROM mortality;
```

| count |
|------ |
| 18     |


- One table has data for 4 years, and the other table has data for 18 years. Yet, in the output, we are getting data for only 2 years (2015 and 2016).
- These two years are the only two years which are overlapping in the year column of both the tables.
- In this implicit style of join, you only get those rows where the condition is met in both tables. This type of join is called **Inner Join**, and implicit joins can be thought of as alternate syntax to achieve this type of join.

## Types of Join

We are going to use venn diagrams to represent the types of joins. Each table is represented by a circle, and if the circle is filled, the output will contain rows from that table. The following diagram shows the 4 types of join.

![Join types](https://raw.githubusercontent.com/amangup/data-analysis-bootcamp/master/03-SQL_Advanced/joins.png)


|Join Clause| explanation |
|-----|-------|
| (INNER) JOIN  | Data is returned from both tables when only the condition is met |
| LEFT (OUTER) JOIN | Data is returned from all rows from left table, and rows from right table when the condition is met |
| RIGHT (OUTER) JOIN | Data is returned from all rows from right table, and rows from left table when the condition is met |
| FULL (OUTER) JOIN | Data is returned from all rows from both tables, and rows where the condition is met has columns from both tables filled |

- The part in paranthesis can be ommitted in SQL queries.

Let's look at each of these one by one.

### INNER JOIN

We have already looked at inner join - so let's look at the same example, but written using the `INNER JOIN` syntax.

```SQL
SELECT d.state, d.year, d.Number_of_Deaths, m.All_causes*10 FROM drug_overdose AS d INNER JOIN mortality AS m ON m.statecode = d.state AND d.year = m.year ORDER BY d.state;
```

The output is identical to what we got before:

|state|year|number_of_deaths|?column?|
|-----|----|----------------|--------|
|AK   |2015|50086           |43160   |
|AK   |2016|51141           |44940   |
|AL   |2015|607641          |519090  |
|AL   |2016|608611          |524660  |
|AR   |2015|368117          |316170  |
|AR   |2016|366574          |317560  |
|AZ   |2015|642297          |542990  |
|AZ   |2016|679217          |566450  |
|CA   |2015|3056952         |2592060 |
|CA   |2016|3139581         |2622400 |


- In the query, instead of listing both tables in the `FROM` clause, we instead use the syntax `FROM left_table INNER JOIN right table`.
- Instead of the `WHERE` clause, we use the `ON` keyword to describe our join condition.

That's it. With those two syntax changes, we are now using the JOIN clause syntax.

### LEFT JOIN

Let's just change the keyword `INNER` to `LEFT` in the previous query, and look at the output:

```SQL
SELECT d.state, d.year, d.Number_of_Deaths, m.All_causes*10 FROM drug_overdose AS d INNER JOIN mortality AS m ON m.statecode = d.state AND d.year = m.year ORDER BY d.state, d.year;
```

The first few rows of output are:

|state|year|number_of_deaths|?column?|
|-----|----|----------------|--------|
|AK   |2015|50086           |43160   |
|AK   |2016|51141           |44940   |
|AK   |2017|51597           |        |
|AK   |2018|30182           |        |
|AL   |2015|607641          |519090  |
|AL   |2016|608611          |524660  |
|AL   |2017|623769          |        |
|AL   |2018|372909          |        |
|AR   |2015|368117          |316170  |
|AR   |2016|366574          |317560  |
|AR   |2017|378076          |        |
|AR   |2018|225148          |        |
|AZ   |2015|642297          |542990  |
|AZ   |2016|679217          |566450  |
|AZ   |2017|686329          |        |
|AZ   |2018|420441          |        |

- You can see that we are getting 4 rows per state, instead of 2 which we were getting with the `INNER JOIN`.
- In `LEFT JOIN`, the output contains the rows from the left table even if the right table doesn't have any row that satisfies the join condition (in this case, the `mortality` table doesn't have data for 2017 and 2018).

### RIGHT JOIN

- It's an exercise. Run a `RIGHT JOIN` query and look at the output.

### FULL OUTER JOIN

- It's an exercise. Run a `FULL OUTER JOIN` query and look at the output.

## Join performance

- Join queries can take a lot of time to execute if large tables are involved.
  - Finding and reading a matching row takes time.
- Inner joins are faster than outer joins.

If you're writing queries which run repeatedly, and it has joins, it's usually worth spending time to optimize the query for performance. We will not go into those details in this lecture.

# Recursive queries

Recursive SQL query is a query which _runs_ a query on top of the output of another query. The most convenient way to do that is to write out the full base query in the `FROM` clause of the _2nd level_ query. A 2 level query has this format:

```SQL
SELECT <columns>
FROM (<SELECT ...>) AS base_result;
```

In the inner join, we compared the death counts from the two death sources. Let's look at the ratio of those counts (to get a better idea of the magnitude of difference) state by state.

```SQL
SELECT state, sum(deaths1::float)/sum(deaths2) 
FROM (SELECT d.state, d.year, d.Number_of_Deaths AS deaths1, m.All_causes*10 AS deaths2 
      FROM drug_overdose AS d, mortality AS m 
      WHERE m.statecode = d.state AND d.year = m.year) AS comparison 
GROUP BY state
ORDER BY state;
```

The query has become complicated enough to use a multi-line syntax.

- The `base query` is the same as the one used in the implicit join, but I have added aliases to some columns to make it easy to refer to them in the `2nd level` query.
- The `2nd level` query need not concern itself with the base query - it can just assume that it's a table and you write as you would any other query.
  - In case the `FROM` clause is succeeded by another query, aliasing the output of that query is mandatory. In the query above, we have named it `comparison`.
- The outer query computes the ratio of counts summed across all years, separately for each state.
   - Unlike Python, in PostgreSQL dividing two integer types results in a 0. So, I have done a cast to convert a column to float. This syntax is PostgreSQL specific.


The output looks like this:

|state|?column?          |
|-----|------------------|
|AK   |1.1490011350737799|
|AL   |1.165271377245509 |
|AR   |1.1593123254382782|
|AZ   |1.191154095760023 |
|CA   |1.1883364720412084|
|CO   |1.2078384926704475|
|CT   |1.1933904188087363|
|DC   |1.3880904319741623|
|DE   |1.1863599908340972|
|FL   |1.2027921860943322|
|GA   |1.1927440044617958|
|HI   |1.2061322043157607|
|IA   |1.185601474517231 |


- Recursive queries are very common, and I find that many times it can be simpler to understand even if sometimes the same result can be achieved using without it (typically by using more advanced SQL features).


# Exercises for this dataset

We have not really explored this dataset properly. Let's do a few exercises:

1. Using the `drug_overdose` table, find how much % of the deaths due to drug overdoses are due to Opiods (for the states for which we have this data).
2. Using the `drug_overdose` table, find in which states can you attribute more than 80% of drug overdose deaths to opioids.
3. Using both the tables, find if more people died by suicide or by drug overdose in the US in 2015 and 2016.
4. While looking at the data in `mortality` table, it seems that death numbers are generally increasing over time. We want to know if there are some states where this is not true. 
  - Find all states where the year in which maximum deaths occurred is not 2016.



   




