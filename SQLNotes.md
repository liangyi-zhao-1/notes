# SQL notes
## When to use composite index
1. need to see the distribution of the data
2. need to look the index whether it worth

## when the index is useful
- if the main search param is the index
  
## Use UNION ALL when query has OR

## When query with optional params
1. Separate each param to one UNION ALL
2. Use OPTION(RECOMPILE)


## Date
### Add Date
```SQL
-- MySQL
DATE_ADD('2025-10-11', INTERVAL 1 DAY)
DATE_SUB('2025-11-12', INTERVAL 1 DAY)
-- MS SQL
DATEADD(DAY, -1, '2025-11-12')
```

## Rounding
```SQL
ROUND(123.45, -2) --100.00
```

1. Column names must be PascalCased – not all lower case
```sql
Platform.Jurisdiction_Deu_Banking.ReservationsTransactionTypes
Platform.DBACustomer.GstVatDetails
```
2. Multi-table queries: Columns must be prefixed with the table they come from
3. Parameter sizes should match source tables
4. CSV Parsers - Use the ones in CSFBMaster. If you need the results of an input variable multiple times, put them in a temp table or table variable and reuse - DO NOT parse the value multiple times. Do not use the CSV parsers in an EXISTS/NOT EXISTS/IN/NOT IN statement. Maximum of 1 parser should be called per query.
```sql
RETURNS TABLE AS
    RETURN
    (
        WITH E1(N) AS
        (
            SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL
            SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL
            SELECT 1 UNION ALL SELECT 1 UNION ALL SELECT 1 UNION ALL
            SELECT 1
        ),
        E2(N) AS (SELECT 1 FROM E1 CROSS JOIN E1 AS B),
        E3(N) AS (SELECT 1 FROM E2 CROSS JOIN E2 AS B),
        E4(N) AS (SELECT 1 FROM E3 CROSS JOIN (SELECT TOP 500 N FROM E3) AS B),
        T(N)  AS (SELECT 0 UNION ALL SELECT TOP (DATALENGTH(ISNULL(@ListValues, 1))) ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) FROM E4),
        S(N)  AS (SELECT T.N + 1 FROM T WHERE (SUBSTRING(@ListValues, T.N, 1) = ',' OR T.N = 0))

        SELECT DISTINCT CAST(SUBSTRING(@ListValues, S.N, ISNULL(NULLIF(CHARINDEX(',', @ListValues, s.N), 0) - s.N, 8000)) AS INT) AS TabValue FROM S
        WHERE @ListValues <> '' AND @ListValues IS NOT NULL
    )
```
5. Avoid Optional parameters on Stored procedures. Breaking up into multiple procedures. Or, turn it into a view
6. If less than 100 rows, then use a table variable. If more than 100  rows use a temp table (make sure they're indexed)
```sql
DECLARE @ThresholdDate AS DATE = GETDATE()-90

SELECT ClAccountID
INTO #ClosedAccounts
FROM Platform.Accounts.ClientAccountStatusHistory 
WHERE ToStatus = 'Closed' AND ChangeDate > @ThresholdDate

CREATE UNIQUE CLUSTERED INDEX CUIDX_ClAccountId ON #ClosedAccounts(ClAccountId)

DECLARE @ClosedAccounts AS TABLE
(ClAccountId VARCHAR(20) NOT NULL PRIMARY KEY)

INSERT INTO @ClosedAccounts
SELECT ClAccountID
FROM Platform.Accounts.ClientAccountStatusHistory 
WHERE ToStatus = 'Closed' AND ChangeDate > @ThresholdDate
```
7. Right way to get Head Account. Use the consolidate table or fnHeadAccounts function.
```sql
SELECT                 
head.ClAccountID AS HeadClAccountId, 
sub.ClAccountID AS SubClAccountId 
FROM ClientAccount.dbo.SeClientAccount sub 
INNER JOIN ClientAccount.dbo.Consolidate c ON c.SubClAccountID = sub.ClAccountID
INNER JOIN ClientAccount.dbo.ClientDetails cd ON cd.ClAccountId = c.ClAccountId AND cd.InvestorType <> 'Consolidated'
INNER JOIN ClientAccount.dbo.SeClientAccount head ON Head.ClAccountID = c.ClAccountID
WHERE c.ClAccountID <> c.SubClAccountID

SELECT 
ha.HeadClAccountID AS HeadClAccountId, 
sub.ClAccountID AS SubClAccountId 
FROM ClientAccount.dbo.SeClientAccount sub 
INNER JOIN ClientAccount.dbo.fnHeadAccounts() ha ON ha.ClAccountID = sub.ClAccountID
WHERE ha.Consolidated = 0

 --OR If you already have Head Account
DECLARE @HeadClAccountId AS VARCHAR(20) 
SET @HeadClAccountId = 'HA1000004'

SELECT  
ha.ClAccountID AS SubClAccountId 
FROM ClientAccount.dbo.fnHeadAccounts() ha 
WHERE ha.Consolidated = 0 AND ha.HeadClAccountID = @HeadClAccountId
```
8. New table design
    * Indexes: If there’s no indexes on a new table, review what is reading it. You are likely missing indexes to support the reads.  If you are adding index to Nullable column then index should be filtered. For example, if a column is nullable, the index that uses that column should only include rows where the value is not null.
    ```sql
    CREATE NONCLUSTERED INDEX IDX_TradeId 
    ON CashLedger.OrderTransactions (TradeId) 
    WHERE TradeId IS NOT NULL
    ```
    * Good names: Don't name columns: ID, Value - or other extremely generic names.  Be descriptive.
9. Any stored procedure that returns account or client data needs permission checks.
```sql
DECLARE    
@UserId INT = 500020,
@AccountId INT= 3002

SELECT
    sca.Id AS AccountId,
    sca.ClAccountId AS ClAccountId

FROM ClientAccount.dbo.SEClientAccount sca 
-- JOIN ON Other tables

WHERE SCA.Id = @AccountId
    AND    EXISTS (
        -- verify adv permissions
        SELECT 1
        FROM ClientAccount.dbo.ClientAccountSecurity CAS
        WHERE CAS.AdvisorCodes = SCA.PrimaryAdviser
            AND CAS.ClAccountId IS NULL
            AND CAS.ClientId = @UserId      
 
        UNION ALL
 
        -- verify direct permissions
        SELECT 1
        FROM ClientAccount.dbo.ClientAccountSecurity CAS
        INNER JOIN ClientAccount.dbo.Consolidate CON
            ON CON.ClAccountID = CAS.ClAccountID
        WHERE Con.SubClAccountID = sca.ClAccountID
            AND CAS.ClientId = @UserId
 
        UNION ALL
        -- verify SystemUser
        SELECT 1
        WHERE @UserID = 8245

    )
```
10. When we use a function on an indexed column in the WHERE clause, it will usually prevent that index from being used, which results in a poorer query performance.
```sql
SELECT 
ST.AsAt,
ST.ClAccountId,
ST.InstrumentCode,
ST.Quantity 
FROM ClientAccount.dbo.ScripTransactions ST
WHERE DATEDIFF(DAY,ST.AsAt,GETDATE()) <= 10

SELECT 
ST.AsAt,
ST.ClAccountId,
ST.InstrumentCode,
ST.Quantity 
FROM ClientAccount.dbo.ScripTransactions ST
WHERE ST.AsAt >= CAST(GETDATE() - 10 as date)
```
11. When comparing values in SQL Server, the values must be of the same data type. If they are not, then SQL Server will either internally convert one of the values to match the data type of the other (implicit conversion) or throw an error. When an indexed column is converted to another data type, the corresponding index cannot be effectively utilized and therefore results in poorer performance.
12. Unnecessary use of DISTINCT. There is a performance cost to the use of DISTINCT because the database needs to compare all the rows to each other to make sure there are no repeating rows. Investigate why duplicate rows are returned, by checking each join clause. It's worth noting that if you need to add window functions (CTE) to your view, it is complex enough to be made a stored procedure. Use of DISTINCT in a view is a warning sign that your query is incomplete or you might be joining on too many tables with not enough conditions. There should be no aggregation in views - this is any MIN(), MAX(), SUM(), COUNT(),AVG() functions and more.
13. Interchanging usage of UNION vs UNION ALL
    * Issue 1: using UNION to combine multiple SELECT statements even when no duplicates are expected in the result-set.
        * Solution: use UNION ALL instead.
    * Issue 2: using UNION to combine multiple SELECT statements in order to remove duplicates from the result-set.
        * Solution: ensure the SELECT statements contain data that are mutually exclusive (i.e. a data row only exists in one of the SELECT statements, not in multiple).
14. There can only be 1 clustered index on a SQL table         
