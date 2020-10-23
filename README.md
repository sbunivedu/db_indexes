# SQLite Indexes

This example demonstrates the use of indexes on the following database:
* Person(SSID, first_name, last_name, email, phone_no, city)

The data set is in [the script file](./script.txt) and the code examples can be run on https://sqliteonline.com/

* create an index on `phone_no`.
```sql
CREATE INDEX index_person_phone
ON Person(phone_no);
```

* use `EXPLAIN QUERY PLAN` to check whether indexes are used.
```sql
EXPLAIN QUERY PLAN
SELECT first_name,last_name,email
FROM Person
WHERE phone_no = '84697';
```

It seems the database engine does plan to use the index to speedup the query.
```
id          order       from        detail
----------  ----------  ----------  ----------------------------------------------------
3           0           0           SEARCH TABLE Person USING INDEX idx_person_phone(phone_no=?)
```

* create an index that allows only UNIQUE values.
```sql
CREATE UNIQUE INDEX index_person_email ON Person(email);

INSERT INTO Person(first_name, last_name, email, phone_no, city)
VALUES('Vinay', 'Jariwala', 'vinay@gmail.com', '8798878', 'Vapi');

INSERT INTO Person(first_name, last_name, email, phone_no, city)
VALUES('Vinay', 'patel', 'vinay@gmail.com', '965652', 'Surat');
```

The database throws the following "unique constraint violation" error because
we created a unique index on the email column.
```
Error: UNIQUE constraint failed: Person.email
```

* create an index on multiple attributes.
Without an index on the name attributes, the database has to scan the table to
find persons by name.
```sql
EXPLAIN QUERY PLAN
SELECT * FROM Person WHERE first_name = 'Vinay';
```
```
id          order       from        detail
----------  ----------  ----------  -----------------
2           0           0           SCAN TABLE Person
```

We can define an index on multiple attributes (composite index).
```sql
CREATE INDEX index_person_name ON Person(first_name, last_name);
```
The following queries will use this index.
```sql
EXPLAIN QUERY PLAN
SELECT * FROM Person WHERE first_name = 'Vinay';
```
For each index attribute there exists a B-tree.
```
id          order       from        detail
----------  ----------  ----------  ----------------------------------------------------
3           0           0           SEARCH TABLE Person USING INDEX index_person_name (first_name=?)
```

```SQLite
EXPLAIN QUERY PLAN
SELECT * FROM Person WHERE first_name = 'vinay' AND last_name = 'jariwala';
```

```
id          order       from        detail
----------  ----------  ----------  ----------------------------------------------------
3           0           0           SEARCH TABLE Person USING INDEX index_person_name (first_name=? AND last_name=?)

```
The following queries will NOT use the index.
```sql
EXPLAIN QUERY PLAN
SELECT * FROM Person WHERE first_name = 'vinay' OR last_name = 'jariwala';
```
```
id          order       from        detail
----------  ----------  ----------  -----------------
2           0           0           SCAN TABLE Person
```

```sql
EXPLAIN QUERY PLAN
SELECT * FROM Person WHERE last_name = 'jariwala';
```
```
id          order       from        detail
----------  ----------  ----------  -----------------
2           0           0           SCAN TABLE Person
```

To solve the problem, we can create two separate indexes:
```sql
CREATE INDEX index_person_first_name ON Person(first_name);
CREATE INDEX index_person_last_name ON Person(last_name);
```

```sql
EXPLAIN QUERY PLAN
SELECT * FROM Person WHERE first_name = 'vinay' OR last_name = 'jariwala';
```
```
id    parent       notused        detail
--    ------       -------        -------------------------------------------------
4     0            0              MULTI-INDEX OR
5     4            0              INDEX1
11    5            0              SEARCH TABLE Person USING INDEX index_person_first_name (first_name=?)
16    4            0              INDEX2
22    16           0              SEARCH TABLE Person USING INDEX index_person_last_name (last_name=?)
```


rowID | first_name | last_name | email
------|------------|-----------|------
1 | C | M | E
2 | A | M | F
3 | A | L | G
4 | B | M | O
5 | C | Q | P
6 | A | N | X


first_name | last_name | rowID
-----------|-----------|------
A | M | 2
A | N | 6
A | L | 3
B | M | 4
C | M | 1
C | Q | 5




References
* https://www.tutlane.com/tutorial/sqlite/sqlite-indexes
* https://dzone.com/articles/database-btree-indexing-in-sqlite
* https://www.sqlitetutorial.net/sqlite-index
