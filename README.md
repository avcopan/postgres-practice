# postgres-practice

## Running PostgreSQL directly

Switch user to postgres:
```
sudo -i -u postgres
```

Create a database called "mydb":
```
createdb mydb
```
(to delete it, you would call "dropdb mydb")

Run psql to interact with the database
```
psql mydb
```

Now you can do some different commands. See
[here](https://www.postgresql.org/docs/8.0/tutorial-select.html).

## SQL tutorial notes:

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
Propwer database design would have been instead to make our list of cities in
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

## Notes on connecting to Postgres databases in Python

Following along [here](https://pynative.com/python-postgresql-tutorial/).

## Notes on converting to SQL databases from file system databases

A regular file system is a *hierarchical database management system*, whereas a
table-based system like PostgreSQL is a *relational database management system*
(RDBMS).

Info on converting from HDBMS to RDBMS:

 - https://stackoverflow.com/questions/4048151/what-are-the-options-for-storing-hierarchical-data-in-a-relational-database



