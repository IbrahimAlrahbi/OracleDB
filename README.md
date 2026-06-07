# Oracle Database Setup Guide (SQL*Plus + SQL Developer)

## 1. Connect as SYSDBA

Open SQL*Plus and connect as a database administrator:


```sql
sqlplus sys as sysdba
```

---

## 2. Check Available Pluggable Databases (PDBs)

```sql
SHOW PDBS;
```

Example output:

```text
CON_NAME
------------------------------
ORCLPDB
```

---

## 3. Switch to the Target PDB

```sql
ALTER SESSION SET CONTAINER = ORCLPDB;
```

Verify:

```sql
SHOW CON_NAME;
```

Expected:

```text
ORCLPDB
```

---

## 4. Create a New User

If the password contains special characters, use double quotes:

```sql
CREATE USER 'user_name' IDENTIFIED BY 'password';
```

---

## 5. Grant Required Privileges

Allow login:

```sql
GRANT CREATE SESSION TO ibrahim;
```

Allow object creation:

```sql
GRANT CREATE TABLE TO ibrahim;
GRANT CREATE VIEW TO ibrahim;
GRANT CREATE SEQUENCE TO ibrahim;
GRANT CREATE PROCEDURE TO ibrahim;
```

---

## 6. Assign Tablespace Quota

```sql
ALTER USER ibrahim
DEFAULT TABLESPACE USERS
QUOTA UNLIMITED ON USERS;
```

---

## 7. Verify User Creation

```sql
SELECT username, account_status
FROM dba_users
WHERE username = 'IBRAHIM';
```

Expected:

```text
IBRAHIM    OPEN
```

---

## 8. Create a Connection in SQL Developer

Connection settings:

* Username: 
* Password: 
* Role: Default
* Connection Type: Basic
* Hostname: localhost
* Port: 1521
* Service Name: ORCLPDB

Click **Test** and then **Connect**.

---

## 9. Create Tables

Example:

```sql
CREATE TABLE employees (
    employee_id NUMBER PRIMARY KEY,
    first_name  VARCHAR2(50),
    last_name   VARCHAR2(50),
    salary      NUMBER(10,2)
);
```

---

## 10. Verify Table Creation

List tables:

```sql
SELECT table_name
FROM user_tables;
```

Describe table structure:

```sql
DESC employees;
```

---

## 11. Insert Data

```sql
INSERT INTO employees
VALUES (1, 'Ibrahim', 'Said', 1500);

COMMIT;
```

---

## 12. Query Data

```sql
SELECT * FROM employees;
```

---

## Common Issues

### ORA-00922: Missing or Invalid Option

Cause:

```sql
CREATE USER ibrahim IDENTIFIED BY @Ibra7bi;
```

Fix:

```sql
CREATE USER ibrahim IDENTIFIED BY "@Ibra7bi";
```

### ORA-12514: Listener Does Not Currently Know of Service

Cause:

* Wrong Service Name in SQL Developer.

Fix:

* Check available services:

```sql
SELECT name FROM v$services;
```

* Use the correct Service Name (e.g., ORCLPDB).

### User Cannot Create Tables

Grant permission:

```sql
GRANT CREATE TABLE TO username;
```

And ensure quota exists:

```sql
ALTER USER username QUOTA UNLIMITED ON USERS;
```
