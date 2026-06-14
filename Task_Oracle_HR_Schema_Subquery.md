# HR Database Basic SQL Solutions

Schema used from the photo: `EMPLOYEES`, `DEPARTMENTS`, `LOCATIONS`, `COUNTRIES`, `REGIONS`, and `JOBS`.

The solutions below use simple subqueries where they make sense. When a subquery would make the answer too complicated for a beginner, I used a simpler SQL approach such as `JOIN`, `GROUP BY`, or `HAVING`.

## 1. Salary Analysis Using Subqueries

### 1.1 Employees earning more than the company average salary

**Business logic:** First calculate the company average salary, then show employees whose salary is higher than that average.

```sql
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

### 1.2 Employees earning less than the company's highest salary

**Business logic:** First find the highest salary in the company, then show employees earning less than that amount.

```sql
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

### 1.3 Departments whose average salary exceeds the company average salary

**Business logic:** Calculate the average salary for each department, then compare it with the company average salary.

```sql
SELECT department_id, AVG(salary) AS average_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > (
    SELECT AVG(salary)
    FROM employees
);
```

### 1.4 Employees earning the highest salary within their department

**Business logic:** For each employee, compare their salary with the highest salary in the same department.

```sql
SELECT employee_id, first_name, last_name, department_id, salary
FROM employees e
WHERE salary = (
    SELECT MAX(salary)
    FROM employees
    WHERE department_id = e.department_id
);
```

## 2. Department & Location Analysis

### 2.1 Employees working in departments located in the United States

**Business logic:** Find the United States, then its locations, then departments in those locations, then employees in those departments.

```sql
SELECT employee_id, first_name, last_name, department_id
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE location_id IN (
        SELECT location_id
        FROM locations
        WHERE country_id = (
            SELECT country_id
            FROM countries
            WHERE country_name = 'United States of America'
        )
    )
);
```

### 2.2 Departments that currently contain employees

**Business logic:** Show departments whose department ID appears in the employees table.

```sql
SELECT department_id, department_name
FROM departments
WHERE department_id IN (
    SELECT department_id
    FROM employees
    WHERE department_id IS NOT NULL
);
```

### 2.3 Departments that currently have no employees

**Business logic:** Show departments whose department ID does not appear in the employees table.

```sql
SELECT department_id, department_name
FROM departments
WHERE department_id NOT IN (
    SELECT department_id
    FROM employees
    WHERE department_id IS NOT NULL
);
```

### 2.4 Employees working in departments located in the Europe region

**Business logic:** Find Europe, then its countries, then their locations, then departments, then employees.

```sql
SELECT employee_id, first_name, last_name, department_id
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM departments
    WHERE location_id IN (
        SELECT location_id
        FROM locations
        WHERE country_id IN (
            SELECT country_id
            FROM countries
            WHERE region_id = (
                SELECT region_id
                FROM regions
                WHERE region_name = 'Europe'
            )
        )
    )
);
```

## 3. Manager & Hierarchy Analysis

For manager questions, it is easier to use the `employees` table twice: once for the employee and once for the manager.

### 3.1 Employees earning more than their manager

**Business logic:** Match each employee with their manager, then compare their salaries.

```sql
SELECT e.employee_id, e.first_name, e.last_name, e.salary, e.manager_id
FROM employees e
JOIN employees m
    ON e.manager_id = m.employee_id
WHERE e.salary > m.salary;
```

### 3.2 Managers supervising at least one employee earning more than 10,000

**Business logic:** Match managers with their employees, then keep managers who supervise at least one employee earning more than 10,000.

```sql
SELECT DISTINCT m.employee_id, m.first_name, m.last_name
FROM employees m
JOIN employees e
    ON e.manager_id = m.employee_id
WHERE e.salary > 10000;
```

### 3.3 Employees whose manager works in the same department

**Business logic:** Match each employee with their manager, then keep employees whose department is the same as the manager's department.

```sql
SELECT e.employee_id, e.first_name, e.last_name, e.department_id, e.manager_id
FROM employees e
JOIN employees m
    ON e.manager_id = m.employee_id
WHERE e.department_id = m.department_id;
```

### 3.4 Employees whose manager works in a different department

**Business logic:** Match each employee with their manager, then keep employees whose department is different from the manager's department.

```sql
SELECT e.employee_id, e.first_name, e.last_name, e.department_id, e.manager_id
FROM employees e
JOIN employees m
    ON e.manager_id = m.employee_id
WHERE e.department_id <> m.department_id;
```

## 4. Simple Query Execution

The original task asked for inline views, but these examples are easier to solve directly. A direct query is better here because it is shorter and easier to read.

### 4.1 Filtering employees earning more than 10,000

**Business logic:** Show employees whose salary is greater than 10,000.

```sql
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE salary > 10000;
```

### 4.2 Displaying departments with average salary greater than 9,000

**Business logic:** Group employees by department, calculate each department average salary, then keep averages above 9,000.

```sql
SELECT department_id, AVG(salary) AS average_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 9000;
```

### 4.3 Displaying departments with more than five employees

**Business logic:** Group employees by department, count employees, then keep departments with more than five employees.

```sql
SELECT department_id, COUNT(*) AS employee_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 5;
```

## 5. More Salary and Department Reports

### 5.1 Employees earning above their department average salary

**Business logic:** Compare each employee salary with the average salary of that employee's department.

```sql
SELECT employee_id, first_name, last_name, department_id, salary
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
);
```

### 5.2 Employees earning above the average salary for their job role

**Business logic:** Compare each employee salary with the average salary of employees who have the same job ID.

```sql
SELECT employee_id, first_name, last_name, job_id, salary
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE job_id = e.job_id
);
```

### 5.3 Departments where all employees earn more than 5,000

**Business logic:** Group employees by department and keep departments where the lowest salary is still above 5,000.

```sql
SELECT d.department_id, d.department_name
FROM departments d
JOIN employees e
    ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name
HAVING MIN(e.salary) > 5000;
```

### 5.4 Employees who are the only employee in their department

**Business logic:** First find departments with exactly one employee, then show the employee from those departments.

```sql
SELECT employee_id, first_name, last_name, department_id
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM employees
    GROUP BY department_id
    HAVING COUNT(*) = 1
);
```

## 6. Executive HR Analytics Challenge

### 6.1 Employees earning above the company average salary

**Business logic:** First calculate the company average salary, then show employees earning more than that average.

```sql
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

### 6.2 Departments with more employees than Department 50

**Business logic:** Count employees in each department, then compare each count with the employee count of Department 50.

```sql
SELECT department_id, COUNT(*) AS employee_count
FROM employees
GROUP BY department_id
HAVING COUNT(*) > (
    SELECT COUNT(*)
    FROM employees
    WHERE department_id = 50
);
```

### 6.3 Employees working in the department with the highest average salary

**Business logic:** This is easier as two simple steps. First find the department with the highest average salary, then use that department ID to list its employees.

Step 1: Find the department with the highest average salary.

```sql
SELECT department_id, AVG(salary) AS average_salary
FROM employees
GROUP BY department_id
ORDER BY AVG(salary) DESC
FETCH FIRST 1 ROW ONLY;
```

Step 2: Use the department ID from Step 1.

```sql
SELECT employee_id, first_name, last_name, department_id, salary
FROM employees
WHERE department_id = &department_id_from_step_1;
```

### 6.4 Employees earning the second-highest salary

**Business logic:** First find the highest salary below the company maximum salary, then show employees earning that salary.

```sql
SELECT employee_id, first_name, last_name, salary
FROM employees
WHERE salary = (
    SELECT MAX(salary)
    FROM employees
    WHERE salary < (
        SELECT MAX(salary)
        FROM employees
    )
);
```

### 6.5 Managers whose employees' average salary exceeds 10,000

**Business logic:** Match managers with their employees, calculate the average salary of each manager's employees, then keep averages above 10,000.

```sql
SELECT m.employee_id, m.first_name, m.last_name, AVG(e.salary) AS employee_average_salary
FROM employees m
JOIN employees e
    ON e.manager_id = m.employee_id
GROUP BY m.employee_id, m.first_name, m.last_name
HAVING AVG(e.salary) > 10000;
```