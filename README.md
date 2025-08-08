## Database:

### What Is Data?
Data is a collection of individual facts/raw  or statistics.
Data can come in the form of text, observations, figures, images, numbers, graphs, or symbols. For example, data might include individual prices, weights, addresses, ages, names, temperatures, dates, or distances.

- Data is unorganised and unrefined facts or Data is a raw and unorganized fact.
- Data is an individual unit that contains raw materials which do not carry any specific meaning.
- Data doesn’t depend on information.


_There are two main types of data:_

- **Quantitative data**: is provided in numerical form, like the weight, volume, or cost of an item.
- **Qualitative data**: is descriptive, but non-numerical, like the name, sex, or eye color of a person.



### What Is Information?
Information is processed, organised and structured data. It provides context for data and enables decision making.

- Information comprises processed, organised data presented in a meaningful context
- Information is a group of data that collectively carries a logical meaning.
- Information depends on data.



### Database:
A database is an organized collection of data that is stored and accessed electronically. It is the collection of tables, queries, reports, views and other objects.  It allows data to be efficiently inserted, managed, retrieved, and updated.




### Transaction:
A transaction is a sequence of one or more SQL operations that are executed as a single logical unit of work. A transaction **must be completed fully** or **not at all** — ensuring the `integrity` and `consistency` of the database.


📌 **ACID** is an acronym that defines the key properties that ensure reliable processing of database transactions. These principles guarantee that even in cases of system failures, power loss, or errors, the database remains correct and consistent.

_These properties ensure reliable transaction processing:_

| Property            | Description                                                              |
| ------------------- | ------------------------------------------------------------------------ |
| **A - Atomicity**   | All operations in a transaction succeed, or none do. No partial changes. |
| **C - Consistency** | The database remains in a valid state before and after the transaction.  |
| **I - Isolation**   | Concurrent transactions do not interfere with each other.                |
| **D - Durability**  | Once committed, the changes are permanent, even after a crash.           |



#### Atomicity: 
- **"All or nothing."**
- A transaction is treated as a single unit.
- Either all operations within the transaction are completed successfully, or none are applied.
- If any part fails, the database is rolled back to its previous state.
- All queries must succeed. If one fail all should rollback.
- **Example**: If money is withdrawn from one account but fails to deposit into another, the withdrawal should also be canceled.


#### Consistency: 
- **"Preserve database rules."**
- A transaction must take the database from one valid state to another.
- Data must remain consistent with rules, constraints, and relationships defined in the schema.
- **Example**: A student ID must always be unique — a transaction violating that constraint will fail.

- Consistency in Data
	- defined by the user
	- referential integrity (foreign keys)
	- atomicity
	- isolation
- Consistency in reads
	- if a transaction committed a change will a new transaction immediately see the change?
	- relational and NoSQL databases suffer from this
	- eventual consistency


#### Isolation:

- **"Transactions don’t interfere with each other."**
- Multiple transactions can run concurrently without impacting each other's execution.
- The result should be the same as if the transactions were run one after the other.
- **Example**: Two users booking the last seat on a flight at the same time — isolation ensures only one succeeds.

- Can my inflight transaction see changes made by other transactions?
- Read phenomena
	- Dirty reads
	- Non-repeatable reads
	- Phantom reads
	- (Lost updates)
- Isolation levels


#### Durability:

- **"Once committed, always saved."**
- After a transaction is committed, changes are permanent, even in case of system crash or power failure.
- Ensures data is safely stored to disk.
- Commited transactions must be persisted in a durable non-volatile storage. 



#### Summary of ACID Table:

| Property        | Meaning                          | Goal                          |
| --------------- | -------------------------------- | ----------------------------- |
| **Atomicity**   | All-or-nothing execution         | Avoid partial changes         |
| **Consistency** | Valid state before/after         | Enforce rules and constraints |
| **Isolation**   | Independent concurrent execution | Prevent transaction conflicts |
| **Durability**  | Permanence of committed changes  | Survive crashes and failures  |



### Database Management System (DBMS): 

Database management system is a software which is used to manage the database. A Database Management System (DBMS) is software that allows users to define, create, manage, and manipulate databases. It provides an interface between users/applications and the data stored in a structured format, ensuring data integrity, security, and efficient access.


A database management system (DBMS) is system software for creating and managing databases and software through which we can store data, access data and manage data. 

_Typically, a DBMS has the following **elements**:_
- Kernel code:
    - This code manages memory and storage for the DBMS.

- Repository of metadata:
    - This repository is usually called a data dictionary.

- Query language:
    - This language enables applications to access the data.




### Key Functions of a DBMS:

- **Data Definition** : `Create`, `alter`, and `drop` database schemas (using DDL: Data Definition Language).
- **Data Manipulation** : `Insert`, `update`, `delete`, and `SELECT` query data (using DML: Data Manipulation Language).
- **Data Security** : Control access to data (authentication, authorization).
- **Transaction Management** : Ensure data consistency and recovery through ACID properties.
- **Concurrency Control** : Allow multiple users to access data simultaneously without conflict.
- **Backup and Recovery** : Provide data restoration capabilities in case of failure.



### Data Definition Language (DDL):

Defines and modifies database structure (schema).

- Used to define database objects like `tables`, `indexes`, `views`, etc.
- Changes made using DDL affect the structure of the database.
- DDL statements include `CREATE Table`, `ALTER Table`, `DROP Table`, `RENAME`, `TRUNCATE`, `CREATE INDEX`.
- DDL statements are not transactional, meaning they cannot be rolled back.
- DDL does not use WHERE clause in its statement.
- DDL statements are usually executed by a database administrator.
- Example: 

```
CREATE TABLE employees (
  id INT PRIMARY KEY,
  name VARCHAR(100),
  department VARCHAR(50)
);

ALTER TABLE employees ADD salary DECIMAL(10, 2);

DROP TABLE old_employees;
```


### Data Manipulation Language (DML):

Used to access and manipulate the data within existing database objects.

- Used to manipulate data within the database.
- Changes made using DML affect the data stored in the database.
- DML statements include `SELECT`, `INSERT`, `UPDATE`, and `DELETE`, `MERGE`
- DML statements are transactional, meaning they can be rolled back if necessary.
- While DML uses WHERE clause in its statement.
- DML statements are executed by application developers or end-users.
- Example: 

```
INSERT INTO employees (id, name, department, salary) VALUES (1, 'Alice', 'HR', 55000.00);

UPDATE employees SET salary = 60000.00 WHERE id = 1;

DELETE FROM employees WHERE department = 'HR';

SELECT * FROM employees;
```



### SQL Commands Overview: 

SQL is divided into 5 types of sub-languages based on nature of commands.

- DDL: Object level
	- Used to create or change or delete any database objects.
	- `CREATE` – Create a new table, view, index, etc.
    - `ALTER` – Modify the structure of existing database objects.
    - `DROP` – Delete database objects.
    - `TRUNCATE` – Quickly remove all records from a table (no rollback).
    - `RENAME` – Rename a database object.
	
- DML: Data level
	- select (data query lanuage)
    - `INSERT` – Add new data.
    - `UPDATE` – Modify existing data.
    - `DELETE` – Remove data.
    - Note: DML operations can be rolled back or committed (transactional).

	
- DCL: Control level (These commands are used by DBA)
	- `GRANT` – Give privileges to users.
    - `REVOKE` – Take away privileges.
	
- TCL: Transaction level
	- `COMMIT` – Save changes made in a transaction.
    - `ROLLBACK` – Undo changes made in the current transaction.
    - `SAVEPOINT` – Set a point to rollback to within a transaction.

- DQL/DRL: Selection level (retrieval)
    - `SELECT` – Retrieve data (may include filtering, joins, grouping, etc.)






### Common DBMS Software:
- **Oracle DB** : Enterprise-grade RDBMS with powerful features.
- **MySQL** : Open-source RDBMS widely used in web applications.
- **PostgreSQL** : Advanced open-source RDBMS with strong standards compliance.
- **MongoDB** : Document-based NoSQL DB, great for semi-structured data.
- **SQLite** : Lightweight, serverless RDBMS used in embedded systems.



### License:
This project is licensed under the MIT License.

