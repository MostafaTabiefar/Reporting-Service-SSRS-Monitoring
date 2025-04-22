# Report Server Resource Consuming Reports
This query retrieves reports with their average returning row count and average loading duration

---
```sql
SELECT 
	   C.Name																												AS [Report Name]
	   ,C.Path																												AS [Report Path] 
     -- ,[UserName]																											AS [User Name] -- Users With Resource Consuming Parameters		
	  ,CAST(AVG(TRY_CAST((TimeDataRetrieval + TimeProcessing + TimeRendering) AS BIGINT))/60000.00 AS smallmoney)           AS Avg_LoadTime_Minute
	  ,AVG([RowCount])													AS Avg_Row_Count
FROM [dbo].ExecutionLogStorage AS E WITH(NOLOCK)
	LEFT JOIN [dbo].Catalog AS C WITH(NOLOCK) 
	ON (E.ReportID = C.ItemID)
WHERE 1=1
	AND [Status] = 'rsSuccess' 
	AND RequestType <> 2
	AND ReportAction = 1
	--AND CAST(CONVERT(NVARCHAR(8),[TimeStart], 112) AS INT) BETWEEN start AND end

GROUP BY C.Name, C.Path
ORDER BY 3 DESC, 4 DESC

