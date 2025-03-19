# Merge
```sql
SELECT TOP 100 *
FROM Platform.Validation.MandatoryFields
WHERE 1=1
--AND MANDATORYFIELDKEY LIKE '%CreateCustomerInstructionWithCustomerResponseCommand%'
ORDER BY 1 DESC

MERGE Platform.Validation.MandatoryFields AS T
USING (
	SELECT 'CustomerInstruction.PersonalDetails.Name' AS MandatoryFieldKey, 1 AS IsEnabled UNION ALL
	SELECT 'CustomerInstruction.PersonalDetails.Countries', 1 UNION ALL
	SELECT 'CustomerInstruction.PersonalDetails.DateOfBirth', 1 
) AS S
ON T.MandatoryFieldKey = S.MandatoryFieldKey
WHEN NOT MATCHED THEN
	INSERT(MandatoryFieldKey,IsEnabled)
	VALUES(S.MandatoryFieldKey,S.IsEnabled)
WHEN MATCHED THEN UPDATE SET
	T.IsEnabled = S.IsEnabled;
```
