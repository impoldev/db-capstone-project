# Database Engineer Capstone Project

This is my final project for the Database Engineer Capstone course hosted at Coursera. It includes:
- Robust SQL scripts with inserts, joins, views, subqueries, procedures, prepared statements, and more.
- Data analysis with Tableau.
- Integration with MySQL Connector Python using Jupyter.

## 📦 1. Database

Find the data model extracted from MySQL Workbench, png snapshots for every task and the final SQL database in the folder: **`1. Database`**. Below are all the SQL statements used in the project after building the database from the data model.

### ✅ SQL INSERT statements

```sql
-- Insert into Customers
INSERT INTO Customers (CustomerID, FullName, Email, ContactNumber) VALUES
(1, 'Alice Smith', 'alice@example.com', '1234567890'),
(2, 'Bob Johnson', 'bob@example.com', '9876543210'),
(3, 'Carla Gomez', 'carla@example.com', '5551234567'),
(4, 'David Lee', 'david@example.com', '4449876543'),
(5, 'Emma Davis', 'emma@example.com', '3337778899');

-- Insert into Staff
INSERT INTO Staff (StaffID, StaffName, Role, Salary) VALUES
(1, 'John Doe', 'Waiter', 30000),
(2, 'Jane Roe', 'Chef', 45000),
(3, 'Mark Lin', 'Manager', 60000),
(4, 'Olivia Kim', 'Waiter', 32000);

-- Insert into MenuItems
INSERT INTO MenuItems (MenuItemID, MenuItemName, Type, Price, Stock) VALUES
(1, 'Bruschetta', 'starter', 6.99, 50),
(2, 'Garlic Bread', 'starter', 5.50, 40),
(3, 'Margherita Pizza', 'course', 12.99, 30),
(4, 'Lasagna', 'course', 14.99, 25),
(5, 'Tiramisu', 'dessert', 5.99, 20),
(6, 'Panna Cotta', 'dessert', 6.50, 18);

-- Insert into Menu
INSERT INTO Menu (MenuID, MenuName, Cuisine, MenuItemID) VALUES
(1, 'Italian Classics', 'Italian', 1),
(2, 'Italian Classics', 'Italian', 2),
(3, 'Italian Mains', 'Italian', 3),
(4, 'Italian Mains', 'Italian', 4),
(5, 'Sweet Delights', 'Italian', 5),
(6, 'Sweet Delights', 'Italian', 6);

-- Insert into Bookings
INSERT INTO Bookings (BookingID, BookingDate, TableNumber, CustomerID) VALUES
(1, '2022-10-10', 5, 1),
(2, '2022-11-12', 3, 3),
(3, '2022-10-11', 2, 2),
(4, '2022-10-13', 2, 1);

-- Insert into OrderDeliveryStatus
INSERT INTO OrderDeliveryStatus (OrderDeliveryID, Status, DeliveryDate) VALUES
(1, 'Delivered', '2025-04-08'),
(2, 'Pending', NULL),
(3, 'Delivered', '2025-04-10'),
(4, 'Delivered', '2025-04-11'),
(5, 'Cancelled', NULL);

-- Insert into Orders
INSERT INTO Orders (OrderID, Date, Quantity, TotalCost, MenuID, CustomerID, StaffID, OrderDeliveryID) VALUES
(1, '2025-04-08', 2, 13.98, 1, 1, 1, 1),
(2, '2025-04-09', 3, 38.97, 3, 2, 2, 2),
(3, '2025-04-10', 2, 11.49, 2, 3, 1, 3),
(4, '2025-04-11', 1, 210.00, 4, 4, 3, 4),
(5, '2025-04-12', 2, 250.00, 5, 5, 4, 5);

-- Insert into Customers for anonymous bookings
INSERT INTO Customers (CustomerID, FullName, Email, ContactNumber) VALUES 
(999, 'Guest', 'Guest', 'Guest');
```

### 🔍 Specific statements for tasks

```sql
--- 1.1 Virtual table query
CREATE 
    ALGORITHM = UNDEFINED 
    DEFINER = `db`@`%` 
    SQL SECURITY DEFINER
VIEW `ordersview` AS
    SELECT 
        `orders`.`OrderID` AS `OrderID`,
        `orders`.`Quantity` AS `Quantity`,
        `orders`.`TotalCost` AS `Cost`
    FROM
        `orders`
    WHERE
        (`orders`.`Quantity` > 2)

SELECT * FROM OrdersView;

--- 1.2 JOIN statement
SELECT Customers.CustomerID, Customers.FullName, Orders.OrderID, Orders.TotalCost AS Cost, Menu.MenuName, MenuItems.MenuItemname AS ItemName, MenuItems.Type FROM Orders
INNER JOIN Customers ON Orders.CustomerID = Customers.CustomerID
INNER JOIN Menu ON Orders.MenuID = Menu.MenuID
INNER JOIN MenuItems ON Menu.MenuItemID = MenuItems.MenuItemID
WHERE MenuItems.Type != 'dessert' AND Orders.TotalCost > 150
ORDER BY Cost;

--- 1.3 Subquery
SELECT MenuItems.MenuItemName AS MenuName FROM MenuItems 
INNER JOIN Menu ON Menu.MenuItemID = MenuItems.MenuItemID
WHERE Menu.MenuID = ANY (SELECT MenuId FROM Orders WHERE Quantity > 2);

--- 2.1 Stored procedure(1)
CREATE DEFINER=`db`@`%` PROCEDURE `GetMaxQuantity`()
BEGIN
SELECT MAX(Quantity) AS 'Max Quantity In Order' FROM Orders;
END

CALL GetMaxQuantity();

--- 2.2 Prepared statement
PREPARE GetOrderDetail FROM
'
SELECT OrderID, Quantity, TotalCost AS OrderCost FROM Orders 
WHERE CustomerID = ?
';

SET @id = 1;
EXECUTE GetOrderDetail USING @id;

DEALLOCATE PREPARE GetOrderDetail;

--- 2.3 Stored procedure(2)
CREATE DEFINER=`db`@`%` PROCEDURE `CancelOrder`(IN id INT)
BEGIN
DELETE FROM Orders WHERE OrderID = id;
SELECT CONCAT('Order ', id, ' is cancelled') AS Confirmation;
END

CALL CancelOrder(1);

--- 3.1 Populate Bookings with specific data (done in the first section)
SELECT * FROM Bookings;

--- 3.2 CheckBooking procedure
CREATE DEFINER=`db`@`%` PROCEDURE `CheckBooking`(IN booking_date DATE, IN booking_table INT)
BEGIN

DECLARE booked INT;
SELECT COUNT(BookingID) INTO booked FROM Bookings 
WHERE TableNumber = booking_table AND BookingDate = booking_date;

SELECT IF(
booked > 0, 
CONCAT('Table ', booking_table, ' is already booked'), 
CONCAT('Table ', booking_table, ' is available')
) AS 'Booking status';
END

CALL CheckBooking('2022-11-12', 3);

--- 3.3 AddValidBooking procedure
CREATE DEFINER=`db`@`%` PROCEDURE `AddValidBooking`(IN booking_date DATE, IN booking_table INT)
BEGIN

DECLARE booked INT;
SELECT COUNT(BookingID) INTO booked FROM Bookings 
WHERE TableNumber = booking_table AND BookingDate = booking_date;

START TRANSACTION;

INSERT INTO Bookings(CustomerID, BookingDate, TableNumber) VALUES(999, booking_date, booking_table);

IF booked > 0 THEN
ROLLBACK;
SELECT CONCAT('Table ', booking_table, ' is already booked - booking cancelled') AS 'Booking status';
ELSE
COMMIT;
SELECT CONCAT('Table ', booking_table, ' is available - booking completed') AS 'Booking status';
END IF;

END

CALL AddValidBooking('2022-12-17', 6);

--- 4.1 AddBooking procedure
CREATE DEFINER=`db`@`%` PROCEDURE `AddBooking`(IN booking_id INT, IN customer_id INT, IN table_number INT, IN booking_date DATE)
BEGIN

DECLARE booked INT;
SELECT COUNT(BookingID) INTO booked FROM Bookings 
WHERE BookingID = booking_id;

IF booked > 0 THEN
SELECT 'BookingID is already taken.' AS 'Error';
ELSE
INSERT INTO Bookings(BookingID, BookingDate, TableNumber, CustomerID) VALUES(booking_id, booking_date, table_number, customer_id);
SELECT 'New booking added.' AS 'Confirmation';
END IF;

END

CALL AddBooking(9, 3, 4, '2022-12-30');

--- 4.2 UpdateBooking procedure
CREATE DEFINER=`db`@`%` PROCEDURE `UpdateBooking`(IN booking_id INT, IN booking_date DATE)
BEGIN

DECLARE booked INT;
SELECT COUNT(BookingID) INTO booked FROM Bookings 
WHERE BookingID = booking_id;

IF booked = 0 THEN
SELECT CONCAT('Booking ', booking_id, " doesn't exist") AS 'Error';
ELSE
UPDATE Bookings SET BookingDate = booking_date WHERE BookingID = booking_id;
SELECT CONCAT('Booking ', booking_id, " updated") AS 'Confirmation';
END IF;

END

CALL UpdateBooking(9, '2022-12-17');

-- 4.3 CancelBooking procedure
CREATE DEFINER=`db`@`%` PROCEDURE `CancelBooking`(IN booking_id INT)
BEGIN

DECLARE booked INT;
SELECT COUNT(BookingID) INTO booked FROM Bookings 
WHERE BookingID = booking_id;

IF booked = 0 THEN
SELECT CONCAT('Booking ', booking_id, " doesn't exist") AS 'Error';
ELSE
DELETE FROM Bookings WHERE BookingID = booking_id;
SELECT CONCAT('Booking ', booking_id, " cancelled") AS 'Confirmation';
END IF;

END

CALL CancelBooking(9);
```

## 📊 2. Tableau

Find the data source, Tableau workbook, and png snapshots for every task in the folder: **`2. Tableau`**

## 🐍 3. MySQL Connector Python

Folder: **`3. MySQL Connector Python`** contains:
- Jupyter Notebook file.
- A png snapshot with code outputs.

Remember tu use your own credentials to establish a valid connection.

## ✉️ Contact

- GitHub: https://github.com/impoldev
- LinkedIn: https://www.linkedin.com/in/pabloolle/
- Email: contact@impol.dev