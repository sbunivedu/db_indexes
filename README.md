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

* create an index that allows only UNIQUE values.
```sql
CREATE UNIQUE INDEX index_person_email ON Person(email);

INSERT INTO Person(first_name, last_name, email, phone_no, city)
VALUES('Vinay', 'Jariwala', 'vinay@gmail.com', '8798878', 'Vapi');

INSERT INTO Person(first_name, last_name, email, phone_no, city)
VALUES('Vinay', 'patel', 'vinay@gmail.com', '965652', 'Surat');
```

* create an index on multiple attributes.
```sql
CREATE INDEX index_person_name ON Person(first_name, last_name);
```
The following queries will use this index.
```sql
EXPLAIN QUERY PLAN
SELECT * FROM Person WHERE first_name = 'Vinay';

EXPLAIN QUERY PLAN
SELECT * FROM Person WHERE first_name = 'vinay' AND last_name = 'jariwala';
```

The following queries will NOT use the index.
```sql
EXPLAIN QUERY PLAN
SELECT * FROM Person WHERE first_name = 'vinay' OR last_name = 'jariwala';

EXPLAIN QUERY PLAN
SELECT * FROM Person WHERE last_name = 'jariwala';
```

References
* https://www.tutlane.com/tutorial/sqlite/sqlite-indexes
