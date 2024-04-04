# What is PostgreSQL?
Applications written in a common programming language like C, Python or Java do not store data permanantly. All variables are deleted as soon as the program is finished. This is a problem, since almost every app needs permanent data storage.

The first solution one could come up with is saving all data in files like csv. Most major programs provide libraries for exactely this purpose. But when data becomes larger and more complex, csv is way too inefficient, inflexible, hard to scale and confusing.

This is where databases like PostgreSQL come to the rescue. It is a very powerful  open-source relational database system following the **ACID-principle**.
# Prerequisites
None
# Tooling and preparation
Install postgreSQL
# Architecture
A Postgres database system consists of several databases, which each consist of several tables. Every table consists of rows and columns. Their intersections form cells containing the actual data in a format dictated by the column.

Process-wise, postgreSQL uses a client-server model. The server process (called postgres) receives commands from the client process and manages the database accordingly. The client process is the process sending the requests. It could be any application specifically designed for this purpose (e.g. pgadmin) or integrated in some other application.
# Hello World
## Create a database
During the instalation of postgres, it automatically created a new user called `postgres`. Therefore, we can easily enter the postgres database system like this:
```bash
sudo -u postgres psql
```
Now the command line should show `postgres=#`. In order to list all databases in the system, run
```SQL
\l 
```
This should show the databases postgres, template0 and template1. Those are created by default. But we won't use them, but instead create our own database. This is done by running
```SQL
CREATE DATABASE hellopostgres;[^1]
```
[^1]: SQL-commands are usually written in uppercase to distinguish them, but this is not mandatory.

If you list your databases again it should show a fourth database called hellopostgres.

Now let's connect to the newly created database like this:
```sql
\c hellopostgres
```
Now your terminal should show `hellopostgres=#`.
## Create a table
As mentioned, every database consists of tables. Let's create one! It is done by the `CREATE TABLE` command. This is a more complex command, because creating a table requires us to specify name and type of every column. Let's look at an example:
```sql
CREATE TABLE dogs(
    name VARCHAR(50),
    gender BOOL,
    age INT);
```
This command created a table of dogs with 3 columns:
- name (a string with a max length of 50)
- gender (a bool that can be true or false)
- and age (an integer)

Run this command and verify it worked by running `\d dogs` (This should give you information about all the columns of the table).
## Write and read data
Now that we have a table, we can finally write data to it. Let's create a new dog called "Pluto", who is male and 13 years old:
```sql
INSERT INTO dogs VALUES ('Pluto',true,13);
```
You can verify this worked by running
```sql
SELECT * FROM dogs;
```
Does it show the data you entered correctly? If yes, great! You have sucessfully written data to a database and read it out again.