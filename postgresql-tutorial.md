## PostgreSQL tutorial

Mostly following along with the main tutorial [here](https://www.postgresql.org/docs/8.0/tutorial-select.html).

See instructions elsewhere for getting up and running to the point of having a `psql` command line.

### Create a New Database

If you haven't already, create a new database and connect to it.

You can create a new database as follows:
```
CREATE DATABASE db;
```
Now, we can connect to that database like this:
```
\c db
```
Note that "\c" stands for "\connect".

### Creating tables

The table creation command looks like this:
```
CREATE TABLE weather (
    city            varchar(80),
    temp_lo         int,           -- low temperature
    temp_hi         int,           -- high temperature
    prcp            real,          -- precipitation
    date            date
);
```
Here's another:
```
CREATE TABLE cities (
    name            varchar(80),
    location        point
);
```
To delete the first table, you would do "DROP TABLE weather;".

To add values into the table, we use the INSERT statement.
The simple form is:
```
INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');
```
But it is considered better practice to explicitly list the columns (in which
case order no longer matters):
```
INSERT INTO weather (city, temp_lo, temp_hi, prcp, date)
    VALUES ('San Francisco', 43, 57, 0.0, '1994-11-29');
```

### Querying tables

To simply view the table, do this:
```
SELECT * FROM weather;
```
The asterisk character automatically grabs all columns and is equivalent to the
following:
```
SELECT city, temp_lo, temp_hi, prcp, date FROM weather;
```
In production code, it is considered best practice to always list all columns.

You can easily create expressions using data from the table:
```
SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;
```

### Simple joins

```
SELECT city, temp_lo, temp_hi, prcp, date, location
    FROM weather, cities
    WHERE city = name;
```
This is equivalent to an inner join as shown next.

### Inner joins

Inner joins zip two tables together on matching column entries.
For example:
```
SELECT * FROM weather INNER JOIN cities ON (weather.city = cities.name);
```
where the `weather` table has city names in the column `city` and the `cities`
table has them in the column `name.
The result is:
```
     city      | temp_lo | temp_hi | prcp |    date    |     name      | location  
---------------+---------+---------+------+------------+---------------+-----------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27 | San Francisco | (-194,53)
 San Francisco |      43 |      57 |    0 | 1994-11-29 | San Francisco | (-194,53)
(2 rows)
```
The result contains all columns in the intersection of `weather.city` and `cities.name`.

### Outer joins

Outer joins are similar to inner joins, but the result contains the *union* of
the columns rather than their intersection.
Missing data from unmatched columns in one or the other table can be
automatically filled in.
So,
```
# SELECT * FROM weather FULL OUTER JOIN cities ON (weather.city = cities.name);
```
will give
```
     city      | temp_lo | temp_hi | prcp |    date    |     name      | location  
---------------+---------+---------+------+------------+---------------+-----------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27 | San Francisco | (-194,53)
 San Francisco |      43 |      57 |    0 | 1994-11-29 | San Francisco | (-194,53)
 Hayward       |      37 |      54 |      | 1994-11-29 |               | 
               |         |         |      |            | New York      | (666,666)
(4 rows)
```
where we note the blank spaces for Hayward and New York.

A left outer join will include all rows from the left table, whereas a right
outer join will include all rows from the right table.
So,
```
# SELECT * FROM weather LEFT OUTER JOIN cities ON (weather.city = cities.name);
```
results in
```
     city      | temp_lo | temp_hi | prcp |    date    |     name      | location  
---------------+---------+---------+------+------------+---------------+-----------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27 | San Francisco | (-194,53)
 San Francisco |      43 |      57 |    0 | 1994-11-29 | San Francisco | (-194,53)
 Hayward       |      37 |      54 |      | 1994-11-29 |               | 
(3 rows)
```
where we note the extra blank entries for Hayward, which had no match in the
cities table.

### Views

Any query can be saved as a "view" without changing the underlying data.
According to the tutorial, "Making liberal use of views is a key aspect of good
SQL database design. Views allow you to encapsulate the details of the
structure of your tables, which may change as your application evolves, behind
consistent interfaces."

Views can be treated exactly like tables, but they do not store any new data --
they are simply a "view" on the table data through a particular operation.
Example:
```
# CREATE VIEW myview AS
     SELECT * FROM weather FULL OUTER JOIN cities ON (weather.city = cities.name);
```
Result:
```
     city      | temp_lo | temp_hi | prcp |    date    |     name      | location  
---------------+---------+---------+------+------------+---------------+-----------
 San Francisco |     -54 |      48 | 0.25 | 1994-11-27 | San Francisco | (-194,53)
 San Francisco |     -57 |      55 |    0 | 1994-11-29 | San Francisco | (-194,53)
 Hayward       |     -63 |      52 |      | 1994-11-29 |               | 
               |         |         |      |            | New York      | (666,666)
(4 rows)
```

### Primary keys and foreign keys

In the previous examples we ended up with blank rows upon joining tables
because we allowed the weather table to have data for locations that weren't
included in our list of cities.
Proper database design would have been instead to make our list of cities in
the cities table a "primary key" and the list of cities in the weather table a
"foreign key" referring to that first primary key.
This will require data added to the weather table to correspond to one of the
cities from our cities table.
```
CREATE TABLE cities (
        city     varchar(80) primary key,
        location point
);

CREATE TABLE weather (
        city      varchar(80) references cities(city),
        temp_lo   int,
        temp_hi   int,
        prcp      real,
        date      date
);
```
Now, if we were to try adding a city to the weather table without first adding
it to the list of cities, we would get an error:
```
# INSERT INTO weather VALUES ('Berkeley', 45, 53, 0.0, '1994-11-28');
```
```
ERROR:  insert or update on table "weather" violates foreign key constraint "weather_city_fkey"
DETAIL:  Key (city)=(Berkeley) is not present in table "cities".
```
Once we add "Berkeley" to our list of cities in the cities table, though, this
will work again.
