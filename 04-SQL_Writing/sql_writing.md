# Introduction

This lecture will introduce SQL statements that are used to create and update tables and columns in a DB. After that, will look at some particular SQL features which are helpful but we haven't looked at them yet.

# Write statements

## CREATE TABLE

In Lecture 2 and 3, we've been working with some tables already present in the database. To create those tables, I've used SQL statements to create tables and insert data. Let's use one of those tables to understand how to create tables.

The following is the SQL statement for creating the table `top_songs` which we used in lecture 2, minus a few columns.

```SQL
CREATE TABLE top_songs_test(
   id       SERIAL PRIMARY KEY
  ,year     INTEGER  NOT NULL
  ,position INTEGER  NOT NULL
  ,artist   VARCHAR(62) NOT NULL
  ,song     VARCHAR(48) NOT NULL
  ,score    NUMERIC(9,3) NOT NULL
  ,us       INTEGER
);
```

- The syntax is clear from the statement. The SQL keywords used are `CREATE TABLE`. You supply the table name after that, and description of each column in the table in the parenthesis after that.

- Each column description contains it's **name**, it's **type**, and some **constraints** that we want to enforce on that column.
  - There are many types that SQL DBs supports, and many of them are specific to the type of SQL DB you are using (and even the version, as new types can be introduced over time). [Here is a document](https://www.w3resource.com/sql/data-type.php) that describes the types as defined in the SQL standard.
  - Constraints are used to make sure the value in a column satisfies some criteria. For example, in most of columns in `top_songs`, columns are `NOT NULL` which means they can't contain a `NULL` value. If you every try to insert a row in that table where `year` is NULL, SQL will give you an error.
  - [Here is the list of commonly used constraints](https://www.w3schools.com/sql/sql_constraints.asp). It's good to understand all of them, as most of them are used in real use cases.
  
  
- You can run `SELECT * FROM top_songs_test;` and you will find that it returns 0 rows.

## INSERT

To add data into the table, we need to use the `INSERT` statement. Here is an example statement to insert a row into `top_songs_test` table.

```SQL
 INSERT 
    INTO top_songs_test(year,position,artist,song,score,us) 
    VALUES (2016,1,'Sia','Cheap Thrills',14801.661,1);
 ```

- The `INSERT` statement has two parts, one in which we specify what table and columns we are inserting the data into, and the other in which we specify what values are we supposed to enter in those columns.
- Note that we have not included the `id` column in this statement. The `id` column is of type `SERIAL`, which means that PostgreSQL automatically adds the next highest integer when we insert a row.

If you run a select all query, this is what you get:

|id |year|position|artist|song         |score    |us |
|---|----|--------|------|-------------|---------|---|
|1  |2016|1       |Sia   |Cheap Thrills|14801.661|1  |


You can leave out columns in the INSERT statement which you leave as NULL in that row:

```SQL
 INSERT 
    INTO top_songs_test(year,position,artist,song,score) 
    VALUES (2016,2,'Drake, Wizkid & Kyla','One Dance',14641.871);
```

Now your result for select all query is:

|id |year|position|artist              |song         |score    |us |
|---|----|--------|--------------------|-------------|---------|---|
|1  |2016|1       |Sia                 |Cheap Thrills|14801.661|1  |
|2  |2016|2       |Drake, Wizkid & Kyla|One Dance    |14641.871|   |

As you can see, the `us` column is empty in row 2.

If you leave out a column with `NOT NULL` constraint, that will give you an error. The following statement:


```SQL
INSERT 
	INTO top_songs_test(position,artist,song,score,us) 
	VALUES (4,'The Chainsmokers & Daya','Don''t Let Me Down',12936.962,3);
```

gives the following error:

```
ERROR:  null value in column "year" violates not-null constraint
```

## UPDATE

After insertion, the next most common _write_ statement that's used is the `UPDATE` statement. This query modifies the content of one or more existing row.

Let's start with a query to modify one row.

```SQL
UPDATE top_songs_test
    SET us = 3, score = 14700
    WHERE id = 2;
```

Running a select all query now gives us this:

|id |year|position|artist              |song         |score    |us |
|---|----|--------|--------------------|-------------|---------|---|
|1  |2016|1       |Sia                 |Cheap Thrills|14801.661|1  |
|2  |2016|2       |Drake, Wizkid & Kyla|One Dance    |14700.000|3  |

- In the syntax, the new values are provided to the `SET` clause in the `UPDATE` statement. You can specify as many columns as you want.
- The constraint checks apply every time you make any modification to the table. For example, you cannot set column `id` to 1, as that would violate the `PRIMARY KEY` constraint.

Here is a statement that changes all rows in the table:

```SQL
UPDATE top_songs_test
    SET year=2017;
```

The table contents after this statement look like this:

|id |year|position|artist              |song         |score    |us |
|---|----|--------|--------------------|-------------|---------|---|
|1  |2017|1       |Sia                 |Cheap Thrills|14801.661|1  |
|2  |2017|2       |Drake, Wizkid & Kyla|One Dance    |14700.000|3  |


## DELETE

SQL allows you to remove rows from an existing table. This use case is uncommon in data analysis, unless we are correcting a mistake that was made earlier.

The DELETE statement is very simple. It just needs a condition by which to filter rows, and it will remove all of those rows.

```SQL
DELETE FROM top_songs_test
    WHERE id = 2;
```

As expected, the table now contains just one row:

|id |year|position|artist|song         |score    |us |
|---|----|--------|------|-------------|---------|---|
|1  |2016|1       |Sia   |Cheap Thrills|14801.661|1  |

## ALTER TABLE

This is a query which can be used to add new columns or delete existing columns from a table. I leave this as an exercise:

**Exercise:** Add a column called `genre`, which has the `NOT NULL` constraint, and the default value is `Unknown`.

## DROP TABLE

As the name suggests, it will delete the table complete from the DB. The statement

```SQL
DROP TABLE top_songs_test;
```

deletes the table that we created at the beginning of this lecture.



## SELECT INTO

All of the statements that we've learned in this lecture are something data analysts don't use frequently. The raw data is usually already available in the database, and analysts work to answer questions using that data.

But there is one statement that creates a new table, and I believe is very useful for data analysts. 
- It creates a new table and stores the results of any `SELECT` query in that table. 
- This is useful because many times there is a data transformation that is the basis of many further queries, and one wants to store the result of that transformation for future use. 
- It is especially helpful if that data transformation takes a lot of time to execute (e.g., its a join).

To see how it works, consider the following query. In this query, we are using the original `top_songs` table that we used in lecture 2.

```SQL
SELECT year, position, artist, song 
    INTO top_billboard_songs 
    FROM top_songs 
    WHERE 1 IN (us, uk, de, fr, ca, au);
```

- The `SELECT` query finds all rows where the song has been in #1 position in any of the 6 countries for which we have data, in any year. We select four columns from those rows.
- A new table called `top_billboard_songs` is created which contains the results of the `SELECT` query. 


`SELECT * from top_billboard_songs LIMIT 5;` gives us the following result:

|year|position|artist              |song                  |
|----|--------|--------------------|----------------------|
|2016|1       |Sia                 |Cheap Thrills         |
|2016|2       |Drake, Wizkid & Kyla|One Dance             |
|2016|3       |Justin Bieber       |Love Yourself         |
|2016|6       |Justin Timberlake   |Can't Stop The Feeling|
|2016|7       |Lukas Graham        |Seven Years           |

and, `SELECT count(*) FROM top_billboard_songs;` gives us:

| count |
|-------|
| 76    |

# More SQL

Here is some more useful stuff that is useful to know about.

## LIKE operator

In lecture 2, we talked about operators in the WHERE condition of a query. One of those operators, called `LIKE` is very useful, and can be used in multiple ways.

Here is an example query:

```SQL
SELECT * FROM top_songs WHERE song LIKE '%Feeling%';
```

This returns:

|id  |year|position            |artist                |song                  |score    |us |uk |de |fr |ca |au |
|----|----|--------------------|----------------------|----------------------|---------|---|---|---|---|---|---|
|6   |2016|6                   |Justin Timberlake     |Can't Stop The Feeling|11988.748|1  |2  |1  |1  |1  |3  |
|144 |2017|44                  |Justin Timberlake     |Can't Stop The Feeling|5126.575 |13 |21 |   |23 |20 |9  |
|212 |2018|12                  |Drake                 |In My Feelings        |8707.825 |1  |1  |4  |5  |1  |1  |

Basically, the songs with the word `Feeling` in its title. I'm surprised there are only 3.

- As you can already guess, `LIKE` operator allows you to match a pattern.
- The two characters `%` and `_` are used to create the pattern.
  - `%` in the pattern matches zero, one or more characters.
  - `_` in the pattern matches exactly one character.
- The LIKE operator pattern is case _sensitive_. In the query above, if you replaced `%Feeling%` with `%feeling%`, you get no results.
  - To get around this, use the `lower()` function. For example: `SELECT * FROM top_songs WHERE lower(song) LIKE '%feeling%';` gives the same output.

**Exercise:** Try to find all songs in which **Drake** is the main artist, or in collaboration with other artists.
  
## CASE statement

The `CASE` statement is the equivalent of `if/then/else` statements in programming languages. In the `SELECT` clause, you can use it to add conditional logic. Here is an example:

```SQL
SELECT year, 
    position,
    CASE WHEN artist = 'Justin Bieber' THEN 'Just a beaver' ELSE artist END,
    song
FROM top_songs
WHERE us=1 AND year=2016;
```

This gives us:

|year|position|artist                   |song                  |
|----|--------|-------------------------|----------------------|
|2016|1       |Sia                      |Cheap Thrills         |
|2016|2       |Drake, Wizkid & Kyla     |One Dance             |
|2016|3       |Just a beaver            |Love Yourself         |
|2016|6       |Justin Timberlake        |Can't Stop The Feeling|
|2016|8       |Rihanna & Drake          |Work                  |
|2016|10      |Just a beaver            |Sorry                 |
|2016|12      |Desiigner                |Panda                 |
|2016|14      |The Chainsmokers & Halsey|Closer                |
|2016|24      |Adele                    |Hello                 |
|2016|26      |Zayn                     |Pillow talk           |
|2016|59      |Rae Sremmurd & Gucci Mane|Black Beatles         |

- As you can see, only if the artist was Justin Bieber, the name of the artist has changed.
- The syntax is simple to understand, though it introduces new keywords. Don't forget to end the `CASE` statement with `END`.

Like in the if statements in programming languages, you can add multiple conditions in one CASE statement.

```SQL
SELECT year, 
    position,
    CASE WHEN artist = 'Justin Bieber' THEN 'Just a beaver'
         WHEN artist LIKE '%Drake%' THEN 'Francis'
         WHEN artist = 'Adele' THEN 'Adele Laurie Blue Adkins' 
         ELSE artist END
    song
FROM top_songs
WHERE us=1 AND year=2016;
```

This is the result:


|year|position|song                     |
|----|--------|-------------------------|
|2016|1       |Sia                      |
|2016|2       |Francis                  |
|2016|3       |Just a beaver            |
|2016|6       |Justin Timberlake        |
|2016|8       |Francis                  |
|2016|10      |Just a beaver            |
|2016|12      |Desiigner                |
|2016|14      |The Chainsmokers & Halsey|
|2016|24      |Adele Laurie Blue Adkins |
|2016|26      |Zayn                     |
|2016|59      |Rae Sremmurd & Gucci Mane|





