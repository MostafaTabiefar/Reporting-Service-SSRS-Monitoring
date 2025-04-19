# Report Server Subscription Monitoring
This query retrieves key details about report subscriptions, including their execution performance, associated report metadata, and subscription status.

---
```sql
SELECT
	   CU.UserName															AS [Created By]
	  ,C.Name																AS [Report Name]
	  ,C.Path																AS [Report Path]
	  ,CASE
			WHEN [LastStatus] LIKE  '%Mail sent to%' OR [LastStatus] LIKE '%has been saved%' OR [LastStatus] = 'Restore shared data source.' OR [LastStatus] = 'Cache refresh succeeded.' THEN 'Succesful'
			WHEN [LastStatus] LIKE '%not valid%' OR [LastStatus] LIKE '%Failure%' OR [LastStatus] LIKE '%Error%' THEN 'Error'
			WHEN [LastStatus] = 'Running'	THEN 'Running'
			WHEN [LastStatus] = 'Disabled' OR [LastStatus] LIKE '%not enabled%' THEN 'Disabled'
			ELSE 'Unknown/Other'
			END										                        AS [Last Status]
	  , [LastStatus] AS [Last Status Detail]
      ,[EventType]
      ,CAST([LastRunTime] AS smalldatetime)					AS [LastRunTime]
FROM [dbo].[Subscriptions] AS S
	LEFT JOIN dbo.Users AS CU ON CU.UserID = OwnerID
	LEFT JOIN dbo.catalog AS C ON C.ItemID = S.Report_OID 

