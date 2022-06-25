---
layout: post
title:  "Import Excel data into MySQL"
slug: 'import-excel-data-into-mysql'
date: 2020-06-14 12:02:00 +0300
category: database
tags: [mysql, csv, excel, import data]
icon: table
---

Recently I had to import an Excel document into a MySQL database so I want to show you how I did this.


We will do this in 4 steps:

1. Open your file and click Save As. Choose to save it as a `.csv` (Comma Separated) file. 
If you are running Excel **on a Mac**, in order to maintain the correct formatting, you have to save the file as a **Windows Comma Separated (.csv)** or **CSV (Windows)**.

2. After you saved it as a `.csv` file log into your MySQL and create a brand new database. 

> Note! The `--local-infile` option is needed by some MySQL versions for the data loading we’ll do in the following steps.

```
$ mysql -u root -p --local-infile
mysql> create database books;
mysql> use books;
```

3. Next define the schema for our `books` table using the `CREATE TABLE` command. For more details, see the (MySQL documentation)[https://dev.mysql.com/doc/refman/8.0/en/creating-tables.html].

```
CREATE TABLE books (
    id INT NOT NULL PRIMARY KEY,
    title VARCHAR(40),
    author_id INT,
    created_at DATE
    price FLOAT
);
```

4. Now that your table is created, the data from your `csv` can be imported using the `LOAD DATA` command.

```
LOAD DATA LOCAL INFILE "/home/books.csv" INTO TABLE library_db.books
    FIELDS TERMINATED BY ','
    LINES TERMINATED BY '\n'
    IGNORE 1 LINES 
    (id, title, author_id, @created_date, price)
    set created_at = STR_TO_DATE(@created_date,'%m/%d/%Y');
```
