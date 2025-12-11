## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `sqlLibraryProject-2`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

![libraryimage](https://github.com/user-attachments/assets/7bd045d0-e582-4268-88da-5e13c3ca4e7f)

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup
![ERD]( https://github.com/najirh/Library-System-Management---P2/blob/main/library_erd.png?raw=true )

- **Database Creation**: Created a database named `sqlLibraryProject-2`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
CREATE DATABASE sqlLibraryProject-2;

DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
            branch_id VARCHAR(10) PRIMARY KEY,
            manager_id VARCHAR(10),
            branch_address VARCHAR(30),
            contact_no VARCHAR(15)
);


-- Create table "Employee"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
            emp_id VARCHAR(10) PRIMARY KEY,
            emp_name VARCHAR(30),
            position VARCHAR(30),
            salary DECIMAL(10,2),
            branch_id VARCHAR(10),
            FOREIGN KEY (branch_id) REFERENCES  branch(branch_id)
);


-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
            member_id VARCHAR(10) PRIMARY KEY,
            member_name VARCHAR(30),
            member_address VARCHAR(30),
            reg_date DATE
);



-- Create table "Books"
DROP TABLE IF EXISTS books;
CREATE TABLE books
(
            isbn VARCHAR(50) PRIMARY KEY,
            book_title VARCHAR(80),
            category VARCHAR(30),
            rental_price DECIMAL(10,2),
            status VARCHAR(10),
            author VARCHAR(30),
            publisher VARCHAR(30)
);



-- Create table "IssueStatus"
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
(
            issued_id VARCHAR(10) PRIMARY KEY,
            issued_member_id VARCHAR(30),
            issued_book_name VARCHAR(80),
            issued_date DATE,
            issued_book_isbn VARCHAR(50),
            issued_emp_id VARCHAR(10),
            FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
            FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
            FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn) 
);



-- Create table "ReturnStatus"
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(30),
            return_book_name VARCHAR(80),
            return_date DATE,
            return_book_isbn VARCHAR(50),
            FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);

```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
SELECT * FROM books;
```
**Task 2: Update an Existing Member's Address**

```sql
UPDATE members
SET member_address = '987 Main St'
where member_name = 'Alice Johnson';
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS114' from the issued_status table.

```sql
DELETE FROM issued_status
WHERE   issued_id =   'IS114';
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101'
```


**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
SELECT 
    ist.issued_emp_id,
     e.emp_name
    -- COUNT(*)
FROM issued_status as ist
JOIN
employees as e
ON e.emp_id = ist.issued_emp_id
GROUP BY 1, 2
HAVING COUNT(ist.issued_id) > 1;
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
CREATE TABLE book_cnts
AS    
SELECT 
    b.isbn,
    b.book_title,
    COUNT(ist.issued_id) as no_issued
FROM books as b
JOIN
issued_status as ist
ON ist.issued_book_isbn = b.isbn
GROUP BY 1, 2;
```


### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
SELECT * FROM books
WHERE category = 'Classic';
```

8. **Task 8: Find Total Rental Income by Category**:

```sql
SELECT
    b.category,
    SUM(b.rental_price),
    COUNT(*)
FROM books as b
JOIN
issued_status as ist
ON ist.issued_book_isbn = b.isbn
GROUP BY 1;
```

9. **List Members Who Registered in the Last 10 Days**:
```sql
SELECT * FROM members
where reg_date >= CURRENT_DATE - INTERVAL '10days';
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
SELECT 
	e.emp_id,
	e.emp_name,
	b.branch_id,
	b.manager_id,
    b.branch_address,
	e2.emp_name
FROM employees e
JOIN branch b
on e.branch_id = b.branch_id
JOIN employees e2
on e2.emp_id = b.manager_id;
```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
CREATE TABLE books_above_7usd
AS
SELECT * FROM books
where rental_price >=7.00;

SELECT * FROM books_above_7usd;
```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
SELECT 
	*
FROM issued_status i
LEFT JOIN return_status r
ON i.issued_id = r.issued_id
WHERE r.return_id is null
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
SELECT
	m.member_id,
	m.member_name,
	b.book_title,
	i.issued_date
FROM issued_status i
JOIN members m
ON m.member_id = i.issued_member_id
JOIN books b
ON b.isbn = i.issued_book_isbn
LEFT JOIN return_status r
ON i.issued_id = r.issued_id
WHERE r.return_id is null
AND (CURRENT_DATE - i.issued_date)>30;
```


**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).


```sql

CREATE OR REPLACE PROCEDURE add_return_books(p_return_id varchar(10),p_issued_id varchar(30))
LANGUAGE plpgsql
as $$
DECLARE 
	v_isbn varchar(50);
	v_issued_book_name varchar(80);

 BEGIN
 	INSERT INTO return_status(return_id,issued_id,return_date)
	 VALUES (p_return_id,p_issued_id,CURRENT_DATE);

	 SELECT 
	 	issued_book_isbn,
		issued_book_name
		INTO
		v_isbn,
		v_issued_book_name
	 FROM issued_status
	 WHERE issued_id = p_issued_id;
	 
	 UPDATE books
	 SET status = 'yes'
	 where isbn = v_isbn;

	 
    RAISE NOTICE 'Thank you for returning the book: %', v_issued_book_name;
 END;
$$

CALL add_return_books('RS137','IS34');
-- SELECT * FROM books 
-- WHERE isbn = '978-0-307-58837-1'

-- SELECT * FROM issued_status
-- WHERE issued_book_isbn = '978-0-375-41398-8'
-- SELECT * FROM books
-- "978-0-375-41398-8"

```




**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
CREATE TABLE branch_reports
AS
SELECT 
	b.branch_id,
	COUNT(i.issued_id) as num_of_books,
	COUNT(return_id) as num_of_returns_books,
	SUM(rental_price) as total
FROM issued_status i
JOIN employees e
on i.issued_emp_id = e.emp_id
JOIN branch b
on b.branch_id = e.branch_id
LEFT JOIN return_status r
on r.issued_id = i.issued_id
JOIN books bo
on bo.isbn = i.issued_book_isbn
GROUP BY b.branch_id;

SELECT * FROM branch_reports;
```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql

INSERT INTO issued_status(issued_id,issued_member_id,issued_book_name,issued_date)
VALUES ('IS105','C109','Game of Thrones','2025-08-02')

CREATE TABLE active_members
AS
SELECT 
	m.member_id,
	m.member_name,
	i.issued_book_name,
	i.issued_date
FROM members m
LEFT JOIN issued_status i
ON m.member_id = i.issued_member_id
WHERE i.issued_date >= CURRENT_DATE - INTERVAL '6months';

-- 'c102' 2024-03-26 >= 2024-08-07 - 2024-06-07
--        2024-03-26 >= 2024-02-07
```


**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
SELECT 
    e.emp_name,
    b.*,
    COUNT(i.issued_id) as no_book_issued
FROM issued_status as i
JOIN employees as e
ON e.emp_id = i.issued_emp_id
JOIN branch as b
ON e.branch_id = b.branch_id
GROUP BY 1, 2;
```

**Task 19: Stored Procedure**
Objective:
Create a stored procedure to manage the status of books in a library system.
Description:
Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows:
The stored procedure should take the book_id as an input parameter.
The procedure should first check if the book is available (status = 'yes').
If the book is available, it should be issued, and the status in the books table should be updated to 'no'.
If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.

```sql

CREATE OR REPLACE PROCEDURE issue_books(p_issued_id varchar(10),p_issued_member_id varchar(30),
                                          p_issued_book_isbn varchar(50),p_issued_emp_id varchar(10))
LANGUAGE plpgsql
AS $$
DECLARE 
     v_status varchar (10);
BEGIN 
	SELECT 
		status
		INTO
		v_status
	FROM books
	WHERE isbn = p_issued_book_isbn;

	IF v_status = 'yes' THEN 
	INSERT INTO issued_status(issued_id,issued_member_id,issued_date,
	                              issued_book_isbn,issued_emp_id)
	VALUES(p_issued_id,p_issued_member_id,CURRENT_DATE,
                                          p_issued_book_isbn,p_issued_emp_id);

	RAISE NOTICE 'the book u requested is being issued and added succesfully book isbn: %',p_issued_book_isbn;
	
    UPDATE books
	SET status = 'no'
	WHERE isbn = p_issued_book_isbn;
	
	ELSE
	RAISE NOTICE 'the book u requested is not available book isbn: %',p_issued_book_isbn;
	END IF;
END;
$$

CALL issue_books('IS155','C108','978-0-553-29698-2','E104')
-- SELECT * FROM books
-- WHERE isbn = '978-0-553-29698-2'
-- "978-0-553-29698-2";
```

## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.

## How to Use

1. **Clone the Repository**: Clone this repository to your local machine.
   ```sh
   https://github.com/pandebipinX/Library-System-Mangagement-sql.git
   ```

2. **Set Up the Database**: Execute the SQL scripts in the `database_setup.sql` file to create and populate the database.
3. **Run the Queries**: Use the SQL queries in the `analysis_queries.sql` file to perform the analysis.
4. **Explore and Modify**: Customize the queries as needed to explore different aspects of the data or answer additional questions.

## Author - Bipin Pandey

- **LinkedIn**: [Connect with me professionally](https://www.linkedin.com/in/bipin-pandey-860289287?lipi=urn%3Ali%3Apage%3Ad_flagship3_profile_view_base_contact_details%3BiW7kZb7IRLmuEYYDA7vpGw%3D%3D)

