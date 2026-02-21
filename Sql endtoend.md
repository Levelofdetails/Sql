/* =========================================================
   1. Database (optional)
   ========================================================= */
IF DB_ID('SalesPracticeDB') IS NULL
    CREATE DATABASE SalesPracticeDB;
GO

USE SalesPracticeDB;
GO

/* =========================================================
   2. Drop objects if they already exist (clean rerun)
   ========================================================= */
IF OBJECT_ID('dbo.trg_UpdateOrderTotal', 'TR') IS NOT NULL
    DROP TRIGGER dbo.trg_UpdateOrderTotal;
GO

IF OBJECT_ID('dbo.Refresh_FactSales', 'P') IS NOT NULL
    DROP PROCEDURE dbo.Refresh_FactSales;
GO

IF OBJECT_ID('dbo.FactSales', 'U') IS NOT NULL
    DROP TABLE dbo.FactSales;
GO

IF OBJECT_ID('dbo.Payments', 'U') IS NOT NULL
    DROP TABLE dbo.Payments;
GO

IF OBJECT_ID('dbo.OrderItems', 'U') IS NOT NULL
    DROP TABLE dbo.OrderItems;
GO

IF OBJECT_ID('dbo.Orders', 'U') IS NOT NULL
    DROP TABLE dbo.Orders;
GO

IF OBJECT_ID('dbo.Products', 'U') IS NOT NULL
    DROP TABLE dbo.Products;
GO

IF OBJECT_ID('dbo.Customers', 'U') IS NOT NULL
    DROP TABLE dbo.Customers;
GO

/* =========================================================
   3. Create Tables
   ========================================================= */

-- Customers
CREATE TABLE dbo.Customers (
    CustomerID   INT           NOT NULL PRIMARY KEY,
    FirstName    VARCHAR(50)   NOT NULL,
    LastName     VARCHAR(50)   NOT NULL,
    Email        VARCHAR(100)  NOT NULL,
    Phone        VARCHAR(20)   NULL,
    City         VARCHAR(50)   NULL,
    State        VARCHAR(50)   NULL,
    CreatedDate  DATE          NOT NULL
);
GO

-- Products
CREATE TABLE dbo.Products (
    ProductID   INT           NOT NULL PRIMARY KEY,
    ProductName VARCHAR(100)  NOT NULL,
    Category    VARCHAR(50)   NOT NULL,
    UnitPrice   DECIMAL(10,2) NOT NULL,
    IsActive    BIT           NOT NULL
);
GO

-- Orders
CREATE TABLE dbo.Orders (
    OrderID      INT           NOT NULL PRIMARY KEY,
    CustomerID   INT           NOT NULL,
    OrderDate    DATE          NOT NULL,
    OrderStatus  VARCHAR(20)   NOT NULL,
    TotalAmount  DECIMAL(12,2) NULL
);
GO

-- OrderItems
CREATE TABLE dbo.OrderItems (
    OrderItemID INT           NOT NULL PRIMARY KEY,
    OrderID     INT           NOT NULL,
    ProductID   INT           NOT NULL,
    Quantity    INT           NOT NULL,
    UnitPrice   DECIMAL(10,2) NOT NULL,
    LineTotal   DECIMAL(12,2) NOT NULL
);
GO

-- Payments
CREATE TABLE dbo.Payments (
    PaymentID     INT           NOT NULL PRIMARY KEY,
    OrderID       INT           NOT NULL,
    PaymentDate   DATE          NOT NULL,
    PaymentMethod VARCHAR(20)   NOT NULL,
    AmountPaid    DECIMAL(12,2) NOT NULL,
    PaymentStatus VARCHAR(20)   NOT NULL
);
GO

/* =========================================================
   4. Foreign Keys
   ========================================================= */

ALTER TABLE dbo.Orders
ADD CONSTRAINT FK_Orders_Customers
    FOREIGN KEY (CustomerID) REFERENCES dbo.Customers(CustomerID);
GO

ALTER TABLE dbo.OrderItems
ADD CONSTRAINT FK_OrderItems_Orders
    FOREIGN KEY (OrderID) REFERENCES dbo.Orders(OrderID);
GO

ALTER TABLE dbo.OrderItems
ADD CONSTRAINT FK_OrderItems_Products
    FOREIGN KEY (ProductID) REFERENCES dbo.Products(ProductID);
GO

ALTER TABLE dbo.Payments
ADD CONSTRAINT FK_Payments_Orders
    FOREIGN KEY (OrderID) REFERENCES dbo.Orders(OrderID);
GO

/* =========================================================
   5. Insert Sample Data
   ========================================================= */

-- Customers (20 rows)
INSERT INTO dbo.Customers (CustomerID, FirstName, LastName, Email, Phone, City, State, CreatedDate) VALUES
(1,'Ravi','Kumar','ravi.kumar@example.com','9876543210','Hyderabad','TS','2023-01-10'),
(2,'Sita','Reddy','sita.reddy@example.com','9876543211','Bangalore','KA','2023-01-12'),
(3,'John','Doe','john.doe@example.com','9876543212','Chennai','TN','2023-01-15'),
(4,'Mary','Thomas','mary.t@example.com','9876543213','Mumbai','MH','2023-01-18'),
(5,'Arjun','Patel','arjun.p@example.com','9876543214','Pune','MH','2023-01-20'),
(6,'Kiran','Shah','kiran.shah@example.com','9876543215','Delhi','DL','2023-01-22'),
(7,'Priya','Nair','priya.n@example.com','9876543216','Kochi','KL','2023-01-25'),
(8,'Rahul','Verma','rahul.v@example.com','9876543217','Noida','UP','2023-01-28'),
(9,'Anita','Singh','anita.s@example.com','9876543218','Gurgaon','HR','2023-02-01'),
(10,'Vijay','Menon','vijay.m@example.com','9876543219','Trivandrum','KL','2023-02-03'),
(11,'Deepak','Joshi','deepak.j@example.com','9876543220','Surat','GJ','2023-02-05'),
(12,'Sneha','Kapoor','sneha.k@example.com','9876543221','Indore','MP','2023-02-07'),
(13,'Rohit','Bansal','rohit.b@example.com','9876543222','Nagpur','MH','2023-02-10'),
(14,'Meera','Iyer','meera.i@example.com','9876543223','Chennai','TN','2023-02-12'),
(15,'Tarun','Goyal','tarun.g@example.com','9876543224','Jaipur','RJ','2023-02-14'),
(16,'Lakshmi','Rao','lakshmi.r@example.com','9876543225','Vizag','AP','2023-02-16'),
(17,'Suresh','Pillai','suresh.p@example.com','9876543226','Kollam','KL','2023-02-18'),
(18,'Asha','Mishra','asha.m@example.com','9876543227','Patna','BR','2023-02-20'),
(19,'Naveen','Shetty','naveen.s@example.com','9876543228','Mangalore','KA','2023-02-22'),
(20,'Harsha','Gowda','harsha.g@example.com','9876543229','Bangalore','KA','2023-02-24');
GO

-- Products (20 rows)
INSERT INTO dbo.Products (ProductID, ProductName, Category, UnitPrice, IsActive) VALUES
(1,'Laptop Basic','Electronics',45000,1),
(2,'Laptop Pro','Electronics',78000,1),
(3,'Mouse Wireless','Accessories',1200,1),
(4,'Keyboard Mechanical','Accessories',3500,1),
(5,'Monitor 24 inch','Electronics',11000,1),
(6,'USB-C Cable','Accessories',500,1),
(7,'Smartphone A1','Mobiles',18000,1),
(8,'Smartphone X2','Mobiles',35000,1),
(9,'Earbuds','Audio',2500,1),
(10,'Bluetooth Speaker','Audio',4000,1),
(11,'Tablet Mini','Mobiles',22000,1),
(12,'Tablet Pro','Mobiles',45000,1),
(13,'Printer Inkjet','Electronics',9000,1),
(14,'Webcam HD','Accessories',2500,1),
(15,'Router Dual Band','Networking',3000,1),
(16,'SSD 512GB','Storage',5500,1),
(17,'HDD 1TB','Storage',4000,1),
(18,'Power Bank','Accessories',1500,1),
(19,'Smartwatch','Wearables',7000,1),
(20,'Fitness Band','Wearables',2500,1);
GO

-- Orders (20 rows)
INSERT INTO dbo.Orders (OrderID, CustomerID, OrderDate, OrderStatus, TotalAmount) VALUES
(1,1,'2023-03-01','Completed',45000),
(2,2,'2023-03-02','Completed',1200),
(3,3,'2023-03-03','Completed',35000),
(4,4,'2023-03-04','Pending',2500),
(5,5,'2023-03-05','Completed',78000),
(6,6,'2023-03-06','Completed',11000),
(7,7,'2023-03-07','Completed',4000),
(8,8,'2023-03-08','Completed',18000),
(9,9,'2023-03-09','Completed',2500),
(10,10,'2023-03-10','Completed',7000),
(11,11,'2023-03-11','Completed',5500),
(12,12,'2023-03-12','Completed',9000),
(13,13,'2023-03-13','Completed',45000),
(14,14,'2023-03-14','Completed',3500),
(15,15,'2023-03-15','Completed',2500),
(16,16,'2023-03-16','Completed',3000),
(17,17,'2023-03-17','Completed',1500),
(18,18,'2023-03-18','Completed',2500),
(19,19,'2023-03-19','Completed',4000),
(20,20,'2023-03-20','Completed',2500);
GO

-- OrderItems (20 rows)
INSERT INTO dbo.OrderItems (OrderItemID, OrderID, ProductID, Quantity, UnitPrice, LineTotal) VALUES
(1,1,1,1,45000,45000),
(2,2,3,1,1200,1200),
(3,3,8,1,35000,35000),
(4,4,9,1,2500,2500),
(5,5,2,1,78000,78000),
(6,6,5,1,11000,11000),
(7,7,10,1,4000,4000),
(8,8,7,1,18000,18000),
(9,9,9,1,2500,2500),
(10,10,19,1,7000,7000),
(11,11,16,1,5500,5500),
(12,12,13,1,9000,9000),
(13,13,12,1,45000,45000),
(14,14,4,1,3500,3500),
(15,15,3,2,1200,2400),
(16,16,15,1,3000,3000),
(17,17,18,1,1500,1500),
(18,18,3,1,1200,1200),
(19,19,10,1,4000,4000),
(20,20,3,1,1200,1200);
GO

-- Payments (20 rows)
INSERT INTO dbo.Payments (PaymentID, OrderID, PaymentDate, PaymentMethod, AmountPaid, PaymentStatus) VALUES
(1,1,'2023-03-01','UPI',45000,'Success'),
(2,2,'2023-03-02','Card',1200,'Success'),
(3,3,'2023-03-03','UPI',35000,'Success'),
(4,4,'2023-03-04','UPI',2500,'Pending'),
(5,5,'2023-03-05','Card',78000,'Success'),
(6,6,'2023-03-06','UPI',11000,'Success'),
(7,7,'2023-03-07','Card',4000,'Success'),
(8,8,'2023-03-08','UPI',18000,'Success'),
(9,9,'2023-03-09','UPI',2500,'Success'),
(10,10,'2023-03-10','Card',7000,'Success'),
(11,11,'2023-03-11','UPI',5500,'Success'),
(12,12,'2023-03-12','Card',9000,'Success'),
(13,13,'2023-03-13','UPI',45000,'Success'),
(14,14,'2023-03-14','UPI',3500,'Success'),
(15,15,'2023-03-15','Card',2500,'Success'),
(16,16,'2023-03-16','UPI',3000,'Success'),
(17,17,'2023-03-17','UPI',1500,'Success'),
(18,18,'2023-03-18','Card',1200,'Success'),
(19,19,'2023-03-19','UPI',4000,'Success'),
(20,20,'2023-03-20','UPI',1200,'Success');
GO

/* =========================================================
   6. Final Consumption Table (FactSales)
   ========================================================= */

CREATE TABLE dbo.FactSales (
    SalesID        INT IDENTITY(1,1) PRIMARY KEY,
    OrderID        INT           NOT NULL,
    CustomerID     INT           NOT NULL,
    ProductID      INT           NOT NULL,
    OrderDate      DATE          NOT NULL,
    Quantity       INT           NOT NULL,
    LineTotal      DECIMAL(12,2) NOT NULL,
    PaymentMethod  VARCHAR(20)   NOT NULL,
    PaymentStatus  VARCHAR(20)   NOT NULL,
    LastUpdated    DATETIME      NOT NULL DEFAULT GETDATE()
);
GO

/* =========================================================
   7. Stored Procedure: Incremental Refresh of FactSales
   ========================================================= */

CREATE PROCEDURE dbo.Refresh_FactSales
AS
BEGIN
    SET NOCOUNT ON;

    MERGE dbo.FactSales AS T
    USING (
        SELECT 
            o.OrderID,
            o.CustomerID,
            oi.ProductID,
            o.OrderDate,
            oi.Quantity,
            oi.LineTotal,
            p.PaymentMethod,
            p.PaymentStatus
        FROM dbo.Orders o
        JOIN dbo.OrderItems oi ON o.OrderID = oi.OrderID
        JOIN dbo.Payments p ON o.OrderID = p.OrderID
    ) AS S
    ON T.OrderID = S.OrderID
       AND T.ProductID = S.ProductID

    WHEN MATCHED AND (
        T.Quantity      <> S.Quantity OR
        T.LineTotal     <> S.LineTotal OR
        T.PaymentStatus <> S.PaymentStatus OR
        T.PaymentMethod <> S.PaymentMethod
    )
    THEN UPDATE SET
        T.Quantity      = S.Quantity,
        T.LineTotal     = S.LineTotal,
        T.PaymentMethod = S.PaymentMethod,
        T.PaymentStatus = S.PaymentStatus,
        T.LastUpdated   = GETDATE()

    WHEN NOT MATCHED BY TARGET
    THEN INSERT (OrderID, CustomerID, ProductID, OrderDate, Quantity, LineTotal, PaymentMethod, PaymentStatus)
         VALUES (S.OrderID, S.CustomerID, S.ProductID, S.OrderDate, S.Quantity, S.LineTotal, S.PaymentMethod, S.PaymentStatus);
END;
GO

/* =========================================================
   8. Trigger: Auto-update Orders.TotalAmount from OrderItems
   ========================================================= */

CREATE TRIGGER dbo.trg_UpdateOrderTotal
ON dbo.OrderItems
AFTER INSERT, UPDATE, DELETE
AS
BEGIN
    SET NOCOUNT ON;

    UPDATE o
    SET o.TotalAmount = ISNULL(oi.SumLineTotal, 0)
    FROM dbo.Orders o
    JOIN (
        SELECT OrderID, SUM(LineTotal) AS SumLineTotal
        FROM dbo.OrderItems
        GROUP BY OrderID
    ) oi
        ON o.OrderID = oi.OrderID
    WHERE o.OrderID IN (
        SELECT DISTINCT OrderID FROM inserted
        UNION
        SELECT DISTINCT OrderID FROM deleted
    );
END;
GO

/* =========================================================
   9. First Load of FactSales
   ========================================================= */

EXEC dbo.Refresh_FactSales;
GO

/* =========================================================
   10. Sample Practice Query (CTE + Joins)
   ========================================================= */

-- Top 5 customers by total spend
WITH SalesCTE AS (
    SELECT 
        c.CustomerID,
        c.FirstName + ' ' + c.LastName AS CustomerName,
        SUM(oi.LineTotal) AS TotalSpent
    FROM dbo.Customers c
    JOIN dbo.Orders o      ON c.CustomerID = o.CustomerID
    JOIN dbo.OrderItems oi ON o.OrderID   = oi.OrderID
    GROUP BY c.CustomerID, c.FirstName, c.LastName
)
SELECT TOP 5 *
FROM SalesCTE
ORDER BY TotalSpent DESC;
GO
