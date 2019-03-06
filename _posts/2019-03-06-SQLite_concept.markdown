---
layout: post
title:  "SQLite Concepts and Core API"
date:   2019-03-06 23:37:01 +0800
categories: [SQLite]
tags: [SQLite]
---

<!-- TOC -->autoauto- [Principal Concepts](#principal-concepts)auto    - [Connection](#connection)auto    - [Statement](#statement)auto    - [B-tree](#b-tree)auto    - [Pager](#pager)auto- [Core API](#core-api)auto    - [Connecting to a Database](#connecting-to-a-database)auto    - [Executing Prepared Queries](#executing-prepared-queries)auto    - [Using Parameterized SQL](#using-parameterized-sql)auto    - [Executing Wrapped Queries](#executing-wrapped-queries)auto        - [sqlite3_exec()](#sqlite3_exec)auto        - [sqlite3_get_table()](#sqlite3_get_table)autoauto<!-- /TOC -->

## Principal Concepts

From a programmer’s point of view, the main things to know about are connections, statements, the B-tree, and the pager.

![ NULL](/assets/sqlite_api_object_model.png)

### Connection

A connection represents a single connection to a database as well as a single transaction context. In the C API, they correspond directly to the sqlite3

### Statement

Statements are derived from connections. That is, every statement has an associated connection object.
A statement represents a single compiled SQL statement. Internally, it is expressed in the form of VDBE
byte code—a program that when executed will carry out the SQL command. Statements contain
everything needed to execute a command. They include resources to hold the state of the VDBE program
as it is executed in a stepwise fashion, B-tree cursors that point to records on disk, and other things such
as bound parameters, which are addressed later in the section “Parameter Binding.” Although they
contain many different things, you can simply think of them as cursors with which to iterate through a
result set, or as opaque handles referencing a single SQL command.

### B-tree

Each database object has one B-tree object, which in turn has one pager object.

Statements that read the database iterate over B-trees using cursors. Cursors iterate over
records, and records are stored in pages. As a cursor traverses records, it also traverses pages. 

### Pager

For a cursor to access a page, it must first be loaded from disk into memory. This is the pager’s job. Whenever
the B-tree needs a particular page in the database, it asks the pager to fetch it from disk. The pager then
loads the page into its page cache, which is a memory buffer. Once it is in the page cache, the B-tree and
its associated cursor can get to the records inside the page.

The pager is responsible for reading and writing to and from the database, maintaining a memory cache or pages, and managing transactions. In
addition to this, it manages locks and crash recovery.


## Core API

### Connecting to a Database

The function used to connect, or open, a database in the C API is sqlite3_open() and is basically just a system call for opening a file.

### Executing Prepared Queries

the prepared query method is the actual process by which SQLite executes all SQL
commands. Executing a SQL command is a three-step process:

Preparation: The parser, tokenizer, and code generator prepare the SQL
statement by compiling it into VDBE byte code. In the C API, this is performed by
the sqlite3_prepare_v2() function, which talks directly to the compiler. The
compiler creates a sqlite3_stmt handle (statement handle) that contains the byte
code and all other resources needed to execute the command and iterate over the
result set (if the command produces one).

• Execution: The VDBE executes the byte code. Execution is a stepwise process. In
the C API, each step is initiated by sqlite3_step(), which causes the VDBE to step
through the byte code. The first call to sqlite3_step() usually acquires a lock of
some kind, which varies according to what the command does (reads or writes).
For SELECT statements, each call to sqlite3_step() positions the statement
handle’s cursor on the next row of the result set. For each row in the set, it returns
SQLITE_ROW until it reaches the end, whereupon it returns SQLITE_DONE. For other
SQL statements (insert, update, delete, and so on), the first call to sqlite3_step()
causes the VDBE to process the entire command.

• Finalization: The VDBE closes the statement and deallocates resources. In the C
API, this is performed by sqlite3_finalize(), which causes the VDBE to
terminate the program, freeing resources and closing the statement handle.

![ NULL](/assets/statement_processing.png)

The following pseudocode illustrates the general process of executing a query in SQLite:

```
# 1. Open the database, create a connection object (db)
db = open('foods.db')
# 2.A. Prepare a statement
stmt = db.prepare('select * from episodes')
# 2.B. Execute. Call step() is until cursor reaches end of result set.
while stmt.step() == SQLITE_ROW
print stmt.column('name')
end
# 2.C. Finalize. Release read lock.
stmt.finalize()
# 3. Insert a record
stmt = db.prepare('INSERT INTO foods VALUES (…)')
stmt.step()
stmt.finalize()
# 4. Close database connection.
db.close()
```

### Using Parameterized SQL

```
db = open('foods.db')
stmt = db.prepare('insert into episodes (id, name) values (:id, :name)')
stmt.bind('id', '1')
stmt.bind('name', 'Soup Nazi')
stmt.step()
# Reset and use again
stmt.reset()
stmt.bind('id', '2')
stmt.bind('name', 'The Junior Mint')
# Done
stmt.finalize()
db.close()
```

### Executing Wrapped Queries

#### sqlite3_exec()

```
db = open('foods.db')
db.exec("insert into episodes (id, name) values (1, 'Soup Nazi')")
db.exec("insert into episodes (id, name) values (2, 'The Fusilli Jerry')")
db.exec("begin; delete from episodes; rollback")
db.close()
```

#### sqlite3_get_table()

```
db = open('foods.db')
table = db.get_table("select * from episodes limit 10")
for i=0; i < table.rows; i++
    for j=0; j < table.cols; j++
        print table[i][j]
    end
end
db.close()
```

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
