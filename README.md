# 26030_Cindy_groupB  

ID:26030
Name:ANOUMANTAKY AKESSE CINDY 
Concentration: Software Engineering

  Problem statement
  This project is a Tour Package Booking System 🧳, designed to manage tour packages, customers, bookings, agents, and payments efficiently. The system streamlines the entire travel booking process—from package selection to final payment—ensuring quick confirmations, real-time updates, and accurate tracking of customer transactions.

🛠️ SQL Commands Description

✅ Database Schema Design

Created four related tables:
 Customers Table
SQL> CREATE TABLE Customers (
  2      CustomerID INT PRIMARY KEY,
  3      FullName VARCHAR2(100) NOT NULL,
  4      Email VARCHAR2(100) UNIQUE NOT NULL,
  5      Phone VARCHAR2(20)
  6  );

Table created.

SQL>
SQL> -- TourPackages Table
SQL> CREATE TABLE TourPackages (
  2      PackageID INT PRIMARY KEY,
  3      PackageName VARCHAR2(100) NOT NULL,
  4      Destination VARCHAR2(100) NOT NULL,
  5      Price DECIMAL(10,2) NOT NULL,
  6      StartDate DATE NOT NULL
  7  );

Table created.

SQL>
SQL> -- TravelAgents Table
SQL> CREATE TABLE TravelAgents (
  2      AgentID INT PRIMARY KEY,
  3      AgentName VARCHAR2(100) NOT NULL,
  4      Email VARCHAR2(100) UNIQUE,
  5      Phone VARCHAR2(20)
  6  );

Table created.

SQL>
SQL> -- Bookings Table
SQL> CREATE TABLE Bookings (
  2      BookingID INT PRIMARY KEY,
  3      CustomerID INT,
  4      PackageID INT,
  5      AgentID INT,
  6      BookingDate DATE DEFAULT SYSDATE,
  7      Status VARCHAR2(20) DEFAULT 'Pending',
  8      FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID),
  9      FOREIGN KEY (PackageID) REFERENCES TourPackages(PackageID),
 10      FOREIGN KEY (AgentID) REFERENCES TravelAgents(AgentID)
 11  );

Table created.

SQL>
SQL> -- Payments Table
SQL> CREATE TABLE Payments (
  2      PaymentID INT PRIMARY KEY,
  3      BookingID INT,
  4      PaymentDate DATE DEFAULT SYSDATE,
  5      Amount DECIMAL(10,2),
  6      PaymentStatus VARCHAR2(20) DEFAULT 'Pending',
  7      FOREIGN KEY (BookingID) REFERENCES Bookings(BookingID)
  8  );

Table created.

✅ Data Insertion & Manipulation

SQL> -- Insert Customers
SQL> INSERT INTO Customers VALUES (1, 'Alice Johnson', 'alice@example.com', '0789123456');

1 row created.

SQL> INSERT INTO Customers VALUES (2, 'Bob Smith', 'bob@example.com', '0789234567');

1 row created.

SQL> INSERT INTO Customers VALUES (3, 'Claire Adams', 'claire@example.com', '0789345678');

1 row created.

SQL>
SQL> -- Insert Tour Packages
SQL> INSERT INTO TourPackages VALUES (101, 'Safari Adventure', 'Kenya', 1200.00, TO_DATE('2025-07-10', 'YYYY-MM-DD'));

1 row created.

SQL> INSERT INTO TourPackages VALUES (102, 'Beach Relaxation', 'Maldives', 2500.00, TO_DATE('2025-08-15', 'YYYY-MM-DD'));

1 row created.

SQL> INSERT INTO TourPackages VALUES (103, 'European Tour', 'France, Italy, Spain', 3200.00, TO_DATE('2025-09-01', 'YYYY-MM-DD'));

1 row created.

SQL>
SQL> -- Insert Travel Agents
SQL> INSERT INTO TravelAgents VALUES (201, 'Daniel Kamanzi', 'daniel@agency.com', '0789001122');

1 row created.

SQL> INSERT INTO TravelAgents VALUES (202, 'Emma Uwase', 'emma@agency.com', '0789001133');

1 row created.

SQL>
SQL> -- Insert Bookings
SQL> INSERT INTO Bookings VALUES (301, 1, 101, 201, SYSDATE, 'Confirmed');

1 row created.

SQL> INSERT INTO Bookings VALUES (302, 2, 102, 202, SYSDATE, 'Pending');

1 row created.

SQL> INSERT INTO Bookings VALUES (303, 3, 103, 201, SYSDATE, 'Confirmed');

1 row created.

SQL>
SQL> -- Insert Payments
SQL> INSERT INTO Payments VALUES (401, 301, SYSDATE, 1200.00, 'Completed');

1 row created.

SQL> INSERT INTO Payments VALUES (402, 302, SYSDATE, 2500.00, 'Pending');

1 row created.

SQL> INSERT INTO Payments VALUES (403, 303, SYSDATE, 3200.00, 'Completed');

1 row created.

✅ Procedure to Create a New Booking

CREATE OR REPLACE PROCEDURE sp_create_booking (
    p_CustomerID IN INT,
    p_PackageID IN INT,
    p_AgentID IN INT
) AS
BEGIN
    INSERT INTO Bookings (BookingID, CustomerID, PackageID, AgentID, BookingDate, Status)
    VALUES (BOOKING_SEQ.NEXTVAL, p_CustomerID, p_PackageID, p_AgentID, SYSDATE, 'Pending');

    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        DBMS_OUTPUT.PUT_LINE('Error creating booking: ' || SQLERRM);
END;
/
SQL> CREATE SEQUENCE BOOKING_SEQ
  2  START WITH 304
  3  INCREMENT BY 1
  4  NOCACHE
  5  NOCYCLE;
Sequence created.
SQL> BEGIN
  2      sp_create_booking(1, 101, 201);
  3  END;
  4  /

PL/SQL procedure successfully completed.


✅FUNCTION


SQL> CREATE OR REPLACE FUNCTION fn_calculate_total_payment (
  2      p_BookingID IN INT
  3  ) RETURN DECIMAL IS
  4      v_Amount DECIMAL(10,2);
  5  BEGIN
  6      SELECT Amount
  7      INTO v_Amount
  8      FROM Payments
  9      WHERE BookingID = p_BookingID;
 10
 11      RETURN v_Amount;
 12  EXCEPTION
 13      WHEN NO_DATA_FOUND THEN
 14          RETURN 0;
 15      WHEN OTHERS THEN
 16          RETURN -1;
 17  END;
 18  /

Function created.

✅ Cursor to Update All Pending Bookings to Confirmed

DECLARE
    CURSOR c_pending_bookings IS
        SELECT BookingID FROM Bookings WHERE Status = 'Pending';

    v_BookingID Bookings.BookingID%TYPE;
BEGIN
    OPEN c_pending_bookings;
    LOOP
        FETCH c_pending_bookings INTO v_BookingID;
        EXIT WHEN c_pending_bookings%NOTFOUND;

        UPDATE Bookings
        SET Status = 'Confirmed'
        WHERE BookingID = v_BookingID;
    END LOOP;
    CLOSE c_pending_bookings;

    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        DBMS_OUTPUT.PUT_LINE('Error updating bookings: ' || SQLERRM);
END;
/
PL/SQL procedure successfully completed.

✅ Full Package for Booking Management

SQL> CREATE OR REPLACE PACKAGE BODY pkg_booking_management IS
  2
  3      PROCEDURE sp_create_booking(p_CustomerID INT, p_PackageID INT, p_AgentID INT) IS
  4      BEGIN
  5          INSERT INTO Bookings (BookingID, CustomerID, PackageID, AgentID, BookingDate, Status)
  6          VALUES (BOOKING_SEQ.NEXTVAL, p_CustomerID, p_PackageID, p_AgentID, SYSDATE, 'Pending');
  7
  8          COMMIT;
  9      EXCEPTION
 10          WHEN OTHERS THEN
 11              ROLLBACK;
 12              DBMS_OUTPUT.PUT_LINE('Error creating booking: ' || SQLERRM);
 13      END sp_create_booking;
 14
 15      FUNCTION fn_calculate_total_payment(p_BookingID INT) RETURN DECIMAL IS
 16          v_Amount DECIMAL(10,2);
 17      BEGIN
 18          SELECT Amount
 19          INTO v_Amount
 20          FROM Payments
 21          WHERE BookingID = p_BookingID;
 22
 23          RETURN v_Amount;
 24      EXCEPTION
 25          WHEN NO_DATA_FOUND THEN
 26              RETURN 0;
 27          WHEN OTHERS THEN
 28              RETURN -1;
 29      END fn_calculate_total_payment;
 30
 31  END pkg_booking_management;
 32  /

Package body created.

✅Calculate Total Payment for a Booking Using the Function

-- Suppose we want to calculate the amount for BookingID = 301
DECLARE
    v_TotalAmount DECIMAL(10,2);
BEGIN
    v_TotalAmount := pkg_booking_management.fn_calculate_total_payment(301);
    DBMS_OUTPUT.PUT_LINE('Total Payment Amount: ' || v_TotalAmount);
END;
/

✅TIGGER

SQL> INSERT INTO Payments (PaymentID, BookingID, PaymentDate, Amount, PaymentStatus)
  2  VALUES (404, 301, SYSDATE, 1200.00, 'Completed');

1 row created.

SQL> COMMIT;

Commit complete.  

🖼️ Screenshots & ER Diagram





