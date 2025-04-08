SQL QUERIES

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

INSERT INTO Bookings (BookingID, Date, People, TableNumber, BookingName) VALUES
(1, '2025-04-08', 2, 5, 'Alice Smith'),
(2, '2025-04-09', 4, 2, 'Bob Johnson'),
(3, '2025-04-10', 3, 6, 'Carla Gomez'),
(4, '2025-04-11', 1, 1, 'David Lee'),
(5, '2025-04-12', 5, 3, 'Emma Davis');

-- Insert into OrderDeliveryStatus

INSERT INTO OrderDeliveryStatus (OrderDeliveryID, Status, DeliveryDate) VALUES
(1, 'Delivered', '2025-04-08'),
(2, 'Pending', NULL),
(3, 'Delivered', '2025-04-10'),
(4, 'Delivered', '2025-04-11'),
(5, 'Cancelled', NULL);

-- Insert into Orders

INSERT INTO Orders (OrderID, Date, Quantity, TotalCost, MenuID, CustomerID, StaffID, OrderDeliveryID, BookingID) VALUES
(1, '2025-04-08', 2, 13.98, 1, 1, 1, 1, 1),
(2, '2025-04-09', 3, 38.97, 3, 2, 2, 2, 2),
(3, '2025-04-10', 2, 11.49, 2, 3, 1, 3, 3),
(4, '2025-04-11', 1, 210.00, 4, 4, 3, 4, 4),
(5, '2025-04-12', 2, 250.00, 5, 5, 4, 5, 5);

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

--- 1.3 Subquery statement

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