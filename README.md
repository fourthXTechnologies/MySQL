# MySQL Commands Reference

**From Scratch to Advanced — Essential Commands for Developers**

A practical, example-driven reference covering the most useful MySQL commands on a server environment.

---

## Table of Contents

1. [Getting Started & Connection](#1-getting-started--connection)
2. [Database Management](#2-database-management)
3. [Table Management](#3-table-management)
4. [Data Types Quick Reference](#4-data-types-quick-reference)
5. [Inserting Data](#5-inserting-data)
6. [Querying Data (SELECT)](#6-querying-data-select)
7. [Updating & Deleting Data](#7-updating--deleting-data)
8. [Joins](#8-joins)
9. [Indexes](#9-indexes)
10. [Constraints](#10-constraints)
11. [Views](#11-views)
12. [Stored Procedures & Functions](#12-stored-procedures--functions)
13. [Triggers](#13-triggers)
14. [Transactions](#14-transactions)
15. [Users, Privileges & Security](#15-users-privileges--security)
16. [Import / Export & Backup](#16-import--export--backup)
17. [Performance & Diagnostics](#17-performance--diagnostics)
18. [Useful System & Status Commands](#18-useful-system--status-commands)
19. [Advanced Tips & Best Practices](#19-advanced-tips--best-practices)

---

## 1. Getting Started & Connection

### Login to MySQL

```bash
mysql -u root -p
```
Prompts for password and opens the MySQL shell as root.

```bash
mysql -u username -p -h hostname -P 3306 database_name
```
Connect to a remote server or specific database.

### Exit MySQL

```sql
EXIT;
-- or
QUIT;
-- or
\q
```

### Show current user & version

```sql
SELECT USER();
SELECT VERSION();
STATUS;          -- shows version, connection info, uptime, etc.
```

### Clear screen (in terminal)

```sql
SYSTEM clear;    -- Linux/macOS
SYSTEM cls;      -- Windows
```

---

## 2. Database Management

### Show all databases

```sql
SHOW DATABASES;
```

### Create a database

```sql
CREATE DATABASE company_db;
CREATE DATABASE company_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### Check database character set / collation (and related info)
```sql
SHOW CREATE DATABASE your_database_name;

-- more detailed
SELECT 
    SCHEMA_NAME AS `Database`,
    DEFAULT_CHARACTER_SET_NAME AS `Charset`,
    DEFAULT_COLLATION_NAME AS `Collation`
FROM information_schema.SCHEMATA
WHERE SCHEMA_NAME = 'your_database_name';

-- for all the dbs
SELECT 
    SCHEMA_NAME AS `Database`,
    DEFAULT_CHARACTER_SET_NAME AS `Charset`,
    DEFAULT_COLLATION_NAME AS `Collation`
FROM information_schema.SCHEMATA
WHERE SCHEMA_NAME NOT IN ('mysql', 'information_schema', 'performance_schema', 'sys');
```

### Use / select a database

```sql
USE company_db;
```

### Show currently selected database

```sql
SELECT DATABASE();
```

### Drop a database

```sql
DROP DATABASE company_db;
DROP DATABASE IF EXISTS company_db;   -- safer
```

### Rename a database (MySQL 8.0+ alternative approach)

There is no direct `RENAME DATABASE`. Common workaround:

```sql
CREATE DATABASE new_db;
RENAME TABLE old_db.table1 TO new_db.table1;
-- repeat for all tables, then DROP old_db
```

---

## 3. Table Management

### Show tables in current database

```sql
SHOW TABLES;
SHOW FULL TABLES;                    -- includes views
SHOW TABLES LIKE 'user%';
```

### Create a table

```sql
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    salary DECIMAL(10,2),
    hire_date DATE,
    department_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Describe / show structure of a table

```sql
DESCRIBE employees;
DESC employees;
SHOW COLUMNS FROM employees;
SHOW CREATE TABLE employees;         -- full CREATE statement
```

### Alter a table

```sql
-- Add a column
ALTER TABLE employees ADD COLUMN phone VARCHAR(20) AFTER email;

-- Modify a column
ALTER TABLE employees MODIFY COLUMN salary DECIMAL(12,2);

-- Change column name + type
ALTER TABLE employees CHANGE COLUMN phone mobile VARCHAR(15);

-- Drop a column
ALTER TABLE employees DROP COLUMN mobile;

-- Add/Drop primary key
ALTER TABLE employees ADD PRIMARY KEY (id);
ALTER TABLE employees DROP PRIMARY KEY;

-- Rename table
ALTER TABLE employees RENAME TO staff;
RENAME TABLE employees TO staff;
```

### Drop a table

```sql
DROP TABLE employees;
DROP TABLE IF EXISTS employees;
```

### Truncate a table (remove all rows, keep structure)

```sql
TRUNCATE TABLE employees;
```

---

## 4. Data Types Quick Reference

| Category       | Common Types                          | Notes                              |
|----------------|---------------------------------------|------------------------------------|
| Integer        | `TINYINT`, `SMALLINT`, `INT`, `BIGINT`| Use `UNSIGNED` when possible       |
| Decimal        | `DECIMAL(p,s)`, `FLOAT`, `DOUBLE`     | Prefer `DECIMAL` for money         |
| String         | `CHAR(n)`, `VARCHAR(n)`, `TEXT`       | `VARCHAR` is most common           |
| Date/Time      | `DATE`, `TIME`, `DATETIME`, `TIMESTAMP`| `TIMESTAMP` auto-updates easily   |
| Binary         | `BLOB`, `BINARY`, `VARBINARY`         | For files / binary data            |
| JSON           | `JSON`                                | Native JSON support (MySQL 5.7+)   |
| Boolean        | `BOOLEAN` / `TINYINT(1)`              | Actually stored as 0/1             |

```sql
-- Example with various types
CREATE TABLE products (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    metadata JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 5. Inserting Data

### Insert single row

```sql
INSERT INTO employees (first_name, last_name, email, salary, hire_date)
VALUES ('John', 'Doe', 'john.doe@example.com', 75000.00, '2023-01-15');
```

### Insert multiple rows

```sql
INSERT INTO employees (first_name, last_name, email, salary)
VALUES 
    ('Jane', 'Smith', 'jane@example.com', 82000),
    ('Bob', 'Johnson', 'bob@example.com', 68000),
    ('Alice', 'Williams', 'alice@example.com', 91000);
```

### Insert from another table / SELECT

```sql
INSERT INTO employees_archive (id, first_name, last_name, email)
SELECT id, first_name, last_name, email FROM employees WHERE hire_date < '2020-01-01';
```

### Insert or update (ON DUPLICATE KEY)

```sql
INSERT INTO employees (id, first_name, last_name, email)
VALUES (1, 'John', 'Doe', 'john.updated@example.com')
ON DUPLICATE KEY UPDATE 
    email = VALUES(email),
    last_name = VALUES(last_name);
```

### Insert Ignore (skip duplicates)

```sql
INSERT IGNORE INTO employees (email, first_name, last_name)
VALUES ('john.doe@example.com', 'John', 'Doe');
```

---

## 6. Querying Data (SELECT)

### Basic SELECT

```sql
SELECT * FROM employees;
SELECT first_name, last_name, salary FROM employees;
```

### Filtering with WHERE

```sql
SELECT * FROM employees WHERE salary > 70000;
SELECT * FROM employees WHERE department_id = 3 AND salary >= 80000;
SELECT * FROM employees WHERE last_name LIKE 'S%';
SELECT * FROM employees WHERE hire_date BETWEEN '2022-01-01' AND '2023-12-31';
SELECT * FROM employees WHERE department_id IN (1, 2, 5);
SELECT * FROM employees WHERE email IS NOT NULL;
```

### Sorting & Limiting

```sql
SELECT * FROM employees ORDER BY salary DESC;
SELECT * FROM employees ORDER BY last_name ASC, first_name ASC;
SELECT * FROM employees LIMIT 10;
SELECT * FROM employees LIMIT 10 OFFSET 20;   -- pagination
-- or
SELECT * FROM employees LIMIT 20, 10;         -- offset, count
```

### Aggregations

```sql
SELECT COUNT(*) FROM employees;
SELECT COUNT(DISTINCT department_id) FROM employees;
SELECT AVG(salary), MAX(salary), MIN(salary), SUM(salary) FROM employees;
SELECT department_id, AVG(salary) AS avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 75000;
```

### Distinct & Aliases

```sql
SELECT DISTINCT department_id FROM employees;
SELECT first_name AS fname, last_name AS lname FROM employees;
```

### Subqueries

```sql
SELECT * FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

SELECT first_name, last_name
FROM employees
WHERE department_id IN (
    SELECT id FROM departments WHERE name = 'Engineering'
);
```

### CASE expressions

```sql
SELECT first_name, salary,
    CASE
        WHEN salary >= 90000 THEN 'High'
        WHEN salary >= 70000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_band
FROM employees;
```

---

## 7. Updating & Deleting Data

### Update rows

```sql
UPDATE employees
SET salary = 78000
WHERE id = 5;

UPDATE employees
SET salary = salary * 1.10
WHERE department_id = 2;

UPDATE employees
SET first_name = 'Jonathan', email = 'jonathan@example.com'
WHERE id = 1;
```

### Delete rows

```sql
DELETE FROM employees WHERE id = 10;
DELETE FROM employees WHERE hire_date < '2018-01-01';
DELETE FROM employees;                 -- deletes all rows (dangerous!)
```

> **Tip**: Always run a `SELECT` with the same `WHERE` clause first to verify which rows will be affected.

---

## 8. Joins

### INNER JOIN

```sql
SELECT e.first_name, e.last_name, d.name AS department
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

### LEFT JOIN (LEFT OUTER JOIN)

```sql
SELECT e.first_name, d.name AS department
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

### RIGHT JOIN

```sql
SELECT e.first_name, d.name AS department
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

### FULL OUTER JOIN (emulated in MySQL)

```sql
SELECT * FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
UNION
SELECT * FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

### Cross Join / Self Join

```sql
-- Cartesian product
SELECT * FROM employees CROSS JOIN departments;

-- Self join example (find employees in same department)
SELECT a.first_name AS emp1, b.first_name AS emp2, a.department_id
FROM employees a
JOIN employees b ON a.department_id = b.department_id AND a.id < b.id;
```

### Multiple joins

```sql
SELECT e.first_name, d.name AS dept, p.name AS project
FROM employees e
JOIN departments d ON e.department_id = d.id
JOIN employee_projects ep ON e.id = ep.employee_id
JOIN projects p ON ep.project_id = p.id;
```

---

## 9. Indexes

### Create indexes

```sql
-- Single column
CREATE INDEX idx_lastname ON employees(last_name);

-- Composite index
CREATE INDEX idx_dept_salary ON employees(department_id, salary);

-- Unique index
CREATE UNIQUE INDEX idx_email ON employees(email);

-- Primary key is automatically indexed
```

### Show indexes

```sql
SHOW INDEX FROM employees;
SHOW INDEXES FROM employees;
```

### Drop index

```sql
DROP INDEX idx_lastname ON employees;
ALTER TABLE employees DROP INDEX idx_lastname;
```

### Analyze index usage

```sql
EXPLAIN SELECT * FROM employees WHERE last_name = 'Smith';
EXPLAIN ANALYZE SELECT * FROM employees WHERE department_id = 3;  -- MySQL 8.0.18+
```

> **Best practice**: Index columns used in `WHERE`, `JOIN`, and `ORDER BY`. Avoid over-indexing (slows writes).

---

## 10. Constraints

### Common constraints

```sql
CREATE TABLE orders (
    id INT AUTO_INCREMENT PRIMARY KEY,
    order_number VARCHAR(20) NOT NULL UNIQUE,
    customer_id INT NOT NULL,
    amount DECIMAL(10,2) CHECK (amount > 0),
    status ENUM('pending', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (customer_id) REFERENCES customers(id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);
```

### Add / Drop constraints later

```sql
ALTER TABLE orders
ADD CONSTRAINT fk_customer
FOREIGN KEY (customer_id) REFERENCES customers(id);

ALTER TABLE orders DROP FOREIGN KEY fk_customer;

ALTER TABLE products
ADD CONSTRAINT chk_price CHECK (price >= 0);
```

---

## 11. Views

### Create a view

```sql
CREATE VIEW high_earners AS
SELECT id, first_name, last_name, salary
FROM employees
WHERE salary > 80000;

CREATE OR REPLACE VIEW department_summary AS
SELECT d.name, COUNT(e.id) AS emp_count, AVG(e.salary) AS avg_salary
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.id, d.name;
```

### Use a view

```sql
SELECT * FROM high_earners;
SELECT * FROM department_summary WHERE emp_count > 5;
```

### Drop a view

```sql
DROP VIEW high_earners;
DROP VIEW IF EXISTS high_earners;
```

### Show views

```sql
SHOW FULL TABLES WHERE Table_type = 'VIEW';
```

---

## 12. Stored Procedures & Functions

### Create a stored procedure

```sql
DELIMITER //

CREATE PROCEDURE GetEmployeesByDept(IN dept_id INT)
BEGIN
    SELECT first_name, last_name, salary
    FROM employees
    WHERE department_id = dept_id
    ORDER BY last_name;
END //

DELIMITER ;
```

### Call a procedure

```sql
CALL GetEmployeesByDept(3);
```

### Create a function

```sql
DELIMITER //

CREATE FUNCTION GetAnnualSalary(monthly DECIMAL(10,2))
RETURNS DECIMAL(12,2)
DETERMINISTIC
BEGIN
    RETURN monthly * 12;
END //

DELIMITER ;
```

### Use a function

```sql
SELECT first_name, salary, GetAnnualSalary(salary) AS annual
FROM employees;
```

### Show & Drop

```sql
SHOW PROCEDURE STATUS WHERE Db = 'company_db';
SHOW FUNCTION STATUS WHERE Db = 'company_db';
DROP PROCEDURE GetEmployeesByDept;
DROP FUNCTION GetAnnualSalary;
```

---

## 13. Triggers

### Create a trigger

```sql
DELIMITER //

CREATE TRIGGER before_employee_update
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    IF NEW.salary < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Salary cannot be negative';
    END IF;
END //

DELIMITER ;
```

### After insert trigger (audit log example)

```sql
DELIMITER //

CREATE TRIGGER after_employee_insert
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    INSERT INTO employee_audit (employee_id, action, action_time)
    VALUES (NEW.id, 'INSERT', NOW());
END //

DELIMITER ;
```

### Show & Drop triggers

```sql
SHOW TRIGGERS;
SHOW TRIGGERS FROM company_db;
DROP TRIGGER before_employee_update;
```

---

## 14. Transactions

### Basic transaction

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 1000 WHERE id = 1;
UPDATE accounts SET balance = balance + 1000 WHERE id = 2;

-- If everything is fine:
COMMIT;

-- If something went wrong:
ROLLBACK;
```

### Savepoints

```sql
START TRANSACTION;

UPDATE ... ;
SAVEPOINT sp1;

UPDATE ... ;
SAVEPOINT sp2;

-- Rollback only to a savepoint
ROLLBACK TO sp1;

COMMIT;
```

### Check transaction isolation

```sql
SELECT @@transaction_isolation;
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

---

## 15. Users, Privileges & Security

### Create a user

```sql
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'strong_password';
CREATE USER 'app_user'@'%' IDENTIFIED BY 'strong_password';   -- any host
```

### Grant privileges

```sql
-- Full access to one database
GRANT ALL PRIVILEGES ON company_db.* TO 'app_user'@'localhost';

-- Specific privileges
GRANT SELECT, INSERT, UPDATE ON company_db.* TO 'app_user'@'localhost';
GRANT SELECT ON company_db.employees TO 'readonly'@'localhost';

-- With grant option
GRANT ALL ON company_db.* TO 'admin'@'localhost' WITH GRANT OPTION;
```

### Show grants

```sql
SHOW GRANTS FOR 'app_user'@'localhost';
SHOW GRANTS;          -- for current user
```

### Revoke privileges

```sql
REVOKE INSERT ON company_db.* FROM 'app_user'@'localhost';
REVOKE ALL PRIVILEGES ON company_db.* FROM 'app_user'@'localhost';
```

### Change password

```sql
ALTER USER 'app_user'@'localhost' IDENTIFIED BY 'new_strong_password';
```

### Drop user

```sql
DROP USER 'app_user'@'localhost';
```

### Apply privilege changes

```sql
FLUSH PRIVILEGES;
```

---

## 16. Import / Export & Backup

### Export (mysqldump)

```bash
# Full database
mysqldump -u root -p company_db > company_db_backup.sql

# Single table
mysqldump -u root -p company_db employees > employees.sql

# Structure only
mysqldump -u root -p --no-data company_db > structure.sql

# Data only
mysqldump -u root -p --no-create-info company_db > data.sql

# All databases
mysqldump -u root -p --all-databases > full_backup.sql

# With routines & triggers
mysqldump -u root -p --routines --triggers company_db > full_with_routines.sql
```

### Import

```bash
mysql -u root -p company_db < company_db_backup.sql
```

### Inside MySQL shell

```sql
SOURCE /path/to/backup.sql;
```

### Export query results to CSV (from shell)

```bash
mysql -u root -p -e "SELECT * FROM company_db.employees" | sed 's/\t/,/g' > employees.csv
```

### Load data from CSV

```sql
LOAD DATA INFILE '/var/lib/mysql-files/employees.csv'
INTO TABLE employees
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

---

## 17. Performance & Diagnostics

### Explain query plan

```sql
EXPLAIN SELECT * FROM employees WHERE last_name = 'Smith';
EXPLAIN FORMAT=JSON SELECT ... ;
EXPLAIN ANALYZE SELECT ... ;          -- MySQL 8.0.18+
```

### Show running processes

```sql
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;
```

### Kill a process

```sql
KILL 1234;            -- process id from PROCESSLIST
```

### Table status & size

```sql
SHOW TABLE STATUS FROM company_db;
SELECT 
    table_name,
    ROUND(data_length / 1024 / 1024, 2) AS data_mb,
    ROUND(index_length / 1024 / 1024, 2) AS index_mb
FROM information_schema.tables
WHERE table_schema = 'company_db'
ORDER BY data_length DESC;
```

### Optimize / Analyze / Repair table

```sql
OPTIMIZE TABLE employees;
ANALYZE TABLE employees;
REPAIR TABLE employees;               -- for MyISAM mainly
```

### Slow query log (configuration)

```sql
SHOW VARIABLES LIKE 'slow_query%';
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;       -- seconds
```

---

## 18. Useful System & Status Commands

```sql
-- Server status
SHOW STATUS;
SHOW STATUS LIKE 'Threads%';
SHOW STATUS LIKE 'Connections';

-- Variables
SHOW VARIABLES;
SHOW VARIABLES LIKE 'max_connections';
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';

-- Character set & collation
SHOW CHARACTER SET;
SHOW COLLATION;
SELECT @@character_set_database, @@collation_database;

-- Engines
SHOW ENGINES;

-- Current queries / locks (useful for debugging)
SHOW ENGINE INNODB STATUS\G

-- Binary logs
SHOW BINARY LOGS;
SHOW MASTER STATUS;
```

### Change some session variables

```sql
SET SESSION sql_mode = 'STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION';
SET SESSION time_zone = '+05:30';
```

---

## 19. Advanced Tips & Best Practices

#### 1. Always use transactions for multi-statement writes
```sql
START TRANSACTION;
-- multiple related statements
COMMIT;
```

#### 2. Prefer prepared statements (from application code) to avoid SQL injection.

#### 3. Use meaningful indexes, but don't over-index.

#### 4. Prefer `InnoDB` (default) over MyISAM for most workloads (transactions, row-level locking, foreign keys).

#### 5. Use `EXPLAIN` before optimizing slow queries.

#### 6. Keep backups regular and test restores.

#### 7. Use connection pooling in applications.

#### 8. Monitor slow query log and processlist regularly.

#### 9. For large deletes/updates, do them in batches:
```sql
DELETE FROM logs WHERE created_at < '2022-01-01' LIMIT 10000;
-- repeat until affected rows = 0
```

#### 10. Useful information_schema queries

```sql
-- All tables and their sizes
SELECT table_schema, table_name, 
       ROUND((data_length + index_length) / 1024 / 1024, 2) AS size_mb
FROM information_schema.tables
WHERE table_schema NOT IN ('mysql', 'information_schema', 'performance_schema', 'sys')
ORDER BY size_mb DESC;

-- Columns of a table
SELECT column_name, data_type, is_nullable, column_default
FROM information_schema.columns
WHERE table_schema = 'company_db' AND table_name = 'employees';
```

---

## Quick Reference Card

| Task                    | Command                                      |
|-------------------------|----------------------------------------------|
| Connect                 | `mysql -u user -p`                           |
| Show DBs                | `SHOW DATABASES;`                            |
| Use DB                  | `USE dbname;`                                |
| Show tables             | `SHOW TABLES;`                               |
| Describe table          | `DESC tablename;`                            |
| Select all              | `SELECT * FROM t;`                           |
| Insert                  | `INSERT INTO t (...) VALUES (...);`          |
| Update                  | `UPDATE t SET col=val WHERE ...;`           |
| Delete                  | `DELETE FROM t WHERE ...;`                   |
| Create index            | `CREATE INDEX idx ON t(col);`                |
| Explain plan            | `EXPLAIN SELECT ...;`                        |
| Backup                  | `mysqldump -u root -p db > backup.sql`       |
| Restore                 | `mysql -u root -p db < backup.sql`           |
| Show processlist        | `SHOW PROCESSLIST;`                          |
| Start transaction       | `START TRANSACTION;`                         |

---


