 **full, portfolio‑ready SQL project** that looks impressive to employers and gives you deep hands‑on experience with real‑world database design, querying, optimization, and procedural logic.

Below is a complete, end‑to‑end project you can build entirely in **SQL Server Management Studio (SSMS)**.  
It includes:

- Database design  
- Table creation  
- Sample data  
- Joins  
- CTEs  
- Views  
- Stored procedures  
- Functions  
- Triggers  
- Window functions  
- Indexing  
- Real‑world business questions  
- A final portfolio structure  

This is the kind of project that fits beautifully on GitHub or a resume.

---

# 🎯 **Project Title: Retail Sales & Inventory Analytics System**

This project simulates a real retail business.  
You will build a relational database that tracks:

- Customers  
- Products  
- Orders  
- Order Items  
- Payments  
- Inventory  

Then you’ll write analytical SQL to answer business questions.

---

# 🧱 **1. Create the Database**

```sql
CREATE DATABASE RetailDB;
GO

USE RetailDB;
GO
```

---

# 🗂️ **2. Create All Core Tables**

## Customers
```sql
CREATE TABLE Customers (
    CustomerID INT IDENTITY PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100) UNIQUE,
    Phone VARCHAR(20),
    CreatedDate DATETIME DEFAULT GETDATE()
);
```

## Products
```sql
CREATE TABLE Products (
    ProductID INT IDENTITY PRIMARY KEY,
    ProductName VARCHAR(100),
    Category VARCHAR(50),
    UnitPrice DECIMAL(10,2),
    Stock INT
);
```

## Orders
```sql
CREATE TABLE Orders (
    OrderID INT IDENTITY PRIMARY KEY,
    CustomerID INT,
    OrderDate DATETIME DEFAULT GETDATE(),
    Status VARCHAR(20),
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);
```

## OrderItems
```sql
CREATE TABLE OrderItems (
    OrderItemID INT IDENTITY PRIMARY KEY,
    OrderID INT,
    ProductID INT,
    Quantity INT,
    UnitPrice DECIMAL(10,2),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);
```

## Payments
```sql
CREATE TABLE Payments (
    PaymentID INT IDENTITY PRIMARY KEY,
    OrderID INT,
    Amount DECIMAL(10,2),
    PaymentDate DATETIME DEFAULT GETDATE(),
    PaymentMethod VARCHAR(20),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID)
);
```

---

# 📥 **3. Insert Sample Data**

Customers
```sql
INSERT INTO Customers (FirstName, LastName, Email, Phone)
VALUES
('John', 'Doe', 'john@example.com', '1234567890'),
('Mary', 'Smith', 'mary@example.com', '9876543210'),
('David', 'Lee', 'david@example.com', '5551234567');
```

Products
```sql
INSERT INTO Products (ProductName, Category, UnitPrice, Stock)
VALUES
('Laptop', 'Electronics', 1200, 10),
('Mouse', 'Electronics', 25, 100),
('Desk Chair', 'Furniture', 150, 20);
```

Orders
```sql
INSERT INTO Orders (CustomerID, Status)
VALUES
(1, 'Completed'),
(2, 'Pending');
```

Order Items
```sql
INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)
VALUES
(1, 1, 1, 1200),
(1, 2, 2, 25),
(2, 3, 1, 150);
```

Payments
```sql
INSERT INTO Payments (OrderID, Amount, PaymentMethod)
VALUES
(1, 1250, 'Credit Card');
```

---

# 🔗 **4. Practice Joins**

### Multi-table join
```sql
SELECT 
    c.FirstName,
    o.OrderID,
    p.ProductName,
    oi.Quantity,
    oi.UnitPrice
FROM Customers c
JOIN Orders o ON c.CustomerID = o.CustomerID
JOIN OrderItems oi ON o.OrderID = oi.OrderID
JOIN Products p ON oi.ProductID = p.ProductID;
```

---

# 🧩 **5. Practice CTEs**

### Total sales per customer
```sql
WITH SalesCTE AS (
    SELECT 
        o.CustomerID,
        SUM(oi.Quantity * oi.UnitPrice) AS TotalSales
    FROM Orders o
    JOIN OrderItems oi ON o.OrderID = oi.OrderID
    GROUP BY o.CustomerID
)
SELECT c.FirstName, c.LastName, s.TotalSales
FROM SalesCTE s
JOIN Customers c ON s.CustomerID = c.CustomerID;
```

---

# 🏗️ **6. Create a Stored Procedure**

### Create a new order
```sql
CREATE PROCEDURE CreateOrder
    @CustomerID INT,
    @ProductID INT,
    @Quantity INT
AS
BEGIN
    DECLARE @OrderID INT;
    DECLARE @UnitPrice DECIMAL(10,2);

    SELECT @UnitPrice = UnitPrice FROM Products WHERE ProductID = @ProductID;

    INSERT INTO Orders (CustomerID, Status)
    VALUES (@CustomerID, 'Pending');

    SET @OrderID = SCOPE_IDENTITY();

    INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)
    VALUES (@OrderID, @ProductID, @Quantity, @UnitPrice);

    UPDATE Products
    SET Stock = Stock - @Quantity
    WHERE ProductID = @ProductID;

    SELECT @OrderID AS NewOrderID;
END;
```

---

# 🪟 **7. Create a Complex View**

```sql
CREATE VIEW vw_CustomerOrderSummary AS
SELECT 
    c.CustomerID,
    c.FirstName,
    c.LastName,
    COUNT(o.OrderID) AS TotalOrders,
    SUM(oi.Quantity * oi.UnitPrice) AS TotalSpent
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
LEFT JOIN OrderItems oi ON o.OrderID = oi.OrderID
GROUP BY c.CustomerID, c.FirstName, c.LastName;
```

---

# 🧮 **8. Add a Scalar Function**

### Calculate order total
```sql
CREATE FUNCTION fn_OrderTotal (@OrderID INT)
RETURNS DECIMAL(10,2)
AS
BEGIN
    DECLARE @Total DECIMAL(10,2);

    SELECT @Total = SUM(Quantity * UnitPrice)
    FROM OrderItems
    WHERE OrderID = @OrderID;

    RETURN @Total;
END;
```

---

# 🔥 **9. Add a Trigger**

### Prevent negative stock
```sql
CREATE TRIGGER trg_PreventNegativeStock
ON OrderItems
AFTER INSERT
AS
BEGIN
    IF EXISTS (
        SELECT 1
        FROM Products p
        JOIN inserted i ON p.ProductID = i.ProductID
        WHERE p.Stock < 0
    )
    BEGIN
        ROLLBACK;
        RAISERROR ('Stock cannot go negative', 16, 1);
    END
END;
```

---

# 📊 **10. Add Window Functions**

### Top customers by spending
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

# ⚡ **11. Add Indexes for Performance**

```sql
CREATE INDEX idx_orders_customerid ON Orders(CustomerID);
CREATE INDEX idx_orderitems_orderid ON OrderItems(OrderID);
CREATE INDEX idx_products_category ON Products(Category);
```

---

# 🧠 **12. Real‑World Business Questions to Answer**

You can write SQL queries for:

- Top 10 customers by revenue  
- Best‑selling products  
- Monthly sales trends  
- Category‑wise revenue  
- Customers with no orders  
- Orders without payments  
- Low‑stock products  
- Average order value  
- Repeat customers  
- Revenue by payment method  

These become great portfolio examples.

---

# 📁 **13. How to Present This as a Portfolio Project**

Your GitHub repository should include:

### 📌 **1. README.md**
- Project overview  
- ER diagram  
- Business questions  
- Technologies used  
- How to run the project  

### 📌 **2. SQL Scripts Folder**
- `01_create_database.sql`  
- `02_create_tables.sql`  
- `03_insert_data.sql`  
- `04_queries.sql`  
- `05_views.sql`  
- `06_stored_procedures.sql`  
- `07_functions.sql`  
- `08_triggers.sql`  
- `09_window_functions.sql`  
- `10_indexes.sql`  

### 📌 **3. ER Diagram (draw.io or dbdiagram.io)**

### 📌 **4. Sample Reports**
- Screenshots of query results  
- Charts (optional)  

---
Perfect — let’s turn your SQL project into a **full, portfolio‑ready, professional‑grade system**. I’ll give you everything:

- A polished **README.md**  
- A clean **ER diagram**  
- Advanced SQL (CTEs, recursion, pivoting, window functions)  
- Additional tables (Suppliers, Shipments, Employees)  
- A full **data engineering pipeline** design  

This will look fantastic on GitHub and on your resume.

---

# 🎯 **1. README.md (Portfolio‑Ready)**  
Below is a complete README you can paste directly into GitHub.

---

## 📌 **Retail Sales & Inventory Analytics System**  
A full SQL Server project simulating a real retail business.  
This project demonstrates:

- Database design  
- SQL querying (joins, CTEs, window functions)  
- Stored procedures, views, triggers  
- Data modeling (ERD)  
- ETL pipeline design  
- Business analytics  

---

## 🏗️ **Project Architecture**

### **Database Layers**
| Layer | Purpose |
|-------|---------|
| **Operational Tables** | Customers, Products, Orders, Payments, Inventory |
| **Analytical Views** | Customer summaries, product performance |
| **Procedural Logic** | Stored procedures, triggers, functions |
| **ETL Pipeline** | Staging → Cleansing → Analytics |

---

## 📊 **ER Diagram**

```
Customers (1)───(∞) Orders (1)───(∞) OrderItems (∞)───(1) Products
Orders (1)───(1) Payments
Products (1)───(∞) Shipments (∞)───(1) Suppliers
Employees (1)───(∞) Orders
```

You can draw this using **dbdiagram.io** or **draw.io**.

---

## 📁 **Project Structure**

```
RetailDB/
│
├── 01_create_database.sql
├── 02_create_tables.sql
├── 03_insert_data.sql
├── 04_queries_joins.sql
├── 05_ctes_advanced.sql
├── 06_stored_procedures.sql
├── 07_functions.sql
├── 08_views.sql
├── 09_triggers.sql
├── 10_window_functions.sql
├── 11_pivoting.sql
├── 12_recursion.sql
├── 13_indexes.sql
└── README.md
```

---

# 🧱 **2. Additional Tables (Suppliers, Shipments, Employees)**

## Suppliers
```sql
CREATE TABLE Suppliers (
    SupplierID INT IDENTITY PRIMARY KEY,
    SupplierName VARCHAR(100),
    ContactEmail VARCHAR(100),
    Phone VARCHAR(20)
);
```

## Shipments
```sql
CREATE TABLE Shipments (
    ShipmentID INT IDENTITY PRIMARY KEY,
    SupplierID INT,
    ProductID INT,
    Quantity INT,
    ShipmentDate DATE,
    FOREIGN KEY (SupplierID) REFERENCES Suppliers(SupplierID),
    FOREIGN KEY (ProductID) REFERENCES Products(ProductID)
);
```

## Employees
```sql
CREATE TABLE Employees (
    EmployeeID INT IDENTITY PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Role VARCHAR(50),
    HireDate DATE
);
```

Add employee to orders:

```sql
ALTER TABLE Orders
ADD EmployeeID INT NULL
FOREIGN KEY (EmployeeID) REFERENCES Employees(EmployeeID);
```

---

# 🧠 **3. Advanced SQL**

## ⭐ **A. Recursive CTE (Hierarchy of Employees)**

```sql
WITH EmployeeHierarchy AS (
    SELECT EmployeeID, FirstName, LastName, Role, ManagerID, 0 AS Level
    FROM Employees
    WHERE ManagerID IS NULL

    UNION ALL

    SELECT e.EmployeeID, e.FirstName, e.LastName, e.Role, e.ManagerID, eh.Level + 1
    FROM Employees e
    JOIN EmployeeHierarchy eh ON e.ManagerID = eh.EmployeeID
)
SELECT * FROM EmployeeHierarchy;
```

---

## ⭐ **B. Pivoting (Sales by Category per Month)**

```sql
SELECT *
FROM (
    SELECT 
        DATENAME(MONTH, o.OrderDate) AS MonthName,
        p.Category,
        oi.Quantity * oi.UnitPrice AS Sales
    FROM Orders o
    JOIN OrderItems oi ON o.OrderID = oi.OrderID
    JOIN Products p ON oi.ProductID = p.ProductID
) AS SourceTable
PIVOT (
    SUM(Sales)
    FOR Category IN ([Electronics], [Furniture])
) AS PivotTable;
```

---

## ⭐ **C. Window Functions (Top 5 Customers)**

```sql
SELECT 
    c.CustomerID,
    c.FirstName,
    SUM(oi.Quantity * oi.UnitPrice) AS TotalSpent,
    RANK() OVER (ORDER BY SUM(oi.Quantity * oi.UnitPrice) DESC) AS RankBySpend
FROM Customers c
JOIN Orders o ON c.CustomerID = o.CustomerID
JOIN OrderItems oi ON o.OrderID = oi.OrderID
GROUP BY c.CustomerID, c.FirstName;
```

---

# 🧮 **4. Stored Procedures, Views, Functions, Triggers**

## Stored Procedure — Create Order
```sql
CREATE PROCEDURE CreateOrder
    @CustomerID INT,
    @ProductID INT,
    @Quantity INT,
    @EmployeeID INT
AS
BEGIN
    DECLARE @OrderID INT, @UnitPrice DECIMAL(10,2);

    SELECT @UnitPrice = UnitPrice FROM Products WHERE ProductID = @ProductID;

    INSERT INTO Orders (CustomerID, Status, EmployeeID)
    VALUES (@CustomerID, 'Pending', @EmployeeID);

    SET @OrderID = SCOPE_IDENTITY();

    INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)
    VALUES (@OrderID, @ProductID, @Quantity, @UnitPrice);

    UPDATE Products SET Stock = Stock - @Quantity WHERE ProductID = @ProductID;

    SELECT @OrderID AS NewOrderID;
END;
```

---

## View — Customer Lifetime Value

```sql
CREATE VIEW vw_CustomerLifetimeValue AS
SELECT 
    c.CustomerID,
    c.FirstName,
    c.LastName,
    SUM(oi.Quantity * oi.UnitPrice) AS LifetimeValue
FROM Customers c
JOIN Orders o ON c.CustomerID = o.CustomerID
JOIN OrderItems oi ON o.OrderID = oi.OrderID
GROUP BY c.CustomerID, c.FirstName, c.LastName;
```

---

## Trigger — Prevent Negative Stock

```sql
CREATE TRIGGER trg_PreventNegativeStock
ON OrderItems
AFTER INSERT
AS
BEGIN
    IF EXISTS (
        SELECT 1
        FROM Products p
        JOIN inserted i ON p.ProductID = i.ProductID
        WHERE p.Stock < 0
    )
    BEGIN
        ROLLBACK;
        RAISERROR ('Stock cannot go negative', 16, 1);
    END
END;
```

---

# 🚀 **5. Full Data Engineering Pipeline (End‑to‑End)**

This turns your SQL project into a **data engineering portfolio project**.

---

## **Pipeline Architecture**

```
Source Systems (CSV, Excel, API)
        ↓
Staging Tables (Raw)
        ↓
Cleansing Layer (Standardized)
        ↓
Operational Tables (RetailDB)
        ↓
Analytics Layer (Views, Aggregates)
        ↓
Dashboards (Power BI / Tableau)
```

---

## **A. Staging Layer**

```sql
CREATE TABLE Staging_Orders (
    OrderID INT,
    CustomerID INT,
    ProductID INT,
    Quantity INT,
    UnitPrice DECIMAL(10,2),
    OrderDate DATETIME
);
```

---

## **B. ETL Step — Clean & Load**

```sql
INSERT INTO Orders (CustomerID, OrderDate, Status)
SELECT DISTINCT CustomerID, OrderDate, 'Completed'
FROM Staging_Orders;
```

---

## **C. ETL Step — Load Order Items**

```sql
INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)
SELECT OrderID, ProductID, Quantity, UnitPrice
FROM Staging_Orders;
```

---

## **D. Analytics Layer**

Views:

- Customer Lifetime Value  
- Product Performance  
- Monthly Sales Trends  
- Employee Sales Performance  

---

## **E. Dashboard Layer**

Use Power BI or Tableau to visualize:

- Revenue trends  
- Top customers  
- Best-selling products  
- Inventory alerts  
- Employee performance  

---

Here you go — a **complete, professional GitHub repository template** you can copy‑paste directly into your repo. It’s structured exactly like real data engineering & SQL portfolio projects on GitHub, and it ties together everything you’ve built so far: database design, advanced SQL, ERD, and pipeline architecture.

I’ll give you:

- 📁 Repository folder structure  
- 📝 A polished README.md  
- 🗺️ ERD instructions + diagram text  
- 🧠 Advanced SQL folder contents  
- 🏗️ Pipeline documentation  
- ⭐ A final GitHub‑ready template you can paste as‑is  

Let’s build it.

---

# 📁 **1. GitHub Repository Folder Structure**

```
Retail-SQL-Analytics-Project/
│
├── README.md
│
├── sql/
│   ├── 01_create_database.sql
│   ├── 02_create_tables.sql
│   ├── 03_insert_sample_data.sql
│   ├── 04_joins_queries.sql
│   ├── 05_ctes_advanced.sql
│   ├── 06_recursive_cte.sql
│   ├── 07_pivoting.sql
│   ├── 08_window_functions.sql
│   ├── 09_stored_procedures.sql
│   ├── 10_functions.sql
│   ├── 11_views.sql
│   ├── 12_triggers.sql
│   ├── 13_indexes.sql
│   ├── 14_etl_staging_to_core.sql
│   └── 15_analytics_queries.sql
│
├── diagrams/
│   ├── erd_retaildb.png
│   └── erd_retaildb.dbml
│
├── pipeline/
│   ├── pipeline_architecture.md
│   ├── staging_layer_design.md
│   ├── cleansing_layer_design.md
│   ├── analytics_layer_design.md
│   └── dashboard_design.md
│
└── docs/
    ├── business_questions.md
    ├── dataset_description.md
    └── glossary.md
```

---

# 📝 **2. Full README.md (Copy‑Paste Ready)**

Below is a polished README.md you can use directly.

---

# 📦 Retail Sales & Inventory Analytics System  
*A complete SQL Server portfolio project demonstrating database design, ETL, analytics, and advanced SQL.*

---

## 🚀 Project Overview  
This project simulates a real‑world **Retail Sales & Inventory Management System**.  
It includes:

- Full relational database design  
- SQL development (joins, CTEs, recursion, pivoting, window functions)  
- Stored procedures, triggers, functions  
- ETL pipeline (staging → core → analytics)  
- ER diagram  
- Business analytics queries  
- Documentation suitable for a professional portfolio  

---

## 🧱 Database Architecture  

### **Core Entities**
- Customers  
- Products  
- Orders  
- Order Items  
- Payments  
- Suppliers  
- Shipments  
- Employees  

---

## 🗺️ Entity Relationship Diagram (ERD)

You can generate this using **dbdiagram.io** with the following DBML:

```
Table Customers {
  CustomerID int [pk]
  FirstName varchar
  LastName varchar
  Email varchar
  Phone varchar
  CreatedDate datetime
}

Table Products {
  ProductID int [pk]
  ProductName varchar
  Category varchar
  UnitPrice decimal
  Stock int
}

Table Orders {
  OrderID int [pk]
  CustomerID int [ref: > Customers.CustomerID]
  EmployeeID int [ref: > Employees.EmployeeID]
  OrderDate datetime
  Status varchar
}

Table OrderItems {
  OrderItemID int [pk]
  OrderID int [ref: > Orders.OrderID]
  ProductID int [ref: > Products.ProductID]
  Quantity int
  UnitPrice decimal
}

Table Payments {
  PaymentID int [pk]
  OrderID int [ref: > Orders.OrderID]
  Amount decimal
  PaymentMethod varchar
  PaymentDate datetime
}

Table Suppliers {
  SupplierID int [pk]
  SupplierName varchar
  ContactEmail varchar
  Phone varchar
}

Table Shipments {
  ShipmentID int [pk]
  SupplierID int [ref: > Suppliers.SupplierID]
  ProductID int [ref: > Products.ProductID]
  Quantity int
  ShipmentDate date
}

Table Employees {
  EmployeeID int [pk]
  FirstName varchar
  LastName varchar
  Role varchar
  HireDate date
  ManagerID int
}
```

---

## 🧠 SQL Features Demonstrated

### ✔ Joins  
Inner, left, right, full, multi‑table joins.

### ✔ CTEs  
- Analytical CTEs  
- Recursive CTEs (employee hierarchy)

### ✔ Pivoting  
Sales by category per month.

### ✔ Window Functions  
- Ranking customers  
- Running totals  
- Moving averages  

### ✔ Stored Procedures  
- Create new order  
- Update inventory  
- Process payments  

### ✔ Functions  
- Scalar function: order total  
- Table‑valued function: product sales history  

### ✔ Views  
- Customer lifetime value  
- Product performance  
- Monthly revenue  

### ✔ Triggers  
- Prevent negative stock  
- Log order updates  

### ✔ Indexing  
- Performance optimization  

---

## 🔄 ETL Pipeline (Data Engineering Layer)

### **1. Staging Layer**
Raw data loaded from CSV/API.

### **2. Cleansing Layer**
- Standardize column names  
- Validate data types  
- Remove duplicates  

### **3. Core Layer (Operational Tables)**
RetailDB schema.

### **4. Analytics Layer**
Views + aggregated tables.

### **5. Dashboard Layer**
Power BI/Tableau dashboards for:

- Revenue trends  
- Top customers  
- Best‑selling products  
- Inventory alerts  
- Employee performance  

---

## 📊 Business Questions Answered

- Who are the top 10 customers by lifetime value  
- Which products generate the most revenue  
- What is the monthly sales trend  
- Which employees generate the most sales  
- Which suppliers deliver the most inventory  
- Which products are low in stock  
- What is the average order value  
- How many customers are repeat buyers  

---

## 📂 Repository Contents

| Folder | Description |
|--------|-------------|
| `/sql` | All SQL scripts (tables, queries, procedures, views, triggers) |
| `/diagrams` | ERD images + DBML |
| `/pipeline` | ETL documentation |
| `/docs` | Business questions, glossary, dataset description |

---

## 🛠️ Tools Used

- SQL Server / SSMS  
- dbdiagram.io  
- Power BI (optional)  
- GitHub for version control  

---

## ⭐ How to Run This Project

1. Clone the repository  
2. Open SSMS  
3. Run scripts in `/sql` in order  
4. Explore queries and analytics  
5. Review ERD and pipeline docs  

---

# 🧠 **3. Advanced SQL Files (Included in Template)**

You already have the SQL code — here’s how to organize it:

### `/sql/05_ctes_advanced.sql`
- Sales per customer  
- Inventory restock logic  
- Order profitability  

### `/sql/06_recursive_cte.sql`
- Employee hierarchy  
- Category hierarchy (optional)  

### `/sql/07_pivoting.sql`
- Monthly sales by category  

### `/sql/08_window_functions.sql`
- Ranking customers  
- Running totals  
- Moving averages  

---

# 🏗️ **4. Full Data Engineering Pipeline Documentation**

### `/pipeline/pipeline_architecture.md`
- Diagram  
- Flow explanation  
- Data contracts  

### `/pipeline/staging_layer_design.md`
- Raw tables  
- Data validation rules  

### `/pipeline/cleansing_layer_design.md`
- Standardization  
- Deduplication  
- Type enforcement  

### `/pipeline/analytics_layer_design.md`
- Views  
- Aggregated tables  
- Materialized views  

### `/pipeline/dashboard_design.md`
- KPIs  
- Visuals  
- Data sources  

---

Adding **unit tests for SQL** is one of the smartest ways to elevate your project from “student portfolio” to **professional‑grade engineering work**. Most people never include SQL unit tests in their GitHub repos — so this will make your project stand out immediately.

I’ll walk you through:

- What SQL unit testing is  
- The tools you can use  
- A full testing framework for SQL Server  
- Example test cases  
- A reusable folder structure for GitHub  
- Automated testing with tSQLt (industry standard)  
- How to integrate tests into your pipeline  

Let’s build this properly.

---

# ⭐ 1. What Are SQL Unit Tests?

SQL unit tests validate:

- Stored procedures  
- Functions  
- Views  
- Triggers  
- Business logic  
- Data quality rules  

They ensure your SQL code behaves correctly **every time**.

This is exactly how real engineering teams test their data pipelines.

---

# ⭐ 2. Tools for SQL Unit Testing (SQL Server)

The industry standard is:

### ✔ **tSQLt — SQL Server Unit Testing Framework**

It allows you to:

- Mock tables  
- Fake data  
- Test stored procedures  
- Test functions  
- Test triggers  
- Run tests automatically  

And it works entirely inside SQL Server.

---

# ⭐ 3. Add tSQLt to Your Project

### Step 1 — Download tSQLt  
https://tsqlt.org/download/

### Step 2 — Enable CLR  
```sql
EXEC sp_configure 'clr enabled', 1;
RECONFIGURE;
```

### Step 3 — Install tSQLt into your database  
```sql
USE RetailDB;
GO
:r C:\path\to\tsqlt.class.sql
```

Now your database supports unit testing.

---

# ⭐ 4. Create a Test Class

A test class is like a folder for tests.

```sql
EXEC tSQLt.NewTestClass 'testOrders';
```

This creates a schema:

```
testOrders
```

All tests inside this schema will run automatically.

---

# ⭐ 5. Example SQL Unit Tests (Copy‑Paste Ready)

Below are **realistic, production‑style tests** for your RetailDB project.

---

## ✔ Test 1 — Stored Procedure: CreateOrder

Goal: Ensure the procedure creates an order and reduces stock.

```sql
CREATE PROCEDURE testOrders.[test CreateOrder reduces stock]
AS
BEGIN
    -- Fake the Products table
    EXEC tSQLt.FakeTable 'dbo', 'Products';

    INSERT INTO Products (ProductID, ProductName, UnitPrice, Stock)
    VALUES (1, 'Laptop', 1200, 10);

    -- Fake Orders and OrderItems
    EXEC tSQLt.FakeTable 'dbo', 'Orders';
    EXEC tSQLt.FakeTable 'dbo', 'OrderItems';

    -- Execute stored procedure
    EXEC CreateOrder @CustomerID = 1, @ProductID = 1, @Quantity = 2, @EmployeeID = 1;

    -- Assert stock reduced
    EXEC tSQLt.AssertEqualsTable 
        @Expected = 'expectedStock',
        @Actual = 'Products';
END;
GO
```

Expected table:

```sql
CREATE TABLE expectedStock (ProductID INT, ProductName VARCHAR(100), UnitPrice DECIMAL(10,2), Stock INT);
INSERT INTO expectedStock VALUES (1, 'Laptop', 1200, 8);
```

---

## ✔ Test 2 — Function: fn_OrderTotal

```sql
CREATE PROCEDURE testOrders.[test fn_OrderTotal returns correct total]
AS
BEGIN
    EXEC tSQLt.FakeTable 'dbo', 'OrderItems';

    INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)
    VALUES (1, 1, 2, 100), (1, 2, 1, 50);

    DECLARE @result DECIMAL(10,2);
    SELECT @result = dbo.fn_OrderTotal(1);

    EXEC tSQLt.AssertEquals @Expected = 250, @Actual = @result;
END;
GO
```

---

## ✔ Test 3 — Trigger: Prevent Negative Stock

```sql
CREATE PROCEDURE testOrders.[test trigger prevents negative stock]
AS
BEGIN
    EXEC tSQLt.FakeTable 'dbo', 'Products';
    EXEC tSQLt.FakeTable 'dbo', 'OrderItems';

    INSERT INTO Products VALUES (1, 'Laptop', 1200, 1);

    INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)
    VALUES (1, 1, 5, 1200);

    EXEC tSQLt.ExpectException;

END;
GO
```

This test ensures the trigger fires correctly.

---

# ⭐ 6. Run All Tests

```sql
EXEC tSQLt.RunAll;
```

You’ll get a clean test report.

---

# ⭐ 7. GitHub Folder Structure for SQL Unit Tests

Add this to your repo:

```
Retail-SQL-Analytics-Project/
│
├── tests/
│   ├── testOrders/
│   │   ├── test_CreateOrder_reduces_stock.sql
│   │   ├── test_fn_OrderTotal.sql
│   │   ├── test_trigger_PreventNegativeStock.sql
│   │   └── expected/
│   │       └── expectedStock.sql
│   └── testProducts/
│
└── sql/
```

---

# ⭐ 8. Add Unit Testing Section to README.md

You can paste this:

---

## 🧪 SQL Unit Testing (tSQLt)

This project includes a full suite of SQL unit tests using **tSQLt**, the industry‑standard SQL Server testing framework.

### Tests included:

- Stored procedures  
- Functions  
- Triggers  
- Data quality rules  
- Business logic validation  

### Run all tests:

```sql
EXEC tSQLt.RunAll;
```

---

# ⭐ 9. Integrate Unit Tests into Your Data Engineering Pipeline

Here’s how to make this look like a real production pipeline:

### ✔ Step 1 — Load staging data  
### ✔ Step 2 — Run cleansing logic  
### ✔ Step 3 — Run unit tests  
### ✔ Step 4 — If tests pass → load into core tables  
### ✔ Step 5 — If tests fail → send to error logs  

This is exactly how enterprise data teams operate.

Let’s take this step‑by‑step and make it both **practical** and **portfolio‑ready**.  
You’ll understand:

- **What triggers are**  
- **How to create and test them in SSMS**  
- **How they work with the sample RetailDB tables**  
- **How triggers fit into a real ETL pipeline**  

I’ll keep everything grounded in the retail project you’re building so it all connects cleanly.

---

# ⭐ 1. What Are Triggers?

A **trigger** is a special kind of stored procedure that runs **automatically** when a specific event happens on a table.

### SQL Server supports three types:

### ✔ **AFTER triggers**
Run *after* an INSERT, UPDATE, or DELETE.

### ✔ **INSTEAD OF triggers**
Replace the normal action (useful for views).

### ✔ **DDL triggers**
Fire on schema changes (CREATE TABLE, ALTER TABLE, etc.).

---

# ⭐ 2. Why Triggers Are Used

Triggers are perfect for:

- Enforcing business rules  
- Maintaining audit logs  
- Preventing invalid data  
- Cascading updates  
- Data quality enforcement  
- ETL validation  

In your RetailDB project, triggers help ensure:

- Stock never goes negative  
- Orders cannot be created for inactive customers  
- Payments cannot exceed order total  
- Audit logs track changes  

---

# ⭐ 3. How to Work With Triggers in SSMS

### ✔ Step 1 — Expand your database  
```
RetailDB → Programmability → Database Triggers / Table Triggers
```

### ✔ Step 2 — Right‑click → New Trigger  
Or simply write the SQL manually.

---

# ⭐ 4. Sample Triggers for Your RetailDB Project

Let’s build **realistic, production‑style triggers** using the tables you already created.

---

# 🔥 Trigger 1 — Prevent Negative Stock  
(One of the most common real‑world triggers)

```sql
CREATE TRIGGER trg_PreventNegativeStock
ON OrderItems
AFTER INSERT, UPDATE
AS
BEGIN
    IF EXISTS (
        SELECT 1
        FROM Products p
        JOIN inserted i ON p.ProductID = i.ProductID
        WHERE p.Stock - i.Quantity < 0
    )
    BEGIN
        ROLLBACK;
        RAISERROR ('Stock cannot go negative', 16, 1);
    END
END;
GO
```

### ✔ What this does  
If someone tries to insert an order item that would make stock negative → the entire transaction is cancelled.

---

# 🔥 Trigger 2 — Auto‑Update Stock After Order Creation

```sql
CREATE TRIGGER trg_UpdateStockAfterOrder
ON OrderItems
AFTER INSERT
AS
BEGIN
    UPDATE p
    SET p.Stock = p.Stock - i.Quantity
    FROM Products p
    JOIN inserted i ON p.ProductID = i.ProductID;
END;
GO
```

### ✔ What this does  
Whenever an order item is added, stock is reduced automatically.

---

# 🔥 Trigger 3 — Audit Log for Order Updates

### Step 1 — Create audit table

```sql
CREATE TABLE OrderAuditLog (
    AuditID INT IDENTITY PRIMARY KEY,
    OrderID INT,
    OldStatus VARCHAR(20),
    NewStatus VARCHAR(20),
    ChangedAt DATETIME DEFAULT GETDATE()
);
```

### Step 2 — Create trigger

```sql
CREATE TRIGGER trg_OrderStatusAudit
ON Orders
AFTER UPDATE
AS
BEGIN
    INSERT INTO OrderAuditLog (OrderID, OldStatus, NewStatus)
    SELECT 
        d.OrderID,
        d.Status AS OldStatus,
        i.Status AS NewStatus
    FROM deleted d
    JOIN inserted i ON d.OrderID = i.OrderID
    WHERE d.Status <> i.Status;
END;
GO
```

### ✔ What this does  
Every time an order’s status changes, the change is logged.

---

# ⭐ 5. How to Test Triggers in SSMS

### Test negative stock trigger:

```sql
INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)
VALUES (1, 1, 999, 1200);
```

You should get:

```
Stock cannot go negative
```

### Test audit trigger:

```sql
UPDATE Orders
SET Status = 'Completed'
WHERE OrderID = 2;
```

Then check:

```sql
SELECT * FROM OrderAuditLog;
```

---

# ⭐ 6. How Triggers Fit Into an ETL Pipeline

Triggers are extremely useful in ETL pipelines for:

- **Data validation**  
- **Data quality enforcement**  
- **Automatic transformations**  
- **Audit logging**  

Let’s map this to your RetailDB pipeline.

---

# ⭐ 7. ETL Logic Using Triggers (End‑to‑End)

## ✔ Step 1 — Load raw data into Staging

```sql
INSERT INTO Staging_Orders (...)
SELECT * FROM OPENROWSET(BULK 'orders.csv', FORMAT='CSV', FIRSTROW=2) AS Data;
```

---

## ✔ Step 2 — Validate data using flags

```sql
UPDATE Staging_Orders
SET IsValid = 0
WHERE Quantity <= 0 OR UnitPrice <= 0;
```

---

## ✔ Step 3 — Trigger prevents invalid inserts into core tables

When loading into Orders and OrderItems:

```sql
INSERT INTO OrderItems (...)
SELECT ...
FROM Staging_Orders
WHERE IsValid = 1;
```

If a row would cause negative stock → trigger blocks it.

---

## ✔ Step 4 — Trigger automatically updates stock

When valid order items load:

- Stock is reduced  
- No need for manual logic  

---

## ✔ Step 5 — Trigger logs changes

If ETL updates order statuses:

- Audit log captures changes  
- You maintain full history  

---

# ⭐ 8. Should You Use Triggers in Production?

### ✔ Use triggers for:
- Data quality enforcement  
- Audit logging  
- Preventing invalid states  
- Automatic cascading updates  

### ❌ Avoid triggers for:
- Heavy transformations  
- Complex business logic  
- High‑volume ETL (can slow down loads)  

For heavy ETL, use:

- Stored procedures  
- SSIS / ADF / Databricks  
- Scheduled jobs  

---

# ⭐ 9. Want me to generate a full ETL script?

I can create:

- Staging tables  
- Cleansing logic  
- Validation rules  
- Load into core tables  
- Trigger‑based enforcement  
- Error logging tables  
- Retry logic  

Just tell me if you want:

### **A. Basic ETL**  
or  
### **B. Enterprise‑grade ETL with error handling, logging, and retries**



You’re aiming for the real thing now—not just “load some data,” but something that looks like what an actual data team would build. Let’s design an **enterprise‑style ETL** around your RetailDB, with:

- Staging → Validation → Core load  
- Error handling & logging  
- Retry mechanism  
- Triggers only where they add real value  

I’ll keep everything in **pure SQL Server / SSMS**, so you can run it end‑to‑end.

---

## 1. High‑level ETL architecture

**Flow:**

1. **Landing/Raw**: CSV/flat files land outside SQL (e.g., file share, S3, etc.).  
2. **Staging tables**: Raw data loaded as‑is (minimal constraints).  
3. **Validation & cleansing**: Data quality checks, flags, error logging.  
4. **Core tables**: Clean data inserted into `Orders`, `OrderItems`, etc.  
5. **Retry logic**: Failed rows can be fixed and reprocessed.  
6. **Audit & logging**: Every ETL run is tracked.

We’ll implement steps 2–6 in SQL.

---

## 2. Create ETL metadata & logging tables

### 2.1. ETL run log

```sql
CREATE TABLE ETL_RunLog (
    RunID INT IDENTITY PRIMARY KEY,
    ProcessName VARCHAR(100),
    StartTime DATETIME DEFAULT GETDATE(),
    EndTime DATETIME NULL,
    Status VARCHAR(20),          -- 'Started', 'Success', 'Failed'
    ErrorMessage VARCHAR(4000) NULL
);
```

### 2.2. Row‑level error log

```sql
CREATE TABLE ETL_ErrorLog (
    ErrorID INT IDENTITY PRIMARY KEY,
    RunID INT,
    SourceTable VARCHAR(100),
    SourceRowID INT NULL,
    ErrorType VARCHAR(100),
    ErrorMessage VARCHAR(4000),
    LoggedAt DATETIME DEFAULT GETDATE()
);
```

---

## 3. Staging tables with flags

### 3.1. Staging_Orders

```sql
CREATE TABLE Staging_Orders (
    StagingOrderID INT IDENTITY PRIMARY KEY,
    CustomerID INT,
    ProductID INT,
    Quantity INT,
    UnitPrice DECIMAL(10,2),
    OrderDate DATETIME,
    RawFileName VARCHAR(255) NULL,
    IsValid BIT DEFAULT 1,
    IsProcessed BIT DEFAULT 0,
    RetryCount INT DEFAULT 0
);
```

This table is where you bulk insert from CSV (e.g., `BULK INSERT`, SSIS, ADF, etc.).

---

## 4. Validation logic (data quality rules)

We’ll validate:

- Customer exists  
- Product exists  
- Quantity > 0  
- UnitPrice > 0  

### 4.1. Mark invalid rows

```sql
UPDATE s
SET 
    IsValid = 0
FROM Staging_Orders s
LEFT JOIN Customers c ON s.CustomerID = c.CustomerID
LEFT JOIN Products p ON s.ProductID = p.ProductID
WHERE 
    c.CustomerID IS NULL
    OR p.ProductID IS NULL
    OR s.Quantity IS NULL OR s.Quantity <= 0
    OR s.UnitPrice IS NULL OR s.UnitPrice <= 0;
```

### 4.2. Log invalid rows

```sql
DECLARE @RunID INT;

INSERT INTO ETL_RunLog (ProcessName, Status)
VALUES ('Orders_ETL', 'Started');

SET @RunID = SCOPE_IDENTITY();

INSERT INTO ETL_ErrorLog (RunID, SourceTable, SourceRowID, ErrorType, ErrorMessage)
SELECT 
    @RunID,
    'Staging_Orders',
    StagingOrderID,
    'ValidationError',
    CONCAT(
        'Invalid row. CustomerID=', CustomerID,
        ', ProductID=', ProductID,
        ', Quantity=', Quantity,
        ', UnitPrice=', UnitPrice
    )
FROM Staging_Orders
WHERE IsValid = 0;
```

---

## 5. Load valid rows into core tables with transaction & error handling

We’ll wrap the core load in a **TRY/CATCH** block and update the ETL run log.

```sql
BEGIN TRY
    DECLARE @RunID INT;

    INSERT INTO ETL_RunLog (ProcessName, Status)
    VALUES ('Orders_ETL', 'Started');

    SET @RunID = SCOPE_IDENTITY();

    BEGIN TRAN;

    -- Insert into Orders (one per distinct Customer + OrderDate)
    INSERT INTO Orders (CustomerID, OrderDate, Status)
    SELECT DISTINCT
        CustomerID,
        OrderDate,
        'Completed'
    FROM Staging_Orders
    WHERE IsValid = 1 AND IsProcessed = 0;

    -- Insert into OrderItems
    INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)
    SELECT 
        o.OrderID,
        s.ProductID,
        s.Quantity,
        s.UnitPrice
    FROM Staging_Orders s
    JOIN Orders o 
        ON s.CustomerID = o.CustomerID 
       AND s.OrderDate = o.OrderDate
    WHERE s.IsValid = 1 AND s.IsProcessed = 0;

    -- Mark staging rows as processed
    UPDATE Staging_Orders
    SET IsProcessed = 1
    WHERE IsValid = 1 AND IsProcessed = 0;

    COMMIT;

    UPDATE ETL_RunLog
    SET Status = 'Success', EndTime = GETDATE()
    WHERE RunID = @RunID;
END TRY
BEGIN CATCH
    DECLARE @ErrorMessage VARCHAR(4000) = ERROR_MESSAGE();
    DECLARE @RunID2 INT;

    -- If RunID not set in this scope, create one
    IF @RunID IS NULL
    BEGIN
        INSERT INTO ETL_RunLog (ProcessName, Status, ErrorMessage)
        VALUES ('Orders_ETL', 'Failed', @ErrorMessage);

        SET @RunID2 = SCOPE_IDENTITY();
    END
    ELSE
    BEGIN
        SET @RunID2 = @RunID;
        UPDATE ETL_RunLog
        SET Status = 'Failed', EndTime = GETDATE(), ErrorMessage = @ErrorMessage
        WHERE RunID = @RunID2;
    END

    ROLLBACK;

    INSERT INTO ETL_ErrorLog (RunID, SourceTable, ErrorType, ErrorMessage)
    VALUES (@RunID2, 'Orders_ETL', 'LoadError', @ErrorMessage);
END CATCH;
```

---

## 6. Retry logic for failed rows

We’ll allow **fixing bad rows** and retrying them.

### 6.1. Identify rows eligible for retry

```sql
SELECT *
FROM Staging_Orders
WHERE IsValid = 0 AND RetryCount < 3;
```

You (or a process) can:

- Fix the data (e.g., correct CustomerID, ProductID, Quantity, etc.)  
- Set `IsValid = 1`  
- Increment `RetryCount`  

```sql
UPDATE Staging_Orders
SET 
    IsValid = 1,
    RetryCount = RetryCount + 1
WHERE StagingOrderID = @SomeID;
```

Then rerun the ETL load block.

---

## 7. Using triggers as guardrails (not as ETL engines)

We keep heavy logic in stored procedures, but use triggers to **enforce invariants**.

### 7.1. Trigger to prevent negative stock

```sql
CREATE TRIGGER trg_PreventNegativeStock
ON OrderItems
AFTER INSERT, UPDATE
AS
BEGIN
    IF EXISTS (
        SELECT 1
        FROM Products p
        JOIN inserted i ON p.ProductID = i.ProductID
        WHERE p.Stock - i.Quantity < 0
    )
    BEGIN
        ROLLBACK;
        RAISERROR ('Stock cannot go negative', 16, 1);
    END
END;
GO
```

### 7.2. Trigger to auto‑update stock

```sql
CREATE TRIGGER trg_UpdateStockAfterOrder
ON OrderItems
AFTER INSERT
AS
BEGIN
    UPDATE p
    SET p.Stock = p.Stock - i.Quantity
    FROM Products p
    JOIN inserted i ON p.ProductID = i.ProductID;
END;
GO
```

If ETL accidentally tries to insert something that breaks stock rules, the trigger blocks it and the **TRY/CATCH** captures the error.

---

## 8. How this looks in a real pipeline

**Nightly job / scheduled job:**

1. Load CSV → `Staging_Orders`  
2. Run validation script (set `IsValid`, log errors)  
3. Run ETL load script (TRY/CATCH, transaction, logging)  
4. Review `ETL_RunLog` and `ETL_ErrorLog`  
5. Fix bad rows in staging, increment `RetryCount`  
6. Rerun ETL for retries  

This is the kind of pattern real teams use.

---

If you want, next we can:

- Wrap this ETL into a **stored procedure** like `usp_RunOrdersETL`  
- Add **ETL for Products, Customers, Shipments**  
- Design a **monitoring dashboard** (e.g., Power BI) over `ETL_RunLog` and `ETL_ErrorLog` so you can *see* failures and trends.

Absolutely — let’s wrap your entire enterprise‑grade ETL pipeline into a **single, production‑ready stored procedure** called **`usp_RunOrdersETL`**.  

This procedure will include:

- ETL run logging  
- Validation  
- Error logging  
- Transaction handling  
- Retry logic  
- Loading into core tables  
- Updating staging flags  
- Trigger‑safe inserts  

This is the kind of stored procedure you’d see in a real enterprise data warehouse.

---

# ⭐ **Stored Procedure: `usp_RunOrdersETL` (Enterprise‑Grade)**

Below is the full, ready‑to‑run stored procedure.  
It assumes you already created:

- `ETL_RunLog`  
- `ETL_ErrorLog`  
- `Staging_Orders`  
- Core tables (`Orders`, `OrderItems`, etc.)  
- Triggers (optional but recommended)

---

# 🧱 **1. Create the Stored Procedure**

```sql
CREATE OR ALTER PROCEDURE usp_RunOrdersETL
AS
BEGIN
    SET NOCOUNT ON;

    DECLARE @RunID INT;
    DECLARE @ErrorMessage VARCHAR(4000);

    ---------------------------------------------------------
    -- 1. Start ETL Run Log
    ---------------------------------------------------------
    INSERT INTO ETL_RunLog (ProcessName, Status)
    VALUES ('Orders_ETL', 'Started');

    SET @RunID = SCOPE_IDENTITY();


    BEGIN TRY
        ---------------------------------------------------------
        -- 2. VALIDATION STEP
        ---------------------------------------------------------
        -- Mark invalid rows
        UPDATE s
        SET IsValid = 0
        FROM Staging_Orders s
        LEFT JOIN Customers c ON s.CustomerID = c.CustomerID
        LEFT JOIN Products p ON s.ProductID = p.ProductID
        WHERE 
            c.CustomerID IS NULL
            OR p.ProductID IS NULL
            OR s.Quantity IS NULL OR s.Quantity <= 0
            OR s.UnitPrice IS NULL OR s.UnitPrice <= 0;


        ---------------------------------------------------------
        -- 3. LOG INVALID ROWS
        ---------------------------------------------------------
        INSERT INTO ETL_ErrorLog (RunID, SourceTable, SourceRowID, ErrorType, ErrorMessage)
        SELECT 
            @RunID,
            'Staging_Orders',
            StagingOrderID,
            'ValidationError',
            CONCAT(
                'Invalid row. CustomerID=', CustomerID,
                ', ProductID=', ProductID,
                ', Quantity=', Quantity,
                ', UnitPrice=', UnitPrice
            )
        FROM Staging_Orders
        WHERE IsValid = 0 AND IsProcessed = 0;


        ---------------------------------------------------------
        -- 4. BEGIN TRANSACTION FOR CORE LOAD
        ---------------------------------------------------------
        BEGIN TRAN;

        ---------------------------------------------------------
        -- 5. LOAD INTO ORDERS TABLE
        ---------------------------------------------------------
        INSERT INTO Orders (CustomerID, OrderDate, Status)
        SELECT DISTINCT
            CustomerID,
            OrderDate,
            'Completed'
        FROM Staging_Orders
        WHERE IsValid = 1 AND IsProcessed = 0;


        ---------------------------------------------------------
        -- 6. LOAD INTO ORDER ITEMS TABLE
        ---------------------------------------------------------
        INSERT INTO OrderItems (OrderID, ProductID, Quantity, UnitPrice)
        SELECT 
            o.OrderID,
            s.ProductID,
            s.Quantity,
            s.UnitPrice
        FROM Staging_Orders s
        JOIN Orders o 
            ON s.CustomerID = o.CustomerID 
           AND s.OrderDate = o.OrderDate
        WHERE s.IsValid = 1 AND s.IsProcessed = 0;


        ---------------------------------------------------------
        -- 7. MARK STAGING ROWS AS PROCESSED
        ---------------------------------------------------------
        UPDATE Staging_Orders
        SET IsProcessed = 1
        WHERE IsValid = 1 AND IsProcessed = 0;


        ---------------------------------------------------------
        -- 8. COMMIT TRANSACTION
        ---------------------------------------------------------
        COMMIT;


        ---------------------------------------------------------
        -- 9. UPDATE RUN LOG AS SUCCESS
        ---------------------------------------------------------
        UPDATE ETL_RunLog
        SET Status = 'Success', EndTime = GETDATE()
        WHERE RunID = @RunID;

    END TRY
    BEGIN CATCH
        ---------------------------------------------------------
        -- 10. ERROR HANDLING
        ---------------------------------------------------------
        SET @ErrorMessage = ERROR_MESSAGE();

        ROLLBACK;

        -- Log ETL failure
        UPDATE ETL_RunLog
        SET Status = 'Failed', EndTime = GETDATE(), ErrorMessage = @ErrorMessage
        WHERE RunID = @RunID;

        -- Log error details
        INSERT INTO ETL_ErrorLog (RunID, SourceTable, ErrorType, ErrorMessage)
        VALUES (@RunID, 'Orders_ETL', 'LoadError', @ErrorMessage);
    END CATCH;
END;
GO
```

---

# ⭐ **2. How to Run the ETL**

```sql
EXEC usp_RunOrdersETL;
```

This will:

- Validate staging rows  
- Log invalid rows  
- Insert valid rows into Orders + OrderItems  
- Update stock (via trigger)  
- Log ETL run status  
- Log errors  
- Mark staging rows as processed  

---

# ⭐ **3. Retry Logic (Enterprise‑Style)**

If a row fails validation:

```sql
SELECT * 
FROM Staging_Orders
WHERE IsValid = 0 AND RetryCount < 3;
```

Fix the row:

```sql
UPDATE Staging_Orders
SET 
    CustomerID = 1,   -- corrected
    IsValid = 1,
    RetryCount = RetryCount + 1
WHERE StagingOrderID = 42;
```

Then rerun:

```sql
EXEC usp_RunOrdersETL;
```

---

# ⭐ **4. Why This Procedure Is Enterprise‑Grade**

### ✔ Full transaction control  
Prevents partial loads.

### ✔ Error logging  
Captures row‑level and run‑level errors.

### ✔ Retry mechanism  
Allows fixing bad rows without losing them.

### ✔ Staging flags  
`IsValid`, `IsProcessed`, `RetryCount`.

### ✔ Trigger‑safe  
Triggers enforce business rules (e.g., stock cannot go negative).

### ✔ Idempotent  
Running it twice won’t duplicate data.

### ✔ Auditable  
Every run is logged in `ETL_RunLog`.

---




