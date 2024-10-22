# MYSQL_CTEs
Exploring Common Table Expressions (CTEs) in MySQL for hierarchical salary analysis and employee statistics.

CREATE DATABASE challenge27;

USE challenge27;

CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(100),
    salary DECIMAL(10, 2)
);

INSERT INTO employees (name, department, salary) VALUES
('Alice', 'HR', 60000),
('Bob', 'Engineering', 70000),
('Charlie', 'HR', 65000),
('David', 'Engineering', 80000),
('Eve', 'Marketing', 55000);

SELECT * FROM employees;

![table](https://github.com/user-attachments/assets/04771d67-7072-40ed-b5ed-f6d44396c909)

#1. What is the average salary per department?

WITH AvgSalaries AS (
    SELECT department, AVG(salary) AS average_salary
    FROM employees
    GROUP BY department
)

SELECT * FROM AvgSalaries;

![q1](https://github.com/user-attachments/assets/5d3ed3ef-308a-46f6-92b8-d2687e98a28a)

#2.Non-Recursive CTE: Find the total salary of all employees.

WITH TotalSalary AS (
    SELECT SUM(salary) AS total_salary
    FROM employees
)

SELECT * FROM TotalSalary;

![q2](https://github.com/user-attachments/assets/d3f122af-4f55-4a80-81df-439d2fe1192a)


#3. Create a recursive CTE to display the salary hierarchy, starting from the highest salary down to the lowest.

WITH RECURSIVE SalaryHierarchy AS (
    SELECT name, salary, 1 AS level
    FROM employees
    WHERE salary = (SELECT MAX(salary) FROM employees)  -- Start with the highest salary
    UNION ALL
    SELECT e.name, e.salary, sh.level + 1
    FROM employees e
    JOIN SalaryHierarchy sh ON e.salary < sh.salary
    WHERE e.salary IS NOT NULL  -- Prevent null salaries from joining
)

SELECT * FROM SalaryHierarchy
ORDER BY salary DESC;

![q3](https://github.com/user-attachments/assets/639854e3-c167-45c7-889e-b33142fdcd8b)

#4. Combine Non-Recursive and Recursive CTEs

#How can you get a list of departments, their average salaries, and employees earning above average, while ranking them?

WITH AvgSalaries AS (
    SELECT department, AVG(salary) AS average_salary
    FROM employees
    GROUP BY department
),
AboveAverageEmployees AS (
    SELECT e.name, e.department, e.salary
    FROM employees e
    JOIN AvgSalaries a ON e.department = a.department
    WHERE e.salary > a.average_salary
),
RankedAboveAverage AS (
    SELECT 
        name, 
        department, 
        salary, 
        RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
    FROM AboveAverageEmployees
)

SELECT * FROM RankedAboveAverage;

![q4](https://github.com/user-attachments/assets/7a7d73c1-c626-4bbd-9784-71814527ccdf)

#5.Non-Recursive CTE to Count Employees in Each Department and Recursive CTE for Salary Growth
#Find the number of employees in each department and generate a recursive list of salaries starting from the highest down to the lowest.
WITH EmployeeCounts AS (
    SELECT department, COUNT(*) AS employee_count
    FROM employees
    GROUP BY department
),
SalaryGrowth AS (
    SELECT name, salary, 1 AS level
    FROM employees
    WHERE salary = (SELECT MAX(salary) FROM employees)  -- Start with the highest salary
    UNION ALL
    SELECT e.name, e.salary, sg.level + 1
    FROM employees e
    JOIN SalaryGrowth sg ON e.salary < sg.salary
    WHERE e.salary IS NOT NULL
)
SELECT ec.department, ec.employee_count, sg.name, sg.salary
FROM EmployeeCounts ec
LEFT JOIN SalaryGrowth sg ON sg.salary IS NOT NULL
ORDER BY ec.department, sg.salary DESC;































