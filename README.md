# postgres-practice

# # Running PostgreSQL directly:

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

# # Notes:

A regular filesystem is a *hierarchical database management system*, whereas a
table-based system like PostgreSQL is a *relational database management system*
(RDBMS).

Info on converting from HDBMS to RDBMS:

 - https://stackoverflow.com/questions/4048151/what-are-the-options-for-storing-hierarchical-data-in-a-relational-database



