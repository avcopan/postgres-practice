# postgres-practice

Notes on using PostgreSQL for data storage.

In this repository, you will find:

 - A tutorial for getting up and running with PostgreSQL/psycopg2 from conda (see `from-conda-install.md`).
 - A tutorial for getting up and running with PostgreSQL/psycopg2 from a system installation (see `from-system-install.md`).
 - A tutorial for learning basic SQL (see `postgresql-tutorial.md`).

## Resources Databse vs. File System Tradeoffs

General tradeoffs:

 - https://stackoverflow.com/questions/38120895/database-vs-file-system-storage

Performance differences:

 - https://stackoverflow.com/questions/2147902/is-it-faster-to-access-data-from-files-or-a-database-server


## Notes on converting to SQL databases from file system databases

A regular file system is a *hierarchical database management system*, whereas a
table-based system like PostgreSQL is a *relational database management system*
(RDBMS).

Info on converting from HDBMS to RDBMS:

 - https://stackoverflow.com/questions/4048151/what-are-the-options-for-storing-hierarchical-data-in-a-relational-database
 - [This](https://www.slideshare.net/billkarwin/models-for-hierarchical-data) was the most helpful resource I found on that page. It goes through each of the options, with examples.

Top options, after reading:

 - PostgreSQL has the [`ltree` datatype](https://www.postgresql.org/docs/current/ltree.html), which is essentially equivalent to a filesystem path. We could do something that is, in some sense, very similar to autofile or mirrors it. This would be essentially the **path enumeration** approach, as I understand it.
