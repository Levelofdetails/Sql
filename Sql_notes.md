
# 🧱 **1. Add Sample Data (Realistic & Useful for Analytics)**

## Customers
```sql
INSERT INTO Customers (FirstName, LastName, Email, Phone)
VALUES
('Sophia', 'Williams', 'sophia@example.com', '1112223333'),
('Liam', 'Johnson', 'liam@example.com', '2223334444'),
('Emma', 'Davis', 'emma@example.com', '3334445555'),
('Noah', 'Miller', 'noah@example.com', '4445556666');
```

## Products
```sql
INSERT INTO Products (ProductName, Category, UnitPrice, Stock)
VALUES
('Keyboard', 'Electronics', 45, 50),
('Monitor', 'Electronics', 300, 15),
('Office Desk', 'Furniture', 250, 10),
('Headphones', 'Electronics', 80, 40);
```

## Orders
```sql
INSERT INTO Orders (CustomerID, Status)
VALUES
(3, 'Completed'),
(4, 'Completed'),
(5, 'Pending'),
(6, 'Completed');
```

## OrderItems
```sql
INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)
VALUES
(3, 4, 1, 150),
(3, 5, 1, 45),
(4, 6, 1, 300),
(4, 7, 1, 250),
(5, 8, 2, 80),
(6, 1, 1, 1200);
```

## Payments
```sql
INSERT INTO Payments (OrderID, Amount, PaymentMethod)
VALUES
(3, 195, 'UPI'),
(4, 550, 'Credit Card'),
(6, 1200, 'Cash');
```

---

# 🎯 **2. 20 SQL Analytics Use Cases (with Solutions)**  
These cover **ALL major SQL topics** used in interviews.

---

# ⭐ **1. Total revenue generated**
```sql
SELECT SUM(Quantity * UnitPrice) AS TotalRevenue
FROM OrderItems;
```

---

# ⭐ **2. Revenue by product**
```sql
SELECT p.ProductName, SUM(oi.Quantity * oi.UnitPrice) AS Revenue
FROM OrderItems oi
JOIN Products p ON oi.ProductID = p.ProductID
GROUP BY p.ProductName
ORDER BY Revenue DESC;
```

---

# ⭐ **3. Top 5 customers by spending**
```sql
SELECT c.FirstName, c.LastName, SUM(oi.Quantity * oi.UnitPrice) AS TotalSpent
FROM Customers c
JOIN Orders o ON c.CustomerID = o.CustomerID
JOIN OrderItems oi ON o.OrderID = oi.OrderID
GROUP BY c.FirstName, c.LastName
ORDER BY TotalSpent DESC
OFFSET 0 ROWS FETCH NEXT 5 ROWS ONLY;
```

---

# ⭐ **4. Orders without payments (LEFT JOIN)**
```sql
SELECT o.OrderID, o.CustomerID, o.Status
FROM Orders o
LEFT JOIN Payments p ON o.OrderID = p.OrderID
WHERE p.PaymentID IS NULL;
```

---

# ⭐ **5. Count of orders by status**
```sql
SELECT Status, COUNT(*) AS TotalOrders
FROM Orders
GROUP BY Status;
```

---

# ⭐ **6. Products with low stock (<20)**
```sql
SELECT ProductName, Stock
FROM Products
WHERE Stock < 20;
```

---

# ⭐ **7. Monthly sales trend (DATEPART)**
```sql
SELECT 
    YEAR(OrderDate) AS Year,
    MONTH(OrderDate) AS Month,
    SUM(oi.Quantity * oi.UnitPrice) AS Revenue
FROM Orders o
JOIN OrderItems oi ON o.OrderID = oi.OrderID
GROUP BY YEAR(OrderDate), MONTH(OrderDate)
ORDER BY Year, Month;
```

---

# ⭐ **8. Category‑wise revenue**
```sql
SELECT p.Category, SUM(oi.Quantity * oi.UnitPrice) AS Revenue
FROM OrderItems oi
JOIN Products p ON oi.ProductID = p.ProductID
GROUP BY p.Category;
```

---

# ⭐ **9. Most frequently ordered product**
```sql
SELECT TOP 1 p.ProductName, SUM(oi.Quantity) AS TotalQty
FROM OrderItems oi
JOIN Products p ON oi.ProductID = p.ProductID
GROUP BY p.ProductName
ORDER BY TotalQty DESC;
```

---

# ⭐ **10. Customers who never placed an order**
```sql
SELECT *
FROM Customers
WHERE CustomerID NOT IN (SELECT CustomerID FROM Orders);
```

---

# ⭐ **11. Total number of items sold**
```sql
SELECT SUM(Quantity) AS TotalItemsSold
FROM OrderItems;
```

---

# ⭐ **12. Average order value**
```sql
SELECT AVG(OrderTotal) AS AvgOrderValue
FROM (
    SELECT o.OrderID, SUM(oi.Quantity * oi.UnitPrice) AS OrderTotal
    FROM Orders o
    JOIN OrderItems oi ON o.OrderID = oi.OrderID
    GROUP BY o.OrderID
) t;
```

---

# ⭐ **13. Window function: Rank customers by spending**
```sql
SELECT 
    c.CustomerID,
    c.FirstName,
    SUM(oi.Quantity * oi.UnitPrice) AS TotalSpent,
    RANK() OVER (ORDER BY SUM(oi.Quantity * oi.UnitPrice) DESC) AS SpendingRank
FROM Customers c
JOIN Orders o ON c.CustomerID = o.CustomerID
JOIN OrderItems oi ON o.OrderID = oi.OrderID
GROUP BY c.CustomerID, c.FirstName;
```

---

# ⭐ **14. CTE: Find orders above average value**
```sql
WITH AvgOrder AS (
    SELECT AVG(Quantity * UnitPrice) AS AvgValue
    FROM OrderItems
)
SELECT o.OrderID, SUM(oi.Quantity * oi.UnitPrice) AS OrderValue
FROM Orders o
JOIN OrderItems oi ON o.OrderID = oi.OrderID
GROUP BY o.OrderID
HAVING SUM(oi.Quantity * oi.UnitPrice) > (SELECT AvgValue FROM AvgOrder);
```

---

# ⭐ **15. Subquery: Most expensive product**
```sql
SELECT *
FROM Products
WHERE UnitPrice = (SELECT MAX(UnitPrice) FROM Products);
```

---

# ⭐ **16. Grouping Sets: Revenue by category and overall**
```sql
SELECT 
    p.Category,
    SUM(oi.Quantity * oi.UnitPrice) AS Revenue
FROM OrderItems oi
JOIN Products p ON oi.ProductID = p.ProductID
GROUP BY GROUPING SETS ((p.Category), ());
```

---

# ⭐ **17. Find customers who bought Electronics**
```sql
SELECT DISTINCT c.CustomerID, c.FirstName
FROM Customers c
JOIN Orders o ON c.CustomerID = o.CustomerID
JOIN OrderItems oi ON o.OrderID = oi.OrderID
JOIN Products p ON oi.ProductID = p.ProductID
WHERE p.Category = 'Electronics';
```

---

# ⭐ **18. Payment method distribution**
```sql
SELECT PaymentMethod, COUNT(*) AS TotalPayments
FROM Payments
GROUP BY PaymentMethod;
```

---

# ⭐ **19. Total revenue per order**
```sql
SELECT o.OrderID, SUM(oi.Quantity * oi.UnitPrice) AS OrderRevenue
FROM Orders o
JOIN OrderItems oi ON o.OrderID = oi.OrderID
GROUP BY o.OrderID;
```

---

# ⭐ **20. Inventory impact: Stock after sales**
```sql
SELECT 
    p.ProductName,
    p.Stock - COALESCE(SUM(oi.Quantity), 0) AS RemainingStock
FROM Products p
LEFT JOIN OrderItems oi ON p.ProductID = oi.ProductID
GROUP BY p.ProductName, p.Stock;
```

---

# 🎓 **SQL Topics Covered**
This set covers:

- Joins (INNER, LEFT, NOT IN)  
- Aggregations  
- Window functions  
- CTEs  
- Subqueries  
- Grouping sets  
- Ranking  
- Date functions  
- Filtering  
- Business analytics  

---

# 💼 **Interview‑Style Questions (with answers)**

### **1. Difference between WHERE and HAVING?**
- WHERE filters rows **before** grouping  
- HAVING filters groups **after** aggregation  

### **2. What is a window function?**
A function that performs calculations across a set of rows related to the current row.

### **3. Difference between UNION and UNION ALL?**
- UNION removes duplicates  
- UNION ALL keeps duplicates  

### **4. What is normalization?**
Process of reducing redundancy and improving data integrity.

### **5. What is a CTE?**
A temporary result set used within a query.

### **6. What is the difference between RANK, DENSE_RANK, ROW_NUMBER?**
- RANK: gaps  
- DENSE_RANK: no gaps  
- ROW_NUMBER: unique sequence  

### **7. What is ACID?**
Atomicity, Consistency, Isolation, Durability.

### **8. What is a primary key?**
Uniquely identifies a row.

### **9. What is a foreign key?**
Enforces referential integrity.

### **10. What is an index?**
Improves read performance.

---

RetailDB ER Diagram (Text‑Based + Explanation)
Below is a clear, normalized ER diagram representation.

┌──────────────┐        ┌──────────────┐
│  Customers    │        │   Products   │
├──────────────┤        ├──────────────┤
│ CustomerID PK │        │ ProductID PK │
│ FirstName     │        │ ProductName  │
│ LastName      │        │ Category     │
│ Email         │        │ UnitPrice    │
│ Phone         │        │ Stock        │
└───────┬──────┘        └───────┬──────┘
        │                         │
        │                         │
┌───────▼────────┐      ┌────────▼────────┐
│     Orders      │      │   OrderItems    │
├─────────────────┤      ├─────────────────┤
│ OrderID PK      │◄────►│ OrderID FK      │
│ CustomerID FK   │      │ ProductID FK     │
│ OrderDate       │      │ Quantity         │
│ Status          │      │ UnitPrice        │
└───────┬────────┘      └────────┬────────┘
        │                         │
        │                         │
        │                         │
┌───────▼────────┐
│    Payments     │
├─────────────────┤
│ PaymentID PK    │
│ OrderID FK      │
│ Amount          │
│ PaymentDate     │
│ PaymentMethod   │
└─────────────────┘

Key Relationships
Customers 1‑to‑Many Orders

Orders 1‑to‑Many OrderItems

Products 1‑to‑Many OrderItems

Orders 1‑to‑Many Payments

This is a classic OLTP retail schema.

---

# 🎯 **2. 20 Advanced SQL Questions (with Answers)**  
These cover **window functions, CTEs, subqueries, performance, design, analytics, and joins**.

---

## ⭐ **1. Find the top 3 products by revenue per category**
```sql
SELECT *
FROM (
    SELECT 
        p.Category,
        p.ProductName,
        SUM(oi.Quantity * oi.UnitPrice) AS Revenue,
        RANK() OVER (PARTITION BY p.Category ORDER BY SUM(oi.Quantity * oi.UnitPrice) DESC) AS rnk
    FROM OrderItems oi
    JOIN Products p ON oi.ProductID = p.ProductID
    GROUP BY p.Category, p.ProductName
) t
WHERE rnk <= 3;
```

---

## ⭐ **2. Find customers who placed orders in consecutive months**
```sql
WITH MonthlyOrders AS (
    SELECT 
        CustomerID,
        FORMAT(OrderDate, 'yyyyMM') AS YearMonth,
        ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY FORMAT(OrderDate, 'yyyyMM')) AS rn
    FROM Orders
)
SELECT a.CustomerID
FROM MonthlyOrders a
JOIN MonthlyOrders b
  ON a.CustomerID = b.CustomerID
 AND a.rn + 1 = b.rn
 AND a.YearMonth + 1 = b.YearMonth;
```

---

## ⭐ **3. Find orders where payment amount does NOT match order total**
```sql
SELECT o.OrderID,
       SUM(oi.Quantity * oi.UnitPrice) AS OrderTotal,
       SUM(p.Amount) AS PaidAmount
FROM Orders o
JOIN OrderItems oi ON o.OrderID = oi.OrderID
LEFT JOIN Payments p ON o.OrderID = p.OrderID
GROUP BY o.OrderID
HAVING SUM(oi.Quantity * oi.UnitPrice) <> SUM(p.Amount);
```

---

## ⭐ **4. Running total of revenue by date**
```sql
SELECT 
    OrderDate,
    DailyRevenue,
    SUM(DailyRevenue) OVER (ORDER BY OrderDate) AS RunningRevenue
FROM (
    SELECT 
        CAST(OrderDate AS DATE) AS OrderDate,
        SUM(oi.Quantity * oi.UnitPrice) AS DailyRevenue
    FROM Orders o
    JOIN OrderItems oi ON o.OrderID = oi.OrderID
    GROUP BY CAST(OrderDate AS DATE)
) t;
```

---

## ⭐ **5. Find customers who bought ALL products in a category**
```sql
SELECT CustomerID
FROM Orders o
JOIN OrderItems oi ON o.OrderID = oi.OrderID
JOIN Products p ON oi.ProductID = p.ProductID
WHERE p.Category = 'Electronics'
GROUP BY CustomerID
HAVING COUNT(DISTINCT p.ProductID) = (
    SELECT COUNT(*) FROM Products WHERE Category = 'Electronics'
);
```

---

## ⭐ **6. Detect duplicate customers by email or phone**
```sql
SELECT Email, COUNT(*) AS Cnt
FROM Customers
GROUP BY Email
HAVING COUNT(*) > 1;
```

---

## ⭐ **7. Find orders with more than 3 items**
```sql
SELECT OrderID, COUNT(*) AS ItemCount
FROM OrderItems
GROUP BY OrderID
HAVING COUNT(*) > 3;
```

---

## ⭐ **8. Find the most profitable category**
```sql
SELECT TOP 1 Category, SUM(oi.Quantity * oi.UnitPrice) AS Revenue
FROM OrderItems oi
JOIN Products p ON oi.ProductID = p.ProductID
GROUP BY Category
ORDER BY Revenue DESC;
```

---

## ⭐ **9. Find customers who placed orders but never made payments**
```sql
SELECT DISTINCT c.CustomerID, c.FirstName
FROM Customers c
JOIN Orders o ON c.CustomerID = o.CustomerID
LEFT JOIN Payments p ON o.OrderID = p.OrderID
WHERE p.PaymentID IS NULL;
```

---

## ⭐ **10. Find the average revenue per customer**
```sql
SELECT AVG(CustomerRevenue) AS AvgRevenue
FROM (
    SELECT c.CustomerID, SUM(oi.Quantity * oi.UnitPrice) AS CustomerRevenue
    FROM Customers c
    JOIN Orders o ON c.CustomerID = o.CustomerID
    JOIN OrderItems oi ON o.OrderID = oi.OrderID
    GROUP BY c.CustomerID
) t;
```

---

## ⭐ **11. Find the most recent order for each customer**
```sql
SELECT *
FROM (
    SELECT 
        o.*,
        ROW_NUMBER() OVER (PARTITION BY CustomerID ORDER BY OrderDate DESC) AS rn
    FROM Orders o
) t
WHERE rn = 1;
```

---

## ⭐ **12. Find products that have never been ordered**
```sql
SELECT *
FROM Products
WHERE ProductID NOT IN (SELECT ProductID FROM OrderItems);
```

---

## ⭐ **13. Find customers who spent above the 75th percentile**
```sql
WITH CustSpend AS (
    SELECT c.CustomerID, SUM(oi.Quantity * oi.UnitPrice) AS TotalSpent
    FROM Customers c
    JOIN Orders o ON c.CustomerID = o.CustomerID
    JOIN OrderItems oi ON o.OrderID = oi.OrderID
    GROUP BY c.CustomerID
)
SELECT *
FROM CustSpend
WHERE TotalSpent > (
    SELECT PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY TotalSpent)
    FROM CustSpend
);
```

---

## ⭐ **14. Find the order with the highest number of distinct products**
```sql
SELECT TOP 1 OrderID, COUNT(DISTINCT ProductID) AS DistinctProducts
FROM OrderItems
GROUP BY OrderID
ORDER BY DistinctProducts DESC;
```

---

## ⭐ **15. Find the most common payment method**
```sql
SELECT TOP 1 PaymentMethod, COUNT(*) AS Cnt
FROM Payments
GROUP BY PaymentMethod
ORDER BY Cnt DESC;
```

---

## ⭐ **16. Find the average number of items per order**
```sql
SELECT AVG(ItemCount)
FROM (
    SELECT OrderID, SUM(Quantity) AS ItemCount
    FROM OrderItems
    GROUP BY OrderID
) t;
```

---

## ⭐ **17. Find the customer with the longest gap between orders**
```sql
WITH Gaps AS (
    SELECT 
        CustomerID,
        OrderDate,
        LAG(OrderDate) OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS PrevOrder
    FROM Orders
)
SELECT TOP 1 CustomerID, DATEDIFF(day, PrevOrder, OrderDate) AS GapDays
FROM Gaps
WHERE PrevOrder IS NOT NULL
ORDER BY GapDays DESC;
```

---

## ⭐ **18. Find orders where quantity > average quantity**
```sql
SELECT *
FROM OrderItems
WHERE Quantity > (SELECT AVG(Quantity) FROM OrderItems);
```

---

## ⭐ **19. Find the total revenue per category per month**
```sql
SELECT 
    p.Category,
    YEAR(o.OrderDate) AS Yr,
    MONTH(o.OrderDate) AS Mn,
    SUM(oi.Quantity * oi.UnitPrice) AS Revenue
FROM OrderItems oi
JOIN Orders o ON oi.OrderID = o.OrderID
JOIN Products p ON oi.ProductID = p.ProductID
GROUP BY p.Category, YEAR(o.OrderDate), MONTH(o.OrderDate);
```

---

## ⭐ **20. Find customers who placed orders in the last 30 days**
```sql
SELECT DISTINCT c.*
FROM Customers c
JOIN Orders o ON c.CustomerID = o.CustomerID
WHERE o.OrderDate >= DATEADD(day, -30, GETDATE());
```

---

# 🎯 **3. Stored Procedures (ETL + Reporting + Inserts)**

---

## ⭐ **A. Stored Procedure: Insert New Order (with validation)**

```sql
CREATE PROCEDURE usp_CreateOrder
    @CustomerID INT,
    @ProductID INT,
    @Quantity INT
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @UnitPrice DECIMAL(10,2);
    DECLARE @OrderID INT;

    -- Validate stock
    SELECT @UnitPrice = UnitPrice
    FROM Products
    WHERE ProductID = @ProductID AND Stock >= @Quantity;

    IF @UnitPrice IS NULL
    BEGIN
        RAISERROR('Insufficient stock or invalid product.', 16, 1);
        RETURN;
    END

    -- Create order
    INSERT INTO Orders (CustomerID, Status)
    VALUES (@CustomerID, 'Pending');

    SET @OrderID = SCOPE_IDENTITY();

    -- Insert order item
    INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)
    VALUES (@OrderID, @ProductID, @Quantity, @UnitPrice);

    -- Reduce stock
    UPDATE Products
    SET Stock = Stock - @Quantity
    WHERE ProductID = @ProductID;

    SELECT @OrderID AS NewOrderID;
END;
```

---

## ⭐ **B. Stored Procedure: Daily Revenue Report**

```sql
CREATE PROCEDURE usp_DailyRevenue
AS
BEGIN
    SELECT 
        CAST(o.OrderDate AS DATE) AS OrderDate,
        SUM(oi.Quantity * oi.UnitPrice) AS Revenue
    FROM Orders o
    JOIN OrderItems oi ON o.OrderID = oi.OrderID
    GROUP BY CAST(o.OrderDate AS DATE)
    ORDER BY OrderDate;
END;
```

---

# 🎯 **4. Triggers (Audit + Stock Control)**

---

## ⭐ **A. Trigger: Prevent Negative Stock**

```sql
CREATE TRIGGER trg_PreventNegativeStock
ON OrderItems
AFTER INSERT
AS
BEGIN
    IF EXISTS (
        SELECT 1
        FROM inserted i
        JOIN Products p ON i.ProductID = p.ProductID
        WHERE p.Stock - i.Quantity < 0
    )
    BEGIN
        ROLLBACK;
        RAISERROR('Stock cannot go negative.', 16, 1);
    END
END;
```

---

## ⭐ **B. Trigger: Auto‑update stock after order item insert**

```sql
CREATE TRIGGER trg_UpdateStock
ON OrderItems
AFTER INSERT
AS
BEGIN
    UPDATE p
    SET Stock = Stock - i.Quantity
    FROM Products p
    JOIN inserted i ON p.ProductID = i.ProductID;
END;
```

---

## ⭐ **C. Trigger: Audit Order Status Changes**

```sql
CREATE TABLE OrderAudit (
    AuditID INT IDENTITY PRIMARY KEY,
    OrderID INT,
    OldStatus VARCHAR(20),
    NewStatus VARCHAR(20),
    ChangedAt DATETIME DEFAULT GETDATE()
);
```

```sql
CREATE TRIGGER trg_OrderStatusAudit
ON Orders
AFTER UPDATE
AS
BEGIN
    INSERT INTO OrderAudit (OrderID, OldStatus, NewStatus)
    SELECT d.OrderID, d.Status, i.Status
    FROM deleted d
    JOIN inserted i ON d.OrderID = i.OrderID
    WHERE d.Status <> i.Status;
END;
```

---

# 🎯 **5. ETL Scripts (Staging → Validation → Core Load)**

---

## ⭐ **A. Create Staging Table**

```sql
CREATE TABLE Staging_Orders (
    StagingID INT IDENTITY PRIMARY KEY,
    CustomerID INT,
    ProductID INT,
    Quantity INT,
    UnitPrice DECIMAL(10,2),
    OrderDate DATETIME,
    IsValid BIT DEFAULT 1,
    ErrorMessage VARCHAR(255)
);
```

---

## ⭐ **B. Validation Script**

```sql
UPDATE Staging_Orders
SET IsValid = 0,
    ErrorMessage = 'Invalid quantity'
WHERE Quantity <= 0;

UPDATE Staging_Orders
SET IsValid = 0,
    ErrorMessage = 'Invalid product'
WHERE ProductID NOT IN (SELECT ProductID FROM Products);
```

---

## ⭐ **C. Load Valid Rows into Core Tables**

```sql
INSERT INTO Orders (CustomerID, OrderDate, Status)
SELECT CustomerID, OrderDate, 'Completed'
FROM Staging_Orders
WHERE IsValid = 1;
```

```sql
INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)
SELECT o.OrderID, s.ProductID, s.Quantity, s.UnitPrice
FROM Staging_Orders s
JOIN Orders o ON s.CustomerID = o.CustomerID AND s.OrderDate = o.OrderDate
WHERE s.IsValid = 1;
```

---


*******************************************************************************************

Here it is — a **complete, high‑impact SQL Interview Cheat Sheet** that covers everything you need to shine in interviews.  
This is the kind of resource senior engineers hand to new hires.  
Clear, structured, and packed with examples.

---

# 🎯 **SQL INTERVIEW CHEAT SHEET (Ultimate Edition)**  
Covers:  
✔ SQL Basics  
✔ Joins  
✔ Aggregations  
✔ Window Functions  
✔ CTEs  
✔ Subqueries  
✔ Set Operators  
✔ Normalization  
✔ Indexing  
✔ Transactions  
✔ Stored Procedures  
✔ Triggers  
✔ Performance Tuning  
✔ Advanced SQL Patterns  
✔ 40+ Interview Questions  

---

# 🧱 **1. SQL BASICS**

### **DDL – Data Definition Language**
```sql
CREATE TABLE, ALTER TABLE, DROP TABLE
```

### **DML – Data Manipulation Language**
```sql
SELECT, INSERT, UPDATE, DELETE
```

### **DCL – Data Control Language**
```sql
GRANT, REVOKE
```

### **TCL – Transaction Control Language**
```sql
COMMIT, ROLLBACK, SAVEPOINT
```

---

# 🔗 **2. JOINS (Most Asked Topic)**

### **INNER JOIN**
Returns matching rows.
```sql
SELECT * FROM A INNER JOIN B ON A.id = B.id;
```

### **LEFT JOIN**
Returns all rows from left, matching from right.
```sql
SELECT * FROM A LEFT JOIN B ON A.id = B.id;
```

### **RIGHT JOIN**
Opposite of LEFT JOIN.

### **FULL OUTER JOIN**
Returns all rows from both tables.

### **CROSS JOIN**
Cartesian product.

### **SELF JOIN**
Join table with itself.

---

# 📊 **3. AGGREGATIONS**

### Common functions:
```sql
SUM(), COUNT(), AVG(), MIN(), MAX()
```

### GROUP BY
```sql
SELECT Category, SUM(Revenue)
FROM Sales
GROUP BY Category;
```

### HAVING (filters after grouping)
```sql
HAVING SUM(Revenue) > 10000;
```

---

# 🪟 **4. WINDOW FUNCTIONS (Interview Gold)**

### Ranking
```sql
RANK() OVER (ORDER BY Revenue DESC)
```

### Row Number
```sql
ROW_NUMBER() OVER (PARTITION BY Category ORDER BY Revenue DESC)
```

### Running Total
```sql
SUM(Amount) OVER (ORDER BY Date)
```

### Moving Average
```sql
AVG(Sales) OVER (ORDER BY Date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)
```

---

# 🧩 **5. CTEs (Common Table Expressions)**

### Basic CTE
```sql
WITH SalesCTE AS (
    SELECT * FROM Sales WHERE Amount > 100
)
SELECT * FROM SalesCTE;
```

### Recursive CTE
```sql
WITH cte AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM cte WHERE n < 10
)
SELECT * FROM cte;
```

---

# 🔍 **6. SUBQUERIES**

### Scalar subquery
```sql
SELECT * FROM Products
WHERE Price > (SELECT AVG(Price) FROM Products);
```

### IN subquery
```sql
SELECT * FROM Customers
WHERE CustomerID IN (SELECT CustomerID FROM Orders);
```

### EXISTS
```sql
SELECT * FROM Customers c
WHERE EXISTS (SELECT 1 FROM Orders o WHERE o.CustomerID = c.CustomerID);
```

---

# 🔀 **7. SET OPERATORS**

### UNION (removes duplicates)
### UNION ALL (keeps duplicates)
### INTERSECT
### EXCEPT

---

# 🧱 **8. NORMALIZATION**

### 1NF  
Atomic values, no repeating groups.

### 2NF  
1NF + no partial dependency.

### 3NF  
2NF + no transitive dependency.

### BCNF  
Every determinant is a key.

---

# ⚡ **9. INDEXING**

### Clustered Index  
Sorts the table physically.

### Non‑Clustered Index  
Pointer to data.

### Covering Index  
Contains all columns needed for a query.

### Composite Index  
Index on multiple columns.

---

# 🔐 **10. TRANSACTIONS**

```sql
BEGIN TRAN
UPDATE ...
IF @@ERROR <> 0 ROLLBACK
ELSE COMMIT
```

---

# ⚙️ **11. STORED PROCEDURES**

### Example
```sql
CREATE PROCEDURE usp_GetCustomerOrders @CustomerID INT
AS
BEGIN
    SELECT * FROM Orders WHERE CustomerID = @CustomerID;
END;
```

---

# 🔥 **12. TRIGGERS**

### After Insert Trigger
```sql
CREATE TRIGGER trg_AuditOrders
ON Orders
AFTER INSERT
AS
INSERT INTO AuditTable SELECT * FROM inserted;
```

---

# 🚀 **13. PERFORMANCE TUNING**

### Tips:
- Avoid SELECT *  
- Use proper indexes  
- Avoid functions on indexed columns  
- Use EXISTS instead of IN  
- Avoid correlated subqueries  
- Use window functions instead of self‑joins  
- Analyze execution plan  

---

# 🧠 **14. ADVANCED SQL PATTERNS**

### Pivot
```sql
SELECT * FROM Sales
PIVOT (SUM(Amount) FOR Month IN ([Jan],[Feb],[Mar])) AS p;
```

### Unpivot
```sql
UNPIVOT (Amount FOR Month IN ([Jan],[Feb],[Mar])) AS u;
```

### Grouping Sets
```sql
GROUP BY GROUPING SETS ((Category), (Year), ())
```

### Dynamic SQL
```sql
EXEC('SELECT * FROM ' + @TableName);
```

---

# 🎓 **15. 40 SQL INTERVIEW QUESTIONS (With Answers)**

### **1. Difference between WHERE and HAVING?**  
WHERE filters rows before grouping; HAVING filters groups after aggregation.

### **2. What is the difference between RANK, DENSE_RANK, ROW_NUMBER?**  
- RANK: gaps  
- DENSE_RANK: no gaps  
- ROW_NUMBER: unique sequence  

### **3. What is a correlated subquery?**  
A subquery that depends on the outer query.

### **4. What is ACID?**  
Atomicity, Consistency, Isolation, Durability.

### **5. What is a composite key?**  
A key made of multiple columns.

### **6. What is the difference between DELETE and TRUNCATE?**  
TRUNCATE is faster, resets identity, cannot use WHERE.

### **7. What is a view?**  
A virtual table.

### **8. What is a materialized view?**  
A physically stored view.

### **9. What is the difference between UNION and UNION ALL?**  
UNION removes duplicates.

### **10. What is a deadlock?**  
Two transactions waiting on each other.

### **11. What is an execution plan?**  
Shows how SQL Server executes a query.

### **12. What is cardinality?**  
Number of distinct values.

### **13. What is referential integrity?**  
Foreign key constraints.

### **14. What is a surrogate key?**  
Artificial key (identity).

### **15. What is denormalization?**  
Adding redundancy for performance.

### **16. What is a clustered index?**  
Defines physical order of data.

### **17. What is a non‑clustered index?**  
Separate structure pointing to data.

### **18. What is a covering index?**  
Index containing all needed columns.

### **19. What is a CTE?**  
Temporary named result set.

### **20. What is a recursive CTE?**  
CTE that references itself.

### **21. What is a window function?**  
Function that operates over a window of rows.

### **22. What is a pivot table?**  
Rotates rows into columns.

### **23. What is a transaction?**  
Logical unit of work.

### **24. What is isolation level?**  
Controls concurrency behavior.

### **25. What is dirty read?**  
Reading uncommitted data.

### **26. What is phantom read?**  
New rows appear during transaction.

### **27. What is repeatable read?**  
Prevents changes to rows read.

### **28. What is optimistic locking?**  
Assumes no conflicts.

### **29. What is pessimistic locking?**  
Locks rows immediately.

### **30. What is a stored procedure?**  
Reusable SQL code block.

### **31. What is a trigger?**  
Code executed automatically on DML.

### **32. What is a function?**  
Returns a value; cannot modify data.

### **33. What is a cursor?**  
Row‑by‑row processing.

### **34. What is a temp table?**  
Temporary storage (#table).

### **35. What is a table variable?**  
In‑memory table (@table).

### **36. What is normalization?**  
Organizing data to reduce redundancy.

### **37. What is BCNF?**  
Stronger form of 3NF.

### **38. What is a surrogate key?**  
Artificial primary key.

### **39. What is a natural key?**  
Real‑world unique attribute.

### **40. What is sharding?**  
Horizontal partitioning.

---

