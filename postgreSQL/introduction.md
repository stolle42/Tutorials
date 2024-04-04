# What is PostgreSQL?
Applications written in a common programming language like C, Python or Java do not store data permanantly. All variables are deleted as soon as the program is finished. This is a problem, since almost every app needs permanent data storage.

The first solution one could come up with is saving all data in files like csv. Most major programs provide libraries for exactely this purpose. But when data becomes larger and more complex, csv is way too inefficient, inflexible, hard to scale and confusing.

This is where databases like PostgreSQL come to the rescue. It is a very powerful  open-source relational database system following the **ACID-principle**.
# Prerequisites
None
# Tooling and preparation

# Architecture
A Postgres-Database consists of several tables. Every table consists of rows and columns. Their intersections form cells containing the actual data in a format dictated by the column.
# Hello World
