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
# Group by, Average, Minimum
```mssql
;WITH TMP AS (
	SELECT 1 AS VAL, 'A' AS TP, 10 AS DV UNION ALL
	SELECT 2 AS VAL, 'A' AS TP, 10 AS DV UNION ALL
	SELECT 3 AS VAL, 'A' AS TP, 10 AS DV UNION ALL
	SELECT 4 AS VAL, 'B' AS TP, 10 AS DV UNION ALL
	SELECT 5 AS VAL, 'B' AS TP, 10 AS DV 
)
SELECT AVG(CAST((TMP.VAL ) AS DECIMAL)/ TMP.DV),
	AVG(TMP.VAL) AVERAGE,
	AVG(CAST(TMP.VAL AS DECIMAL)) DECIMALAVERAGE,
    MIN(VAL) MINIMUM
FROM TMP
GROUP BY TMP.TP
```

# Date
```mssql
SELECT GETDATE()

SELECT DATEDIFF(MONTH, 0, GETDATE())

SELECT DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0)

SELECT FORMAT(GETDATE(), 'yyyy-MM')

-- GET NEXT DATE IN TABLE
;WITH FirstLogin AS (
    SELECT MIN(A.event_date) FirstLoginDate,
        A.player_id
    from Activity A
    GROUP BY A.player_id 
), NextDayLogin AS (
    SELECT FL.FirstLoginDate
    FROM FirstLogin FL
    INNER JOIN Activity A ON FL.player_id = A.player_id AND A.event_date = DATEADD(DAY, 1, FL.FirstLoginDate)
)
--SELECT COUNT(*) FROM NextDayLogin
SELECT ROUND((SELECT CAST(COUNT(*) AS DECIMAL) FROM NextDayLogin) / (SELECT COUNT(*) FROM FirstLogin), 2) AS fraction 
```

# Partition
```mssql
SELECT customer_id ,
    CASE WHEN order_date = customer_pref_delivery_date THEN 1.0 ELSE 0.0 END IsImmediate,
    ROW_NUMBER() OVER(PARTITION BY D.customer_id ORDER BY D.order_date  ASC) AS RowNumber
FROM Delivery D 
```
