# Connect to SQL Server

~~~R
sqlConnString <- "Driver=SQLServer;Server=70.12.114.130;Database=VideoShop;uid=sa;pwd=1234;"
~~~

~~~R
connection <- RxSqlServerData(connectionString = sqlConnString,
                              table = "VS_CUSTOMER", rowsPerRead = 1000)
~~~




# Use R functions to examine the data in the VS_CUSTOMER table
head(connection)
rxGetVarInfo(connection)
rxSummary(~., connection)

