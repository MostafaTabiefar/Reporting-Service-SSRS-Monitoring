# User Acess 
This Query Retrieves Users Acess and their last successful execution.

---
 
```sql
WITH Max_Exe AS
(
    SELECT
        E.UserName,                                     -- Username who executed the report
        E.ReportID,                                     -- ID of the report
        CAST(MAX(E.TimeStart) AS SMALLDATETIME) AS Last_Execution -- Latest execution time for the report
    FROM [dbo].[ExecutionLogStorage] AS E WITH (NOLOCK) -- Query the execution log storage with NOLOCK to avoid locks
	WHERE 1=1
		--AND [Status] = 'rsSuccess'
		AND RequestType <> 2
		AND ReportAction = 1
    GROUP BY E.UserName, E.ReportID                    -- Group by UserName and ReportID to find max execution time
),

-- CTE to retrieve report catalog information along with its type
Report_Catalog AS 
(
    SELECT
         C.ItemID                                      -- Unique identifier for the report item
        ,C.PolicyID                                    -- Policy ID associated with the report
        ,C.Name AS [Report Name]                       -- Name of the report
        ,C.Path AS [Report Path]                       -- Path where the report is located
    FROM dbo.Catalog AS C                              -- Query the Catalog table containing report metadata
)
SELECT
     LOWER(U.UserName) AS [UserName]                   
    ,C.[Report Name]                                   
    ,C.[Report Path]                                                                    
    ,E.Last_Execution                                  
    ,ISNULL(DATEDIFF(DAY, E.Last_Execution, GETDATE()), -1) AS [Date Diff] 
    ,'Access Data' AS [Data Type]                                  -- Static flag to distinguish this query block

FROM dbo.PolicyUserRole AS PU                            
    JOIN dbo.Users AS U                                
        ON PU.UserID = U.UserID 
    JOIN Report_Catalog AS C                           
        ON PU.PolicyID = C.PolicyID
    LEFT JOIN Max_Exe AS E                             
        ON E.ReportID = C.ItemID 
        AND E.UserName = U.UserName

-- UNION combines the above query with additional report execution details 
-- for those users who has acess an item with role but execute by their own username
UNION 

SELECT 
     LOWER(E.UserName) AS [UserName]                   
    ,C.[Report Name]                                   
    ,C.[Report Path]                                                                  
    ,E.Last_Execution                                  
    ,ISNULL(DATEDIFF(DAY, E.Last_Execution, GETDATE()), -1) AS [Date Diff] 
    ,'Execution Data' AS [Data Type]                                  -- Static flag to distinguish this query block

FROM Max_Exe AS E                                      
    JOIN Report_Catalog AS C                          
        ON C.ItemID = E.ReportID     