(These instructions assume that you have root permissions.
See `from-conda-install.md` for instructions on how to do this in an environment with conda.)

## Installing PostgreSQL

Like this:
```
sudo apt-get install postgresql
```
Note that this requires sudo permissions.


## Setting Up Cluster and Server

One way to get started immediately is by doing the following:
```
sudo su postgres
```
This switches user to `postgres` with root permissions. Then you can start PostgreSQL without entering a password or setting up a username.
```
psql
```
This works to simply run PostgreSQL, but it will not work for interacting with your databases from Python.

I followed [these instructions](https://ubuntu.com/server/docs/databases-postgresql) in order to do this the right way, enabling interfacing with Python.

### Create a Username and Password

Switch user to postgres:
```
sudo su postgres
```
Now start the `psql` command line running:
```
psql
```
Inside the `psql` command line, run the following to create a new username/password combo:
```
ALTER USER postgres with encrypted password 'pass';
```

Now, quit out of `psql` and edit the following file, configuring postgres to use MD5 authentication with the postgres user:
```
vi /etc/postgresql/12/main/pg_hba.conf
local   all         postgres                          md5
```
Then restart the PostgreSQL service:
```
sudo systemctl restart postgresql.service
```
Then test that the new password worked:
```
psql -U postgres -W
```
From now on, you can start running with the following single command:
```
psql -U postgres
```

### Create a New Database

From the `psql` command line, you can create a new database as follows:
```
CREATE DATABASE db;
```
Now, we can connect to that database like this:
```
\c db
```
Note that "\c" stands for "\connect".

Now you can do some different commands. See
[here](https://www.postgresql.org/docs/8.0/tutorial-select.html).


### Create a Table in the New Database

The table creation command looks like this:
```
CREATE TABLE todo_list (
   task varchar(200),
   date date,
   priority int
);
```
To delete the first table, you would do "DROP TABLE todo_list;".

### Add Values to the Table

To add values into the table, we use the INSERT statement.
The simple form is:
```
INSERT INTO todo_list VALUES ('eat breakfast', '2021-09-06', 1);
INSERT INTO todo_list VALUES ('learn sql', '2021-09-06', 2);
INSERT INTO todo_list VALUES ('take a break', '2021-09-06', 3);
INSERT INTO todo_list VALUES ('sleep', '2021-09-07', 4);
```

In production code, it is considered better practice to explicitly list the columns (in which
case order no longer matters):
```
INSERT INTO todo_list (task, date, priority)
    VALUES ('wake up', '2021-09-07', 5);
```

### View the Table

To view the whole table, do this:
```
SELECT * FROM todo_list;
```
The asterisk character automatically grabs all columns.

To grab a selection of columns, do this:
```
SELECT task, priority FROM todo_list;
```
In production code, it is considered best practice to always list all columns instead of using `*`.

## Connecting to your DB from Python

Following along [here](https://www.postgresqltutorial.com/postgresql-python/).

### Install `psycopg2`

Type `exit` to get out of `psql` if you need to.

Then install `psycopg2` as follows:
```
conda install psycopg2
```

### Connect to your DB

Like so:
```
>>> import psycopg2
>>> conn = psycopg2.connect("dbname='db' port='12345' user='copan'")
>>> cur = conn.cursor()
>>> cur.execute("""SELECT * FROM todo_list""")
>>> cur.fetchall()
```
This should return the contents of your table as a list.

## Notes on converting to SQL databases from file system databases

A regular file system is a *hierarchical database management system*, whereas a
table-based system like PostgreSQL is a *relational database management system*
(RDBMS).

Info on converting from HDBMS to RDBMS:

 - https://stackoverflow.com/questions/4048151/what-are-the-options-for-storing-hierarchical-data-in-a-relational-database
 - [This](https://www.slideshare.net/billkarwin/models-for-hierarchical-data) was the most helpful resource I found on that page. It goes through each of the options, with examples.

Info on performance differences between file system and DBMS:

 - https://stackoverflow.com/questions/2147902/is-it-faster-to-access-data-from-files-or-a-database-server

Top options, after reading:

 - PostgreSQL has the [`ltree` datatype](https://www.postgresql.org/docs/current/ltree.html), which is essentially equivalent to a filesystem path. We could do something that is, in some sense, very similar to autofile or mirrors it. This would be essentially the **path enumeration** approach, as I understand it.

