(These instructions assume that you do not have root permissions.
See `from-system-install.md` for instructions on how to do install PostgreSQL as a system server with root permissions.)

## Installing PostgreSQL with conda

Like this:
```
conda install -y -c conda-forge postgresql
```

## Setting Up Cluster and Server

Instructions from [here](https://gist.github.com/gwangjinkim/f13bf596fefa7db7d31c22efd1627c7a),
as well as Argonne-specific instructions from [here](https://www.alcf.anl.gov/support-center/theta/postgresql-and-sqlite).
I ended up using a combination of the two to get it working.

### Create a New Database Cluster

First, create a database cluster (use your Argonne username):
```
initdb -D dbcluster -U $USER
```
This will be the basic database system that houses all of your databases.

Now, edit lines 59 and 63 of the file dbcluster/postgresql.conf as follows:
```
listen_addresses = '*'      # what IP address(es) to listen on;
                    # comma-separated list of addresses;
                    # defaults to 'localhost'; use '*' for all
                    # (change requires restart)
port = 12345                # (change requires restart)
```
You will also need to edit the file dbcluster/pg_hba.conf, appending the following line to the end:
```
host all all 0.0.0.0/0 trust
```
Make these changes before you start the cluster running (see below), or, if you've already started it, stop and restart the cluster.


### Start Cluster Server Running
```
pg_ctl -D ./dbcluster start
```
You should get a message that says `server started`.

If you ever needed to stop the cluster server, you would run the same command with `stop` instead of `start`.


## Doing Stuff

### Start Running Postgres

Here's what worked for me:
```
psql -p 12345 -U $USER -d postgres
```

### Create a New Database

You can create a new database as follows:
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

## Additional Questions/Concerns

### Running PostgreSQL on Blues

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

