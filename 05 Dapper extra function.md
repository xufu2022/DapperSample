# Dapper Extra Function

## Dapper WHERE IN Parameters

```cs
var ids = new[] { 3, 7, 12 };
var sql = "SELECT * FROM Products WHERE ProductId IN @Ids;";
using (var connection = new SqlConnection(connectionString))
{
	connection.Open();
	var products = connection.Query<Product>(sql, new {Ids = ids }).ToList();
}
```
## Dapper Table-Valued Parameters
```sql
CREATE TYPE TvpExampleType AS TABLE 
(
    ID INT, 
    Name VARCHAR(50)
)

CREATE PROCEDURE dbo.MyStoredProc 
    (@TvpExampleType TvpExampleType READONLY) 
AS 
BEGIN 
    --Do something 
END

```
```cs
var tvpExampleType = new List<TvpExampleType>(); 
tvpExampleType.Add(new TvpExampleType {ID = 1, Name = "John" }); 
tvpExampleType.Add(new TvpExampleType {ID = 2, Name = "Steve" }); 
 
var p = new DynamicParameters(); 
p.Add("TvpExampleType", tvpExampleType.AsTableValuedParameter("dbo.TvpExampleType")); 
await connection.ExecuteAsync("dbo.MyStoredProc", p, commandType: CommandType.StoredProcedure);
```
## Dapper Output Parameter

Dapper provides the ability to use output parameters in stored procedures and parameterized queries, allowing for greater control over the data being returned.

- Output parameters are declared using the same syntax as input parameters but with an additional OUTPUT keyword.
- When declaring an output parameter, it must be given a name (the "parameter name"), an optional data type, and an optional size.
- To use dapper output parameters, you must first create a DynamicParameters object in your code and then add it as a parameter to your query.

```sql
Create Procedure GetCustomerDetails
   @CustomerID          INT,
   @Name                NVARCHAR(Max)         OUTPUT,
   @Description         NVARCHAR(Max)         OUTPUT
AS
   SELECT
      @Name=Name,
      @Description=Description FROM Customers
   WHERE CustomerID=@CustomerID
```
```cs
using (var connection = new SqlConnection(connectionString))
{
    var p = new DynamicParameters();
    p.Add("@CustomerID", 1002);
    p.Add("@Name", null, dbType: DbType.String, direction: ParameterDirection.Output, 50);
    p.Add("@Description", null, dbType: DbType.String, direction: ParameterDirection.Output,50);
    
    connection.Execute("GetCustomerDetails", p, commandType: CommandType.StoredProcedure);
    
    var name = p.Get<string>("@Name");
    var description = p.Get<string>("@Description");
}
```