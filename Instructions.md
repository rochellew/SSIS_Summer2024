# CSCI 3020 - Working with ETL 

## Checklist
- [ ] Create CSV data files
- [ ] Create database / required tables
- [ ] Create SSIS package
- [ ] Obtain screenshots of successful processes 

### Creating CSVs

- Create the following two CSV files, using the provided data (you may include more data if you'd like -- these are the minimum).

*customers.csv*
```
CustomerID,FirstName,LastName,Email
1,John,Doe,john.doe@email.com
2,Jane,Smith,jane.smith@email.com
3,Bob,Johnson,bob.johnson@email.com
```

*orders.csv*
```
OrderID,CustomerID,OrderDate,TotalAmount
101,1,2023-01-15,150.00
102,2,2023-01-16,200.00
103,1,2023-01-17,75.50
104,3,2023-01-18,300.00
```
### Creating Database/Tables
- Create the destination database:
    - Open SSMS and connect to your SQL Server instance.
    - Create a new query on your instance
    - Execute the following SQL Script

```SQL
CREATE DATABASE SalesDB;
GO

USE SalesDB;
GO

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    Email NVARCHAR(100),
    FullName NVARCHAR(101)
);

CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE,
    TotalAmount DECIMAL(10, 2),
    FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);
```
### Create SSIS Project

> :warning: **Make sure you have installed the proper extension as listed on D2L.**
#### Creating the Project
1. Open Visual Studio **as an administrator**.
2. Create a new *Integration Services Project* called "SalesETL"

#### Connection Managers
1. Create a new Flat File Connection Manager by right-clicking the Connection Managers area and choosing ```New Flat File Connection...```
    - Change the default name to "Customer File Connection Manager" in the *Connection Manager Name* box at the top of the window
    - Click 'Browse' and locate *customers.csv*
      - You may need to change the file type from *.txt* to *.csv*
    - Navigate to the *Columns* panel on the left-side of the window
    - Select *Comma {,}* from the Column Delimiter dropdown
    - Click the *Refresh* button and verify the data looks correct in the preview window
    - Click *OK*

2. Create a new Flat File Connection Manager by right-clicking the Connection Managers area and choosing ```New Flat File Connection...```
    - Change the default name to "Order File Connection Manager" in the *Connection Manager Name* box at the top of the window
    - Click 'Browse' and locate *orders.csv*
      - You may need to change the file type from *.txt* to *.csv*
    - Navigate to the *Columns* panel on the left-side of the window
    - Select *Comma {,}* from the Column Delimiter dropdown
    - Click the *Refresh* button and verify the data looks correct in the preview window
    - Click *OK*

>:warning: **You may experience some issues with  step 3 if you are not running Visual Studio as an Administrator.**

3. Create a new OLE DB Connection Manager by right-clicking the Connection Managers area and choosing ```New OLE DB Connection```
    - You will see an empty window here -- click 'New', followed by 'OK'
    - Change the provider to ```Microsoft OLE DB Provider for SQL Server```in the Provider dropdown
    - Click the arrow on the Server Name dropdown, and select your SQL Server instance
      - It may take a few seconds to sync your server names
    - Select the *SalesDB* database from the database dropdown
    - Click on *Test Connection* to verify you performed this section correctly, then OK when the connection is successful.
    - If you would like to rename this connection manager to something else, you can right-click on it, navigate to 'Properties', then change the name in the window that appears.


#### *Customers* Data Flow
##### Customers Task
1. Drag a *Data Flow* task from the Toolbox (located in Favorites) onto the Control Flow design surface from the SSIS toolbox.
    - Right-click, navigate to 'Properties', then rename this task to 'Customer Task' in the Properties window.
2. Double-click Customer Task to open the Data Flow Designer.
##### Flat File Source
1. Drag a **Flat File Source* from the Toolbox (located in 'Other Sources') into the Data Flow Designer.
2. Double-click this source, and choose *Customer Connection Manager* from the dropdown.
3. Navigate to the 'Columns' page on the left side of  the window and verify that the External (columns from the csv) columns are all present.
4. Click 'OK'.
##### Data Conversion
We will need to convert the CSV data into Unicode that SQL  Server can accept. This is an example of one of the 'Transformations' involved in ETL.
1. Drag a *Data Conversion* from the Toolbox (located in 'Common') into the Data Flow Designer.
2. In the Data Flow Designer, click on your Flat File Source, and you will see a blue arrow underneath it. Drag this arrow to the Data Conversion. This links these components together, allowing the conversion component to accept the data from the file.
3. Double-click the Data Conversion to open its settings. We want to change the string columns (```FirstName```, ```LastName```, and ```Email```) into Unicode which can be stored as an ```NVARCHAR``` in our database. Select these three columns from the 'Available Input Columns'. This will allow us to convert them prior to loading them into the database.
4. Below  where you selected the columns, change the data type of **all three** from ```string [DT-STR]``` to ```Unicode string [DT_WSTR]```.  
5. Click 'OK'.

##### OLE DB Destination
1. Drag an *OLE DB Destination* from the Toolbox (located in 'Other Destinations') into the Data Flow Designer.
2. Click the Data Conversion, and drag the blue arrow to the OLE DB Destination.
3. Double-click this destination, and choose the Customers table from the 'Name of the table or view' dropdown.
4. Navigate to *Mappings* on the left side of this window. This will allow us to map our input columns (from the csv) to the output columns for insertion into the database. If the names are the same (such as in this lab), you don't have to change much here.
5. Map the converted ```FirstName```, ```LastName```, and ```Email``` columns from the Data Conversion to the appropriate output columns.
   -  There will be default values here; e.g., the converted ```FirstName``` column will be called ```Copy of FirstName``` if you didn't change anything in the Data Conversion window.
6. Click 'OK'.
---
By this point, we have created one data flow. If you run this package in Visual Studio, it will pull the data from the CSV source, convert the strings to Unicode, and then load the data into the SQL Server database. we aren't finished yet, but feel free to test that this is working, as we'll be duplicating this for the orders data shortly. Note that you will likely need to delete the data from the *Customers* table every time you run this package, or you run the risk of insertion anomalies.
 ```SQL
    DELETE FROM CUSTOMERS;
``` 

Navigate back to the Control Flow panel at the top of the screen before moving on.

Note that the next section is almost the same as the previous one in terms of steps taken.

---
#### *Orders* Data Flow
##### Orders Task
1. Drag a *Data Flow* task from the Toolbox (located in Favorites) onto the Control Flow design surface from the SSIS toolbox.
    - Right-click, navigate to 'Properties', then rename this task to 'Order Task' in the Properties window.
2. Double-click Order Task to open the Data Flow Designer.
##### Flat File Source
1. Drag a *Flat File Source* from the Toolbox (located in 'Other Sources') into the Data Flow Designer.
2. Double-click this source, and choose *Order Connection Manager* from the dropdown.
3. Navigate to *Columns* on the left side of this window and verify that the External (columns from the CSV) columns are all present.
4. Click 'OK'.
##### Data Conversion
We will need to convert the CSV data into Unicode that SQL  Server can accept. This is an example of one of the 'Transformations' involved in ETL.
1. Drag a *Data Conversion* from the Toolbox (located in 'Common') into the Data Flow Designer.
2. In the Data Flow Designer, click on your Flat File Source, and you will see a blue arrow underneath it. Drag this arrow to the Data Conversion. 
3. Double-click the Data Conversion to open its settings. We want to change the string column (```OrderDate```) into Unicode which can be stored as an ```DATE``` in our database. Select the ```OrderDate``` column from the 'Available Input Columns'. This will allow us to convert it prior to loading it into the database.
4. Below  where you selected the column, change the data type from ```string [DT-STR]``` to ```database date [DT_DBDATE]```.  
5. Click 'OK'.

##### OLE DB Destination
1. Drag an *OLE DB Destination* from the Toolbox (located in 'Other Destinations') into the Data Flow Designer.
2. Click the Data Conversion, and drag the blue arrow to the OLE DB Destination.
3. Double-click this destination, and choose the Orders table from the 'Name of the table or view' dropdown.
4. Navigate to *Mappings* on the left side of this window. This will allow us to map our input columns (from the CSV) to the output columns for insertion into the database. If the names are the same (such as in this lab), you don't have to change much here.
5. Map the converted ```OrderDate``` column from the Data Conversion to the appropriate output column.
   -  There will be a default value here; e.g., the converted ```OrderDate``` column will be called ```Copy of OrderDate``` if you didn't change anything in the Data Conversion window.
6. Click 'OK'.

---
The hard part is OVER!!! We have one more step before we can run the entire package. 

---

#### Precedence Constraint
If you recall from the SQL we wrote when we created the *Customers* and *Orders* tables, *Orders* has a foreign key that points to *Customers*. As such, we want to make sure ETL is loading *Customers* BEFORE *Orders*, to prevent any insertion anomalies.
1. Navigate back to the Control Flow panel at the top of the screen.
2. Click on the Customer Task.
3. Drag the green arrow from Customer Task to Order Task.

### Runnin' the Dang Thing
#### ...and gettin' them screenshots.
All the legwork is done -- all that's left is to run the package, and obtain screenshots of its success.
1. Run the package (in or out of Debug mode -- that doesn't really matter here). You should receive a message in the command prompt like the one shown below if your package was successful.
```
DTExec: The package execution returned DTSER_SUCCESS (0).
Started:  12:25:17 AM
Finished: 12:25:18 AM
Elapsed:  0.625 seconds
```
2. Take a screenshot of this portion of your command prompt and paste it into a Word document. 
3. Run the following SQL queries (**after confirming the package runs successfully**) and paste screenshots of their results into the Word document.

> :warning: If you want to rerun this package several times, you'll need to clear the database out first (assuming the data was inserted successfully). **Drop *Orders* first, then *Customers* to honor to the foreign key constraints**.

```SQL
SELECT * FROM CUSTOMERS;
```

```SQL
SELECT * FROM ORDERS;
```
4. **Convert the Word document to a PDF** and upload it to the DropBox by the specified due date on D2L. 