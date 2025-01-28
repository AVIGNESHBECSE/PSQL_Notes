Normalization in SQL is the process of organizing data in a database to minimize redundancy and dependency by dividing large tables into smaller ones and defining relationships among them. It helps improve data integrity and efficiency.

Normalization typically involves several "normal forms" (NF), each with specific rules. Here's a breakdown of the first few normal forms:

1. First Normal Form (1NF)
All columns must contain atomic (indivisible) values.
Each record must have a unique identifier (Primary Key).
2. Second Normal Form (2NF)
The table must be in 1NF.
All non-key attributes must be fully dependent on the primary key (i.e., no partial dependency).
3. Third Normal Form (3NF)
The table must be in 2NF.
No transitive dependency (non-key attributes should not depend on other non-key attributes).
Example:
Consider a table with customer orders before normalization:

Before Normalization (Unnormalized Table):
OrderID	CustomerName	Product	Quantity	OrderDate	CustomerPhone
1	John Smith	Laptop	1	2025-01-01	123-4567
2	Jane Doe	Phone	2	2025-01-02	234-5678
3	John Smith	Mouse	1	2025-01-03	123-4567
Here, you can see redundant data like CustomerName and CustomerPhone being repeated for every order from the same customer.

Step 1: Convert to 1NF (Eliminate Repeating Groups)
Each column should hold atomic values, and we need a unique identifier (e.g., OrderID):

OrderID	CustomerName	Product	Quantity	OrderDate	CustomerPhone
1	John Smith	Laptop	1	2025-01-01	123-4567
2	Jane Doe	Phone	2	2025-01-02	234-5678
3	John Smith	Mouse	1	2025-01-03	123-4567
This is now in 1NF because there are no repeating groups, and each field contains atomic values.

Step 2: Convert to 2NF (Remove Partial Dependencies)
We observe that CustomerName and CustomerPhone depend only on CustomerID (not OrderID). We should split the data into two tables: one for orders and one for customers.

Customers Table:

CustomerID	CustomerName	CustomerPhone
1	John Smith	123-4567
2	Jane Doe	234-5678
Orders Table:

OrderID	CustomerID	Product	Quantity	OrderDate
1	1	Laptop	1	2025-01-01
2	2	Phone	2	2025-01-02
3	1	Mouse	1	2025-01-03
Now, the Orders table only stores the CustomerID, and the CustomerName and CustomerPhone are in the Customers table. The Orders table is in 2NF because there are no partial dependencies.

Step 3: Convert to 3NF (Remove Transitive Dependencies)
In this case, there are no transitive dependencies. However, if we had a table where non-key attributes depended on other non-key attributes, we would split those into a new table as well.

For instance, imagine we had a column CustomerRegion in the Customers table that depends on CustomerPhone (a non-key attribute). We would split the CustomerRegion into another table linked by CustomerPhone.

Final Result (After Normalization):
Customers Table:

CustomerID	CustomerName	CustomerPhone	CustomerRegion
1	John Smith	123-4567	East
2	Jane Doe	234-5678	West
Orders Table:

OrderID	CustomerID	Product	Quantity	OrderDate
1	1	Laptop	1	2025-01-01
2	2	Phone	2	2025-01-02
3	1	Mouse	1	2025-01-03
Now, we have eliminated redundancy and ensured the data is normalized to 3NF.

SQL Code Example:
sql
Copy
-- Create Customers table
CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    CustomerName VARCHAR(100),
    CustomerPhone VARCHAR(15)
);

-- Insert data into Customers table
INSERT INTO Customers (CustomerID, CustomerName, CustomerPhone) VALUES
(1, 'John Smith', '123-4567'),
(2, 'Jane Doe', '234-5678');

-- Create Orders table
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    Product VARCHAR(100),
    Quantity INT,
    OrderDate DATE,
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

-- Insert data into Orders table
INSERT INTO Orders (OrderID, CustomerID, Product, Quantity, OrderDate) VALUES
(1, 1, 'Laptop', 1, '2025-01-01'),
(2, 2, 'Phone', 2, '2025-01-02'),
(3, 1, 'Mouse', 1, '2025-01-03');
This code will normalize the original data into two tables, Customers and Orders, with proper relationships.







