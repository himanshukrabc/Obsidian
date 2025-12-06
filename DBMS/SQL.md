Datatypes in SQL :

> [!info] SQL Data Types for MySQL, SQL Server, and MS Access  
> W3Schools offers free online tutorials, references and exercises in all the major languages of the web.  
> [https://www.w3schools.com/sql/sql_datatypes.asp](https://www.w3schools.com/sql/sql_datatypes.asp)  

  

## Types of SQL commands:

```SQL
-- Datatypes in SQL
-- https://www.w3schools.com/sql/sql_datatypes.asp

-- DATA QUERY LANGUAGE DQL

--Reading Data

SELECT * FROM workers;
SELECT col1,col2 FROM workers;

--WHERE
SELECT * FROM workers WHERE age>40 AND age<50
--BETWEEN
SELECT * FROM workers WHERE age BETWEEN(30,40) --=> Limits are both inclusive
-- IN
SELECT * FROM workers WHERE dept IN ('HR','IT',..);
--AND , OR , NOT

-- IS NULL
SELECT * FROM workers WHERE age IS NULL
--WILDCARDS
SELECT * FROM workers WHERE name LIKE '%a_a%' -- => %-any number of chars, _ - single char
--ORDER BY => sort fetched data
SELECT * FROM workers ORDER BY age

-- GROUP BY
-- > Used for data aggregation.(COUNT, MIN, MAX,)
-- > All columns that are selected/aggregated must be named
SELECT c1,c2,c3 FROM workers GROUP BY c1,c2,c3
-- > If no aggregation is defined it automatically treats it as unique
SELECT department, COUNT(department) FROM workers GROUP BY department

-- HAVING
-- Used in conjugation with GROUP BY statement, substitutes WHERE
SELECT department, COUNT(department) FROM workers GROUP BY department HAVING COUNT(department)>10

-- Constraints
-- 1) PRIMARY KEY => Use in one of the two places specified
CREATE TABLE workers(
    id INT PRIMARY KEY,
    ...,
    ...,
)PRIMARY KEY(id);

-- 2) FOREIGN KEY => 
CREATE TABLE workers(
    ...,
    ...,
    FOREIGN KEY(cust_ref) REFERENCES customer(id)
);

-- UNIQUE
-- CHECK
--DEFAULT
CREATE TABLE workers(
    BALANCE INT NOT  NULL DEFAULT 1000,
    ...,
    CONSTRAINT ACC_CHECK check (BALANCE>500),
);


-- ALTER
-- ADD COLUMNS
ALTER TABLE workers ADD new_col VARCHAR(20) NOT NULL, ADD nc2 dt,...;
-- MODIFY COLUMNS
ALTER TABLE workers MODIFY col_name new_datatype;
--CHANGE COLUMN
ALTER TABLE workers CHANGE COLUMN col_name new_col_name new_datatype;
--DROP
ALTER TABLE workers DROP COLUMN col_name;
--RENAME
ALTER TABLE workers RENAME TO new_tablw_name;


--INSERT
INSERT INTO emp VALUES (, , , , ,); -- => If you want to add values to all columns
INSERT INTO emp(col1, col2 ...) VALUES (v1, v2 ....) -- => else

--UPDATE 
UPDATE workers SET col1=v1,col2=v2,... WHERE condition;
UPDATE workers SET age=age+1;

--DELETE
DELETE FROM workers WHERE condition;
-- If a referenced tuple is to be deleted it will show foreing key constraint failure.
-- --> ON DELETE CASCADE => removes the tuple referencing the tuple to be deleted
CREATE TABLE workers(
    ...,
    ...,
    FOREIGN KEY(cust_ref) REFERENCES customer(id) ON DELETE CASCADE
);
-- --> ON DELETE SET NULL => sets the foreign key to NULL
CREATE TABLE workers(
    ...,
    ...,
    FOREIGN KEY(cust_ref) REFERENCES customer(id) ON DELETE SET NULL
);


-- REPLACE => requires PK value to be specified.
-- If data is already preset, it updates it. Else it inserts it.
REPLACE INTO Customer (id, City) VALUES (1251, 'Colony');
REPLACE INTO Customer (id, cname, City) VALUES (1333, 'codehelp', 'Colony');
REPLACE INTO Customer SET id = 1300, cname = 'Mac', City = 'Utah';
REPLACE INTO Customer(cname, City) SELECT cname, City FROM Customer WHERE id= 500; -- wont work as id is not specified

-- JOINS  ==> Column wise Combination
-- The parent/Left table , the child/right(the one with the FK) table
-- INNER JOIN/JOIN => (PK == FK) => Returns the entries having matches in both tables
SELECT employee.emp_id, employee.first_name, b.branch_name
FROM employee
INNER JOIN branch AS b   
ON employee.emp_id = branch.mgr_id;

-- OUTER JOIN
-- LEFT JOIN => (PK == FK, PK not matching) 
SELECT employee.emp_id, employee.first_name, branch.branch_name
FROM employee
LEFT JOIN branch    
ON employee.emp_id = branch.mgr_id;

-- RIGHT JOIN => (PK == FK, FK == NULL) 
SELECT employee.emp_id, employee.first_name, branch.branch_name
FROM employee
RIGHT JOIN branch    
ON employee.emp_id = branch.mgr_id;

-- FULL JOIN => (PK == FK, PK not matching , FK == NULL) 
SELECT employee.emp_id, employee.first_name, branch.branch_name
FROM employee
FULL JOIN branch    
ON employee.emp_id = branch.mgr_id;

-- SELF JOIN
SELECT A.CustomerName AS CustomerName1, B.CustomerName AS CustomerName2, A.City
FROM Customers A, Customers B
WHERE A.CustomerID <> B.CustomerID
AND A.City = B.City
ORDER BY A.City;


-- UNIONS, INTERSECTIONS, MINUS (SET OPERATIONS) ==> Row wise Combination
-- UNIONS 
-- => selected rows from two tablles are appended one after other.
-- Selected columns from both tables must have same datatypes, Number of columns must be same
SELECT * FROM tab1
UNION 
SELCT * FROM tab2;

-- INTERSECTION == INNER JOIN
-- MINUS
SELECT table1.id from table1 LEFT JOIN table2 
ON table1.id = table2.id WHERE taable2.id = NULL;

--NESTED QUERIES

-- Find names of all employees who have sold over 50,000
SELECT employee.first_name, employee.last_name
FROM employee
WHERE employee.emp_id IN (SELECT works_with.emp_id
                          FROM works_with
                          WHERE works_with.total_sales > 50000);

-- Find all clients who are handles by the branch that Michael Scott manages
-- Assume you know Michael's ID
SELECT client.client_id, client.client_name
FROM client
WHERE client.branch_id = (SELECT branch.branch_id
                          FROM branch
                          WHERE branch.mgr_id = 102);

 -- Find all clients who are handles by the branch that Michael Scott manages
 -- Assume you DONT'T know Michael's ID
 SELECT client.client_name FROM client WHERE
 client.branch_id=(SELECT branch.branch_id FROM branch 
 WHERE branch.mgr_id=( SELECT employee.emp_id FROM employee 
 WHERE employee.first_name='Michael' AND employee.last_name='Scott'));

-- Find the names of employees who work with clients handled by the scranton branch
SELECT first_name,last_name FROM employee
WHERE branch_id=2 AND emp_id IN (SElECT emp_id FROM works_with);

-- Find the names of all clients who have spent more than 100,000 dollars
SELECT client.client_name FROM client
WHERE client.client_id IN (
    SELECT client_id FROM(
        SElECT SUM(total_sales) AS total_spent,client_id
        FROM works_with GROUP BY client_id
    ) AS client_expenditure WHERE total_spent>100000
);

-- CORRELATED QUERY ==> inner query reference sthe outer query
-- Get the 3rd most aged employee
SELECT * FROM employee e1 WHERE (SELECT COUNT(id) FROM employee e2 WHERE e2.age>e1.age) = 3;

-- VIEWS
CREATE VIEW cust_view AS <query>;
ALTER VIEW cust_view AS <new_query>;
DROP VIEW IF EXISTS cust_view;
```

![[IMage/2 3.png|2 3.png]]

![[Image/2 1 2.png|2 1 2.png]]