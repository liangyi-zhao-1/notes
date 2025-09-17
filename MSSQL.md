# Variable
```mssql
DECLARE @RowCount INT;

SELECT @RowCount = COUNT(*)
FROM Employees;

IF @RowCount > 100
    PRINT 'Large table';
ELSE
    PRINT 'Small table';
```
