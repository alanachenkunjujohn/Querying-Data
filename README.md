# Querying-Data
# Step-by-Step Guide: 

This document explains the workflow, SQL logic, and verification steps I used to complete ** Querying Data** as a continuation of DDL & Constarints

---

## Step 1: Connecting to the Existing Database
Instead of recreating or dropping the tables from Assignment 1, I opened a new query tab in **MySQL Workbench** and selected the existing `employee` database:

```sql
USE employee;
```

---

## Step 2: Populating the Tables in Foreign Key Order
Because `employees` has foreign key constraints referencing both `departments` and `location`, I inserted records into the parent tables first before inserting the 30 employee records:
1. Inserted 13 departments into `departments`.
2. Inserted 4 cities (`Chennai`, `Bangalore`, `Hyderabad`, `Pune`) into `location`, letting `AUTO_INCREMENT` assign IDs `1` through `4`.
3. Inserted 30 employees (`5001`–`5030`) into `employees`, matching valid `department_id` and `location_id` values.

---

## Step 3: Basic Querying & Data Cleaning

### 1. Retrieving Distinct Salaries & Using Aliases
I used `DISTINCT` to remove duplicate salary figures and `AS` to display readable column headers:

```sql
SELECT DISTINCT salary FROM employees;

SELECT 
    age AS Employee_Age,
    salary AS Employee_Salary
FROM employees;
```

### 2. Filtering with `WHERE` & Updating Missing Data
* Filtered employees earning over ₹50,000 who were hired before `2016-01-01` (returning 5 employees).
* Located employee `5004` (`Kiara Malhotra`), whose `designation` was `NULL`, and updated it to `'Data Scientist'`. I temporarily disabled MySQL Workbench's safe update mode so I could filter directly by `designation IS NULL`:

```sql
SELECT * FROM employees
WHERE salary > 50000 AND hire_date < '2016-01-01';

SET SQL_SAFE_UPDATES = 0;

UPDATE employees
SET designation = 'Data Scientist'
WHERE designation IS NULL;

SET SQL_SAFE_UPDATES = 1;
```

---

## Step 4: Sorting, Limiting & Aggregating Data

### 1. `ORDER BY` and `LIMIT`
* Sorted all employees by `department_id` ascending (`ASC`) and `salary` descending (`DESC`).
* Used `YEAR(hire_date) = 2018` with `LIMIT 5` to display the first five employees hired in 2018:

```sql
SELECT * FROM employees
ORDER BY department_id ASC, salary DESC;

SELECT * FROM employees
WHERE YEAR(hire_date) = 2018
LIMIT 5;
```

### 2. Aggregate Functions (`SUM` & `MIN`)
* Joined `employees` with `departments` to calculate the total salary budget for the **Finance** department (`170000.00`).
* Used `MIN(age)` to find the youngest employee age (`25`):

```sql
SELECT SUM(e.salary) AS Total_Finance_Salary
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE d.department_name = 'Finance';

SELECT MIN(age) AS Minimum_Age FROM employees;
```

### 3. `GROUP BY` and `HAVING`
* Grouped by `location` to find the highest salary in each city, and used `LIKE '%Analyst%'` to calculate average salaries across all analyst roles.
* Used `HAVING` to filter grouped results—identifying departments with fewer than 3 employees and locations where female employees average under 30 years old:

```sql
-- Maximum salary per location
SELECT l.location, MAX(e.salary) AS Maximum_Salary
FROM employees e
JOIN location l ON e.location_id = l.location_id
GROUP BY l.location;

-- Average salary for Analyst roles
SELECT designation, AVG(salary) AS Average_Salary
FROM employees
WHERE designation LIKE '%Analyst%'
GROUP BY designation;

-- Departments with fewer than 3 employees
SELECT d.department_name, COUNT(e.employee_id) AS Employee_Count
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name
HAVING COUNT(e.employee_id) < 3;

-- Locations where female employees have an average age below 30
SELECT l.location, AVG(e.age) AS Avg_Female_Age
FROM employees e
JOIN location l ON e.location_id = l.location_id
WHERE e.gender = 'F'
GROUP BY l.location_id, l.location
HAVING AVG(e.age) < 30;
```

---

## Step 5: Combining Tables with Joins
Finally, I wrote three join queries to link employee records with their departments and locations:
* **`INNER JOIN`**: Displayed employee names, designations, and department names for matched records.
* **`LEFT JOIN`**: Listed all 13 departments alongside their employee headcounts (ensuring departments with zero employees would still appear with `0`).
* **`RIGHT JOIN`**: Listed all locations alongside assigned employee names (displaying `NULL` if a location had no employees).

```sql
-- Inner Join
SELECT e.employee_name, e.designation, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;

-- Left Join
SELECT d.department_name, COUNT(e.employee_id) AS Total_Employees
FROM departments d
LEFT JOIN employees e ON d.department_id = e.department_id
GROUP BY d.department_id, d.department_name;

-- Right Join
SELECT l.location, e.employee_name
FROM employees e
RIGHT JOIN location l ON e.location_id = l.location_id;
```

---

## Step 6: Verification
I verified every query in **MySQL Workbench** by checking the **Result Grid** and **Action Output** logs after each execution to confirm zero warnings or constraint violations.
