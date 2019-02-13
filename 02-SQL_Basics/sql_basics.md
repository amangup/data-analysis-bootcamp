# What is SQL?

- SQL stands for **Structured Query Language**, and was designed in 1974 to manage data in Relational Database Systems. We'll explain the **Relational database model** in next class.
- The word **database** denotes a collection of data, organized using some structure.
- SQL, first of all, is a _language_, i.e., it is used to communicate your intent with the computer system.
- The intent is to _manage_ the data. SQL can be used to _add_ new data, _modify_ existing data, and most importantly for analysts, _read_ already stored data.
- As an analyst, we use SQL primarily to access the data, and apply transformations to it.
- Code written in SQL is a collection of **queries**.
- The database _system_ is the interpreter for this language - it's like a butler who understands your query and executes your wishes.
- In 1986/1987, the specification of the language was standardized. Which means, the language words are not proprietary to one database developer, and everyone abides (mostly) by this one standard of SQL.

## SQL vs Data System

One thing that I want to highlight is the separation of SQL with the underlying system that stores the data. Shown below is a stock image of an "accountant".



- The paper containing numbers represents the data system.
- The person represents the SQL coder.
- The calculator represents the SQL execution engine.
- If you replace the paper with some other notebook or the model of the calculator, the person will still be able to get through their analysis.

- SQL written according to international standards will work with all database systems.
- Not only that, new models of data storage (especially big data models) are also using SQL as the interface between the analyst and their data.

# Describing data

Whenever we get some _table_ of data, our first goal is to _understand_ the contents of that table. In this tutorial, we are using some data which contains the ranking of top songs in the years 2016-2018. The following `.sql` file can be used to create a table and insert the data in a SQL database. The table called `top_songs` is created.

## Snapshot of the table
A snapshot of a table is the list of columns with a few sample rows that serve as examples of what those columns can contain. Here is the query for that:

```SQL
SELECT * FROM top_songs LIMIT 5;
```

### The output of this query is:


| id | year | position | artist                  | song              | score     | us | uk | de | fr | ca | au |
|----|------|----------|-------------------------|-------------------|-----------|----|----|----|----|----|----|
| 1  | 2016 | 1        | Sia                     | Cheap Thrills     | 14801.661 | 1  | 2  | 1  | 1  | 1  | 6  |
| 2  | 2016 | 2        | Drake, Wizkid & Kyla    | One Dance         | 14641.871 | 1  | 1  | 1  | 1  | 1  | 1  |
| 3  | 2016 | 3        | Justin Bieber           | Love Yourself     | 13580.652 | 1  | 1  | 3  | 4  | 1  | 1  |
| 4  | 2016 | 4        | The Chainsmokers & Daya | Don't Let Me Down | 12936.962 | 3  | 2  | 6  | 19 | 4  | 3  |
| 5  | 2016 | 5        | Twenty One Pilots       | Stressed Out      | 12347.973 | 2  | 12 | 3  | 2  | 3  | 2  |

- This table has the top 100 songs (column `position` contains the position of each song) for each year.
- The songs are described by the title (in the column `song`) and `artist`.
- The `score` column is a custom score that's been used to rank these songs.
- The columns for some contries (like `uk`, `fr`) denotes the highest Billboard position that song reached in that country, in that year. 


### Let's understand the query:

- The query starts with the keyword `SELECT`, which you can use to specify what columns you want the query to return. A value of `*` (asterisk) will return all columns in the table.
- The `FROM` keyword is used to specify the table from which to _select_ the columns.

If we stop here (query: `SELECT * FROM top_songs`), this will return _all_ data from the table `top_songs`. That will many times return 100s or 1000s of rows, which can't possibly go through manually. Hence, we add another keyword to the query:

- The `LIMIT` keyword will simply truncate the output to the number of rows specified, from the top. Note that data stored in the table may not have an order among its rows. For example, in this output, we see the rows with ids 1-5 in the output, but this is not guaranteed. If we _want_ some order in the output, we should specify that explicitly (which we'll learn soon).

- The keywords `SELECT`, `FROM` and `LIMIT` cannot be placed in the query arbitrarily. `FROM top_songs SELECT * LIMIT 5` doesn't work. As we introduce new keywords, it's important to keep their placement in mind.

### Selecting specific columns

As mentioned before, `SELECT` query can be used to fetch specific columns. Let's do that.

```SQL
SELECT year, position, artist, song FROM top_songs LIMIT 10;
```

The output of this query is:

| year | position | artist                  | song                   |
|------|----------|-------------------------|------------------------|
| 2016 | 1        | Sia                     | Cheap Thrills          |
| 2016 | 2        | Drake, Wizkid & Kyla    | One Dance              |
| 2016 | 3        | Justin Bieber           | Love Yourself          |
| 2016 | 4        | The Chainsmokers & Daya | Don't Let Me Down      |
| 2016 | 5        | Twenty One Pilots       | Stressed Out           |
| 2016 | 6        | Justin Timberlake       | Can't Stop The Feeling |
| 2016 | 7        | Lukas Graham            | Seven Years            |
| 2016 | 8        | Rihanna & Drake         | Work                   |
| 2016 | 9        | Mike Posner             | I Took A Pill In Ibiza |
| 2016 | 10       | Justin Bieber           | Sorry                  |


## Number of rows

Next we would like to know how much data is present. Let's use this query:

SELECT count(*) FROM top_songs;

The output is: 

| count |
|-------|
| 300   |

- In this query, we are using a `function` called `count()`, which takes as input a table (or a column), and returns the number of rows in that table.
- SQL functions are in the same sense as in Python - they have an input argument with a certain type, and a return value.
- We can also give count a column name (`SELECT count(artist) FROM top_songs;`) and that will give us the same result - as the number of rows in a table with just `artist` is the same as the total number of rows.

### Null values

Let's run this query

```SQL
SELECT count(fr) FROM top_songs;
```

The output of this query is:

| count |
|-------|
| 235   |

- Up until now, we have seen no reason why this output should not be 300.
- The reason why the column `fr` has a lower count is because many values in that columns are `NULL`. `NULL` is a special value in a SQL table that denotes that nothing is present. -
- By running this query, we now know that the columns for each countries' billboard position have nulls.
- Note that we cannot supply two arguments to the `count()` function - `SELECT count(us, uk) FROM top_songs` doesn't work.

### Aliasing

Using the following query, we can understand the use case for `aliasing` of column names.

If we run this:

```SQL
SELECT count(id), count(uk) FROM top_songs;
```

we get:

| count_duplicate_column_name_1 | count |
|-------------------------------|-------|
| 300                           | 251   |

Note the column names in the output don't tells us which column's count it is. We can write the query instead as follows:

```SQL
SELECT count(id) as count_id, count(uk) count_uk FROM top_songs;
```

and we get:

| count_id | count_uk |
|----------|----------|
| 300      | 251      |

- The `AS` keyword is used to give an **alias** to any column in the output.
- It is very commonly used - the rule is not to depend on the column names which SQL query will return by itself.
- Production SQL queries typically have an alias for every column returned in the output.

# Distinct values

The next thing we want to know if the values in any column are all distint, or have some duplication in them. In our table, it's obvious that the `year` column has many repetitions, but we are not sure about the `artist` column.

## The distinct values

Let's check what all values are present in the `year` column.

```SQL
SELECT distinct(year) FROM top_songs
```

The output is:

| year |
|------|
| 2018 |
| 2017 |
| 2016 |

- `distinct()` is another function in the SQL. This function is different from `count()` in multiple ways:
  - The return value is not a single number, but a table containing many values.
  - The input can actually contain multiple column names, and it will return all possible combinations of those columns which are distinct (the value of many columns are considered the same when the values of all columns is identical). For example:
  
```SQL
SELECT distinct(year, id) FROM top_songs;
```

returns 300 rows, looking like:

| row        |
|------------|
| (2016,1)   |
| (2016,2)   |

- Note that even though we are finding distinct combinations of two columns, the output contains only one column.
- The values of all columns are packed into a tuple.

## Count of distinct

To check if there are duplicates in the `artist` column, we would rather count the number of distinct values in that column. To do this, we are going to compose the two SQL functions we've seen above:

```SQL
SELECT count(distinct(artist)) FROM top_songs;    
```

The output is:

| count |
|-------|
| 218   |

- This means that some artists appear multiple times in this list. They are not one song wonders!
- For the composition to work, we have to make sure that the output of `distinct()` function is of the same type as the required input type for the `count()` function. If that were not true, there would be an error.

# Filtering rows

The single most important transformation in analysis is filtering of rows. Let's say we want to find all the songs by one specific artist - let's find songs by Drake.

```SQL
SELECT year, position, song FROM top_songs WHERE artist = 'Drake';
```

The output is:

| year | position | song           |
|------|----------|----------------|
| 2016 | 36       | Controlla      |
| 2016 | 44       | Hotline Bling  |
| 2017 | 53       | Passionfruit   |
| 2017 | 56       | Fake Love      |
| 2018 | 2        | God's Plan     |
| 2018 | 11       | Nice For What  |
| 2018 | 12       | In My Feelings |
| 2018 | 43       | Nonstop        |

- The `WHERE` keyword is used to describe filters.
- Any filter is simply a boolean condition - if that condition is true, the row is included, otherwise its not.

Let's look at some examples of the conditions:

```SQL
SELECT artist, year, position, song FROM top_songs WHERE artist = 'Drake' OR artist = 'Ed Sheeran';
```

The output is:

| artist     | year | position | song               |
|------------|------|----------|--------------------|
| Drake      | 2016 | 36       | Controlla          |
| Drake      | 2016 | 44       | Hotline Bling      |
| Ed Sheeran | 2017 | 1        | Shape Of You       |
| Ed Sheeran | 2017 | 12       | Castle On The Hill |
| Ed Sheeran | 2017 | 52       | Perfect            |
| Drake      | 2017 | 53       | Passionfruit       |
| Drake      | 2017 | 56       | Fake Love          |
| Ed Sheeran | 2017 | 67       | Galway Girl        |
| Ed Sheeran | 2018 | 1        | Perfect            |
| Drake      | 2018 | 2        | God's Plan         |
| Drake      | 2018 | 11       | Nice For What      |
| Drake      | 2018 | 12       | In My Feelings     |
| Drake      | 2018 | 43       | Nonstop            |
| Ed Sheeran | 2018 | 49       | Shape Of You       |

As you can see, this `WHERE` condition gets us the songs from both Drake and Ed Sheeran, by using the OR operator.

```SQL
SELECT position, song FROM top_songs WHERE artist = 'Ed Sheeran' AND year = 2018;
```

| position | song         |
|----------|--------------|
| 1        | Perfect      |
| 49       | Shape Of You |

This one returns the top songs from Ed Sheeran only in 2018, using the AND operator.

Here is another query - let's say we want to know all the artist whose songs have been in the US top 2 in 2018.

```SQL
SELECT year, us, artist, song FROM top_songs WHERE us <= 2 AND year = 2018;
```

The output is:


| year | us | artist                             | song            |
|------|----|------------------------------------|-----------------|
| 2018 | 1  | Ed Sheeran                         | Perfect         |
| 2018 | 1  | Drake                              | God's Plan      |
| 2018 | 1  | Maroon 5 & Cardi B                 | Girls Like You  |
| 2018 | 1  | Cardi B, Bad Bunny & J Balvin      | I Like It       |
| 2018 | 2  | Bebe Rexha & Florida Georgia Line  | Meant To Be     |
| 2018 | 1  | Camila Cabello & Young Thug        | Havana          |
| 2018 | 1  | Post Malone & Ty Dolla $ign        | Psycho          |
| 2018 | 2  | Juice WRLD                         | Lucid Dreams    |
| 2018 | 1  | Drake                              | Nice For What   |
| 2018 | 1  | Drake                              | In My Feelings  |
| 2018 | 2  | Post Malone & 21 Savage            | Rockstar        |
| 2018 | 1  | XXXTentacion                       | Sad!            |
| 2018 | 1  | Travi$ Scott                       | Sicko Mode      |
| 2018 | 2  | Drake                              | Nonstop         |
| 2018 | 1  | Childish Gambino                   | This Is America |
| 2018 | 2  | Halsey                             | Without Me      |
| 2018 | 2  | Kodak Black, Travi$ Scott & Offset | Zeze            |
| 2018 | 1  | Ariana Grande                      | Thank U, Next   |



[This link on SQL WHERE clause](https://www.w3schools.com/SQl/sql_where.asp) lists all the conditions that you can use.

# GROUP BY

This is one of the most crucial clauses we are going to learn in SQL. Let's start with a problem statement: **For each artist, we want to know how many songs they have in this list.** The artists which have the more songs are more consistent in producing hits.

Before we write the query, let's understand what the data transformation is by itself.
- "For each artist" - this means we are dividing the data into _groups_, where each group contains all the songs by one artist. We will create as many groups as there are artists.
- "how many songs" - this means we want a count of songs in each group.

Here is the query:

```SQL
SELECT artist, count(song) FROM top_songs GROUP BY artist;
```

This returns 218 rows (remember that we found that there are 218 distinct artists before). Here are the first few rows.

| artist                                                         | count |
|----------------------------------------------------------------|-------|
| Ed Sheeran                                                     | 6     |
| Rae Sremmurd                                                   | 1     |
| David Guetta & Sia                                             | 1     |
| Bebe Rexha & Florida Georgia Line                              | 1     |
| Post Malone & Quavo                                            | 1     |
| Shawn Mendes                                                   | 5     |
| Rihanna                                                        | 2     |

Let's understand the query.

- The new keyword here is `GROUP BY` (two words actually). 
- To the `GROUP BY` clause you need to specify the columns which are used to form groups. For now, let's assume we are giving it only one column, `artist` in this case.
- Internally, the SQL execution engine will create groups which can be thought of as a mapping from `artist` value --> _List of rows with that value of artist. Here is a representation of what this mapping might look like:

Rihanna : [(2016,17,Rihanna,Needed Me,8629.708,7,38,94,25,44),(150,2017,50,Rihanna,Love On The Brain,4862.477,5,12,22,)]

Cardi B : [(128,2017,28,Cardi B,Bodak Yellow (Money Moves),6757.79,1,24,70,6,33),(276,2018,76,Cardi B,Be Careful,3481.318,11,24,154,24,)]

- The output MUST contain one row per group. That means that the structure of output is restricted. 


- The next thing is to look at the columns specified in the `SELECT` clause. There are two types of _expressions_ that can be present here.
- The first expression can be the column name. If you are straight up mentioning a column name, it has to be the column you **mentioned in `GROUP BY`**. For example, `SELECT position FROM top_songs GROUP BY artist` will not work. Why?
   - Remember that each row must correspond to a group. If there is a column titled `position` in that row, what value should it have for artists that have multiple songs in the list?
- The second expression must be an **aggregate function** on any of the columns **NOT mentioned in `GROUP BY`**.
  - An aggregate function is one which takes as input a table/column and returns a single value. Like the function `COUNT`.
 
Let's look at another example: Let's measure the average "score" in each year.

```SQL
SELECT year, AVG(score) FROM top_songs GROUP BY year;
```

| year | avg                   |
|------|-----------------------|
| 2018 | 5241.6766400000000000 |
| 2017 | 5703.9580800000000000 |
| 2016 | 5619.2507300000000000 |

# Sorting

In the first example of `GROUP BY`, we notice that the artists are listed in arbitrary order. We would ideally like to sort the list by the _best_ artists first.

```SQL
SELECT artist, count(song) AS hits FROM top_songs GROUP BY artist ORDER BY hits DESC; 
```
	
Here are the first few rows:



| artist                                                         | hits |
|----------------------------------------------------------------|------|
| Drake                                                          | 8    |
| Ed Sheeran                                                     | 6    |
| Ariana Grande                                                  | 6    |
| Shawn Mendes                                                   | 5    |
| Imagine Dragons                                                | 5    |
| Twenty One Pilots                                              | 4    |


- The keyword `ORDER BY` is used to sort the output of a query.
- This keyword takes the _value_ which it should use to sort by. In this case, we have created a column called `hits` in the output (using AS clause), and sorted by that.
- The keyword `DESC` specifies that we want to sort in the descending order.
