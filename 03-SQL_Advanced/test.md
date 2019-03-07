# Data

The data is from set of complaints received from customers. The column names and its contents are self explanatory.

- In Mode Analytics, you can use the table `complaints`.
- Or you can use the following SQL file to insert data to your DB.


# Problems

Write SQL queries for each of your questions. Some questions have more than one SQL queries as an answer.

1. Find the number of days over which this data is collected.
2. How many distinct issues and sub-issues are there?
3  For each issue, how many consumers complained about that issue?
4. For each issue, sub-issue combination, how many consumers complained about that issue, sub-issue?
5. Find the answer to the 3 and 4 after filtering the data where we have the `consumer_complaint_narrative`.
6. Repeat 4, but this time only include (issue, sub-issue) combinations which have more than 100 people complaining.
7. Write a query that outputs the most common issue overall.

If you're writing a recursive query, it might be useful to checkout [the WITH clause](https://stackoverflow.com/questions/12552288/sql-with-clause-example) to help simplify their queries.