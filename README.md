# Smart-library-management-system-
-- =====================================================
-- Smart Library Management System
-- SQL Script with all required queries
-- =====================================================

-- Drop existing tables if they exist
DROP TABLE IF EXISTS Transactions;
DROP TABLE IF EXISTS Members;
DROP TABLE IF EXISTS Books;
DROP TABLE IF EXISTS Authors;

-- =====================================================
-- 1. CREATE TABLES (Database Schema)
-- =====================================================

-- Authors Table
CREATE TABLE Authors (
    author_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100)
);

-- Books Table
CREATE TABLE Books (
    book_id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(200) NOT NULL,
    author_id INT,
    category VARCHAR(50),
    isbn VARCHAR(20),
    published_date DATE,
    price DECIMAL(10, 2),
    available_copies INT DEFAULT 0,
    FOREIGN KEY (author_id) REFERENCES Authors(author_id)
);

-- Members Table
CREATE TABLE Members (
    member_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100),
    phone_number VARCHAR(20),
    membership_date DATE NOT NULL
);

-- Transactions Table
CREATE TABLE Transactions (
    transaction_id INT PRIMARY KEY AUTO_INCREMENT,
    member_id INT NOT NULL,
    book_id INT NOT NULL,
    borrow_date DATE NOT NULL,
    return_date DATE,
    fine_amount DECIMAL(10, 2) DEFAULT 0.00,
    FOREIGN KEY (member_id) REFERENCES Members(member_id),
    FOREIGN KEY (book_id) REFERENCES Books(book_id)
);

-- =====================================================
-- 2. INSERT SAMPLE DATA
-- =====================================================

-- Insert Authors
INSERT INTO Authors (name, email) VALUES
('J.K. Rowling', 'jkrowling@example.com'),
('George Orwell', 'gorwell@example.com'),
('Isaac Asimov', 'iasimov@example.com'),
('Agatha Christie', 'achristie@example.com'),
('Stephen King', 'sking@example.com');

-- Insert Books
INSERT INTO Books (title, author_id, category, isbn, published_date, price, available_copies) VALUES
('Harry Potter and the Philosopher''s Stone', 1, 'Fantasy', '978-0747532699', '1997-06-26', 29.99, 3),
('1984', 2, 'Science', '978-0451524935', '1949-06-08', 15.99, 5),
('Foundation', 3, 'Science', '978-0553293357', '1951-05-01', 18.99, 2),
('Murder on the Orient Express', 4, 'Mystery', '978-0062693662', '1934-01-01', 12.99, 4),
('The Shining', 5, 'Horror', '978-0307743657', '1977-01-28', 22.99, 2),
('Animal Farm', 2, 'Fiction', '978-0452284244', '1945-08-17', 14.99, 6),
('I, Robot', 3, 'Science', '978-0553382563', '2016-03-15', 16.99, 3),
('Death on the Nile', 4, 'Mystery', '978-0062073556', '1998-09-01', 13.99, 3),
('IT', 5, 'Horror', '978-1501142970', '2021-07-01', 25.99, 1),
('The Stand', 5, 'Horror', '978-0307743688', '2022-05-10', 28.99, 2);

-- Insert Members
INSERT INTO Members (name, email, phone_number, membership_date) VALUES
('Alice Johnson', 'alice@example.com', '555-0101', '2020-03-15'),
('Bob Smith', 'bob@example.com', '555-0102', '2021-06-20'),
('Carol White', 'carol@example.com', '555-0103', '2022-01-10'),
('David Brown', 'david@example.com', '555-0104', '2023-04-05'),
('Eve Davis', 'eve@example.com', '555-0105', '2023-08-12');

-- Insert Transactions
INSERT INTO Transactions (member_id, book_id, borrow_date, return_date, fine_amount) VALUES
(1, 1, '2023-11-01', '2023-11-15', 0.00),
(1, 2, '2023-11-20', '2023-12-05', 0.00),
(2, 3, '2023-10-15', '2023-11-01', 0.00),
(2, 4, '2023-12-01', NULL, 0.00),
(3, 5, '2023-09-10', '2023-09-25', 5.00),
(3, 6, '2023-12-10', NULL, 0.00),
(4, 7, '2023-08-01', '2023-08-20', 0.00),
(5, 8, '2023-07-15', '2023-08-05', 10.00);

-- =====================================================
-- 3. BASIC CRUD OPERATIONS
-- =====================================================

-- INSERT new book
INSERT INTO Books (title, author_id, category, isbn, published_date, price, available_copies) 
VALUES ('Sample Book', 1, 'Fiction', '978-1234567890', '2023-01-01', 19.99, 5);

-- UPDATE book availability (after borrowing)
UPDATE Books 
SET available_copies = available_copies - 1 
WHERE book_id = 1 AND available_copies > 0;

-- DELETE member who hasn't borrowed any books
DELETE FROM Members 
WHERE member_id NOT IN (SELECT DISTINCT member_id FROM Transactions) 
AND membership_date < '2020-01-01';

-- RETRIEVE all books with available copies
SELECT * FROM Books WHERE available_copies > 0;

-- =====================================================
-- 4. SQL CLAUSES (WHERE, HAVING, LIMIT)
-- =====================================================

-- Get books published after 2015
SELECT * FROM Books 
WHERE published_date > '2015-01-01'
ORDER BY published_date DESC;

-- Retrieve the top 5 most expensive books
SELECT title, price 
FROM Books 
ORDER BY price DESC 
LIMIT 5;

-- Find members who joined before 2022
SELECT * FROM Members 
WHERE membership_date < '2022-01-01'
ORDER BY membership_date;

-- =====================================================
-- 5. SQL OPERATORS (AND, OR, NOT)
-- =====================================================

-- Get books where category = 'Science' AND price < 500
SELECT * FROM Books 
WHERE category = 'Science' AND price < 500;

-- Find all books that are available for borrowing
SELECT * FROM Books 
WHERE available_copies > 0;

-- List members who joined after 2020 OR have borrowed more than 3 books
SELECT DISTINCT m.* 
FROM Members m
LEFT JOIN Transactions t ON m.member_id = t.member_id
WHERE m.membership_date > '2020-01-01'
GROUP BY m.member_id
HAVING COUNT(t.transaction_id) > 3 OR m.membership_date > '2020-01-01';

-- =====================================================
-- 6. SORTING & GROUPING DATA (ORDER BY, GROUP BY)
-- =====================================================

-- List all books sorted by title in alphabetical order
SELECT * FROM Books 
ORDER BY title ASC;

-- Display the number of books borrowed by each member
SELECT m.name, COUNT(t.transaction_id) AS books_borrowed
FROM Members m
LEFT JOIN Transactions t ON m.member_id = t.member_id
GROUP BY m.member_id, m.name
ORDER BY books_borrowed DESC;

-- Group books by category and show the total count
SELECT category, COUNT(*) AS total_books
FROM Books
GROUP BY category
ORDER BY total_books DESC;

-- =====================================================
-- 7. AGGREGATE FUNCTIONS (SUM, AVG, MAX, MIN, COUNT)
-- =====================================================

-- Find the total number of books in each category
SELECT category, COUNT(*) AS total_books
FROM Books
GROUP BY category;

-- Calculate the average price of books in the library
SELECT AVG(price) AS average_price
FROM Books;

-- Identify the most borrowed book
SELECT b.title, COUNT(t.transaction_id) AS borrow_count
FROM Books b
JOIN Transactions t ON b.book_id = t.book_id
GROUP BY b.book_id, b.title
ORDER BY borrow_count DESC
LIMIT 1;

-- Calculate the total fines collected
SELECT SUM(fine_amount) AS total_fines
FROM Transactions;

-- =====================================================
-- 8. PRIMARY & FOREIGN KEY RELATIONSHIPS
-- =====================================================

-- Ensure books are linked to authors (already established in CREATE TABLE)
-- Establish relationships between members and transactions (already established)

-- Query to verify relationships
SELECT b.title, a.name AS author_name
FROM Books b
INNER JOIN Authors a ON b.author_id = a.author_id;

-- =====================================================
-- 9. IMPLEMENT JOINS (INNER, LEFT, RIGHT, FULL OUTER)
-- =====================================================

-- INNER JOIN: Retrieve books with their author names
SELECT b.book_id, b.title, a.name AS author_name, b.category
FROM Books b
INNER JOIN Authors a ON b.author_id = a.author_id;

-- LEFT JOIN: Get member details who have borrowed books
SELECT m.name, b.title, t.borrow_date
FROM Members m
LEFT JOIN Transactions t ON m.member_id = t.member_id
LEFT JOIN Books b ON t.book_id = b.book_id;

-- RIGHT JOIN: Find books that haven't been borrowed
SELECT b.title, t.transaction_id
FROM Transactions t
RIGHT JOIN Books b ON t.book_id = b.book_id
WHERE t.transaction_id IS NULL;

-- FULL OUTER JOIN: Show members who have never borrowed a book
-- Note: MySQL doesn't support FULL OUTER JOIN directly, use UNION
SELECT m.name, 'Never borrowed' AS status
FROM Members m
LEFT JOIN Transactions t ON m.member_id = t.member_id
WHERE t.transaction_id IS NULL
UNION
SELECT m.name, 'Has borrowed' AS status
FROM Members m
INNER JOIN Transactions t ON m.member_id = t.member_id;

-- =====================================================
-- 10. USE SUBQUERIES
-- =====================================================

-- Find books borrowed by members who registered after 2022
SELECT b.title
FROM Books b
WHERE b.book_id IN (
    SELECT t.book_id
    FROM Transactions t
    JOIN Members m ON t.member_id = m.member_id
    WHERE m.membership_date > '2022-01-01'
);

-- Identify the most borrowed book using a subquery
SELECT title
FROM Books
WHERE book_id = (
    SELECT book_id
    FROM Transactions
    GROUP BY book_id
    ORDER BY COUNT(*) DESC
    LIMIT 1
);

-- Get members who have never borrowed a book
SELECT name
FROM Members
WHERE member_id NOT IN (
    SELECT DISTINCT member_id FROM Transactions
);

-- =====================================================
-- 11. DATE & TIME FUNCTIONS
-- =====================================================

-- Extract the year from published_date to count books by publication year
SELECT YEAR(published_date) AS publication_year, COUNT(*) AS books_count
FROM Books
GROUP BY YEAR(published_date)
ORDER BY publication_year DESC;

-- Find the difference in days between borrow_date and return_date
SELECT transaction_id, member_id, book_id,
       DATEDIFF(return_date, borrow_date) AS days_borrowed
FROM Transactions
WHERE return_date IS NOT NULL;

-- Calculate late return fines (assume $1 per day after 14 days)
SELECT transaction_id, member_id, book_id,
       DATEDIFF(COALESCE(return_date, CURDATE()), borrow_date) AS days_kept,
       CASE 
           WHEN DATEDIFF(COALESCE(return_date, CURDATE()), borrow_date) > 14 
           THEN (DATEDIFF(COALESCE(return_date, CURDATE()), borrow_date) - 14) * 1.0
           ELSE 0
       END AS calculated_fine
FROM Transactions;

-- Format borrow_date as DD-MM-YYYY
SELECT transaction_id, 
       DATE_FORMAT(borrow_date, '%d-%m-%Y') AS formatted_borrow_date
FROM Transactions;

-- =====================================================
-- 12. STRING MANIPULATION FUNCTIONS
-- =====================================================

-- Convert all book titles to uppercase
SELECT UPPER(title) AS uppercase_title
FROM Books;

-- Trim whitespace from author names
UPDATE Authors
SET name = TRIM(name);

-- Replace missing email values with "Not Provided"
SELECT name, COALESCE(email, 'Not Provided') AS email
FROM Authors;

-- =====================================================
-- 13. WINDOW FUNCTIONS
-- =====================================================

-- Rank books based on the number of times they've been borrowed
SELECT b.title, 
       COUNT(t.transaction_id) AS borrow_count,
       RANK() OVER (ORDER BY COUNT(t.transaction_id) DESC) AS rank
FROM Books b
LEFT JOIN Transactions t ON b.book_id = t.book_id
GROUP BY b.book_id, b.title;

-- Show cumulative number of books borrowed per member
SELECT m.name,
       t.borrow_date,
       SUM(1) OVER (PARTITION BY m.member_id ORDER BY t.borrow_date) AS cumulative_borrowed
FROM Members m
JOIN Transactions t ON m.member_id = t.member_id
ORDER BY m.name, t.borrow_date;

-- Display moving average of books borrowed in the last 3 months
SELECT DATE_FORMAT(borrow_date, '%Y-%m') AS month,
       COUNT(*) AS monthly_borrows,
       AVG(COUNT(*)) OVER (ORDER BY DATE_FORMAT(borrow_date, '%Y-%m') 
                          ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS moving_avg
FROM Transactions
GROUP BY DATE_FORMAT(borrow_date, '%Y-%m')
ORDER BY month;

-- =====================================================
-- 14. CASE EXPRESSIONS
-- =====================================================

-- Assign Membership_Status based on borrowing activity
SELECT m.member_id, m.name,
       CASE 
           WHEN EXISTS (
               SELECT 1 FROM Transactions t 
               WHERE t.member_id = m.member_id 
               AND t.borrow_date >= DATE_SUB(CURDATE(), INTERVAL 6 MONTH)
           ) THEN 'Active'
           ELSE 'Inactive'
       END AS Membership_Status
FROM Members m;

-- Categorize books based on publication date
SELECT title, published_date,
       CASE 
           WHEN published_date >= '2020-01-01' THEN 'New Arrival'
           WHEN published_date < '2000-01-01' THEN 'Classic'
           ELSE 'Regular'
       END AS Book_Category
FROM Books;

-- =====================================================
-- 15. ADVANCED QUERIES
-- =====================================================

-- Find members with overdue books (borrowed more than 14 days ago, not returned)
SELECT m.name, b.title, t.borrow_date,
       DATEDIFF(CURDATE(), t.borrow_date) AS days_overdue
FROM Transactions t
JOIN Members m ON t.member_id = m.member_id
JOIN Books b ON t.book_id = b.book_id
WHERE t.return_date IS NULL 
AND DATEDIFF(CURDATE(), t.borrow_date) > 14
ORDER BY days_overdue DESC;

-- Get the top 3 most active members (most books borrowed)
SELECT m.name, COUNT(t.transaction_id) AS total_borrowed
FROM Members m
JOIN Transactions t ON m.member_id = t.member_id
GROUP BY m.member_id, m.name
ORDER BY total_borrowed DESC
LIMIT 3;

-- List all Science books with their availability status
SELECT b.title, b.price, 
       CASE 
           WHEN b.available_copies > 0 THEN 'Available'
           ELSE 'Not Available'
       END AS availability_status
FROM Books b
WHERE b.category = 'Science'
ORDER BY b.title;

-- =====================================================
-- END OF SCRIPT
-- =====================================================
