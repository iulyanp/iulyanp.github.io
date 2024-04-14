---
layout: post
title:  "Find the table size in MySQL"
description: 'Find the table size in MySQL'
slug: 'find-the-table-size-in-mysql'
date: 2020-06-12 09:46:00 +0300
category: database
tags: [mysql, tables, metadata, information schema]
icon: table
---

MySQL provides useful metadata about your database itself. While most other databases refer to this information as a `catalog`, the (official MySQL documentation)[https://dev.mysql.com/doc/refman/8.0/en/information-schema.html] refers to the `INFORMATION_SCHEMA` metadata as `tables`.

The important information provided by these `INFORMATION_SCHEMA` tables can help us determine how much disk space takes a specific table from our database.

## List Database All Tables Sizes

If we take a look in the documentation the `INFORMATION_SCHEMA.TABLES` table contains around 20 columns, but for the purpose of determining the amount of disk space used by tables, we’ll focus on two columns in particular: `DATA_LENGTH` and `INDEX_LENGTH`.

- `DATA_LENGTH` is the size in bytes of all data in the table.
- `INDEX_LENGTH` is the size in bytes of the index file for the table.

Having this information in mind, we can create a query that will list all tables in a specific database along with the disk space of each.

```
SELECT
  TABLE_NAME AS `Table`,
  ROUND((DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024) AS `Size (MB)`
FROM
  information_schema.TABLES
WHERE
  TABLE_SCHEMA = "library"
ORDER BY
  (DATA_LENGTH + INDEX_LENGTH)
DESC;
```

Because the size it is stored in bytes I convert it into something more useful and understandable: **megabytes** (MB).

In this example we are using the `library` database by combining the `DATA_LENGTH` and `INDEX_LENGTH` as bytes, then dividing it by `1024` twice to convert into `kilobytes` and then `megabytes`.

The result will look something like this:

+-------------------------------------+-----------+
| Table                               | Size (MB) |
+-------------------------------------+-----------+
| books                               |       546 |
| authors                             |       123 |
| clients                             |        56 |
...

## List Database Table Size

If you don’t care about all tables in the database and only want the size of a particular table, you can simply add `AND TABLE_NAME = "table_name"` to the `WHERE clause`. Here we only want information about the `authors` table:

```
SELECT
  TABLE_NAME AS `Table`,
  ROUND((DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024) AS `Size (MB)`
FROM
  information_schema.TABLES
WHERE
    TABLE_SCHEMA = "library"
  AND
    TABLE_NAME = "authors"
ORDER BY
  (DATA_LENGTH + INDEX_LENGTH)
DESC;
```

The result will be:

+----------+-----------+
| Table    | Size (MB) |
+----------+-----------+
| authors  |       123 |
+----------+-----------+
1 row in set (0.00 sec)


If you have multiple databases and you want to see which table from which database is the biggest in size, you will find it useful to query for the size of all tables within all databases from your system. 
This can be easily done with another similar query:

```
SELECT
  TABLE_SCHEMA AS `Database`,
  TABLE_NAME AS `Table`,
  ROUND((DATA_LENGTH + INDEX_LENGTH) / 1024 / 1024) AS `Size (MB)`
FROM
  information_schema.TABLES
ORDER BY
  (DATA_LENGTH + INDEX_LENGTH)
DESC;
```

This will return the database name in which the table is, the table name and the size of it.