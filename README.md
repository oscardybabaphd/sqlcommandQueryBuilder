
> Install-Package SQLQueryCommandBuilder -Version 1.0.1

[Bitbucket Docs](https://bitbucket.org/sql-commandquery-builder/sqlquerycommandbuilder/src/master/README.md)

```csharp
  public class user
    {
        public user()
        {

        }
        [SQLResolver(PrimaryKeyName = "id")] //first position
       // [Required]
        public int id { get; set; }
        public int age { get; set; }
        public string name { get; set; }
        public bool isActive { get; set; }
    }
```
`Primary key must have SQLResolver Attribute with the property name as PrimaryKeyName value.: Note(the attribut should come first before any othe attributes)`

SQL Query Resolver Extension Methods
---
```csharp
        /// <summary>
        /// <c>Generate select statement with options</c>
        /// </summary>
        /// <typeparam name="T">type</typeparam>
        /// <param name="obj">type object</param>
        /// <param name="options">sort order</param>
        /// <returns><>strong>Generated SQL query</strong></returns>
1. string Query<T>(this T obj, BuilderOptions options = null)
        /// <summary>
        /// <c>Generate select statement with options</c>
        /// </summary>
        /// <typeparam name="T">type</typeparam>
        /// <param name="obj">type object</param>
        /// <param name="selectColumns">columns to be included in the query</param>
        /// <param name="options">sort order</param>
        /// <returns><>strong>Generated SQL query</strong></returns>
2.string Query<T>(this T obj, List<string> selectColumns, BuilderOptions options = null) 
         /// <summary>
        /// <c>Generate select statement with options</c>
        /// </summary>
        /// <typeparam name="T">type</typeparam>
        /// <param name="obj">type object</param>
        /// <param name="whereClause">lambda expression</param>
        /// <param name="selectColumns">columns to be included in the query</param>
        /// <param name="options">sort order</param>
        /// <returns><>strong>Generated SQL query</strong></returns>
3. string Query<T>(this T obj, Expression<Func<T, bool>> whereClause, List<string> selectColumns = null, BuilderOptions options = null)
        /// <summary>
        /// <c>Generate select statement with options</c>
        /// </summary>
        /// <typeparam name="T">type</typeparam>
        /// <param name="obj">type object</param>
        /// <param name="rows">number of row to be fetched</param>
        /// <param name="offset">page offset</param>
        /// <param name="whereClause">lambda expression</param>
        /// <param name="selectColumns">columns to be included in the query</param>
        /// <param name="options">sort order</param>
        /// <returns><>strong>Generated SQL query</strong></returns>
4. string Query<T>(this T obj, int rows, int offset = 0, Expression<Func<T, bool>> whereClause = null, List<string> selectColumns = null, BuilderOptions options = null)
   /// <summary>
        /// <c>Build customer query with custome header and tail</c>
        /// <typeparam name="T">type</typeparam>
        /// <param name="obj">type object</param>
        /// <param name="queryHead">SELECT [queryHead] FROM ....</param>
        /// <param name="queryTail">SELECT [queryHead] FROM .... WHERE ... [queryTail]</param>
        /// <param name="whereClause">lambda expression</param>
        /// <returns><>strong>Generated SQL query</strong></returns>
5. string Query<T>(this T obj, string queryHead, string queryTail = null, Expression<Func<T, bool>> whereClause = null) 
```

Sample Codes
---

Example 1
***
```csharp
var sql = new user().Query()
```
>Output
```sql
SELECT  *  FROM user 
```
Example 2
***
```csharp
 var sql = new user().Query(
                whereClause: x => ((x.name == "oscar" && x.isActive == true) || x.age > 20),
                options: new SQL.QueryBuilder.Options.BuilderOptions
                {
                    orderBy = SQL.QueryBuilder.Enums.OrderBy.ASC,
                    orderByColumn = "age"
                });
```
>Output

```sql
SELECT  *  FROM user Where ((([name] = 'oscar') AND ([isActive] = 1)) OR ([age] > 20)) ORDER BY age ASC
```
Example 3
***
```csharp
 var sql = new user().Query(
               whereClause: x => ((x.name == "oscar" && x.isActive == true) || x.age > 20),
               options: new SQL.QueryBuilder.Options.BuilderOptions
               {
                   orderBy = SQL.QueryBuilder.Enums.OrderBy.ASC,
                   orderByColumn = "age"
               },
               selectColumns: new List<string>() { "age", "name" }
               );
```
>Output
```sql
SELECT  age,name  FROM user Where ((([name] = 'oscar') AND ([isActive] = 1)) OR ([age] > 20)) ORDER BY age ASC
```

Example 4
***

```csharp
 var items = new List<int>() { 3, 6, 3, 2 };
            var sql = new user().Query(rows: 20, offset: 0,
               selectColumns: new List<string>() { "age", "name" },
               whereClause: x => x.name.Contains("oscar") && items.Contains(x.id)
               );
```
>Output
```sql
SELECT  age,name  FROM  user Where  (nolock)  (([name] LIKE '%oscar%') AND ([id] IN (3,6,3,2)))  OFFSET  0  ROWS FETCH NEXT  20  ROWS ONLY 
```


Example 5
***
```csharp
var today = DateTime.UtcNow.Date;
            var future = DateTime.UtcNow.AddYears(12).Date;
            var list = new List<int>() { 1, 2, 3, 4, 6 };
            var sql = new user().Query(whereClause: x => (((x.dob >= today && x.dob <= future) && x.age >= 30) || !x.isActive) && list.Contains(x.id));
```
>Output
```sql
SELECT  *  FROM user WHERE ((((([dob] >= '9/18/2020 12:00:00 AM') AND ([dob] <= '9/18/2032 12:00:00 AM')) AND ([age] >= 30)) OR (NOT ([isActive] = 1))) AND ([id] IN (1,2,3,4,6))) 
```

Example 6
***

```csharp
   var sql = new user().Query(queryHead: "count(distinct(name))", whereClause: x => x.name.EndsWith("oscar"));
```
>Output
```sql
SELECT count(distinct(name)) FROM user WHERE ([name] LIKE '%oscar') 
```

Example 7
***

```csharp
 var sql = new user().Query(queryHead: "name,age",queryTail:"group by age");
```
>Output
```sql
SELECT name,age FROM user  group by age
```
Supported Lambda Expression Operations
---

|Input|Output|
|-----|------|
|x => x.id == 1|	([id] = 1)|
|x => x.isAcrive|([isAcrive] = 1)|
|x => !x.isAcrive|	(NOT ([isAcrive] = 1))|
|x => x.name == null|	([name] IS NULL)|
|x => x.id == id| (where id = 2)	([id] = 2)|
|x => x.id == 1 && x.name == "Main"|	(([id] = 1) AND ([Name] = 'Main'))|
|x => x.name.Contains(“Main”)|	([name] LIKE ‘%Main%’)|
|x => x.name.StartsWith("R")|	([name] LIKE 'R%')|
|x => list.Contains(x.id)|	([id] IN (1, 2, 3))|
|x => x.id == 1 !! x.name == "Main"|	(([id] = 1) OR ([Name] = 'Main'))|
|x=>x.name.EndWith("R")|([name] LIKE '%R')|


SQL Command Resolver Extension Methods
---
```csharp
        /// <summary>
        ///   Generate insert statement
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="obj"></param>
        /// <returns></returns>
1. string Insert<T>(this T obj)
        /// <summary>
        /// Generate update statement
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="obj"></param>
        /// <param name="whereClause"></param>
        /// <param name="selectColums"></param>
        /// <returns></returns>
2. string Update<T>(this T obj, Expression<Func<T, bool>> whereClause, List<string> selectColums = null) 
        /// <summary>
        /// Generate delete statement
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="obj"></param>
        /// <param name="whereClause"></param>
        /// <returns></returns>
3. string Delete<T>(this T obj, Expression<Func<T, bool>> whereClause)
```

Sample Codes
---

Example 1
***
```csharp
  var sql = new user().Insert();
```
>Output

```sql
INSERT INTO user (age,name,isActive) VALUES (@age,@name,@isActive)
```

Example 2
***
```csharp
  var items = new List<int>() { 3, 6, 3, 2 };
            var sql = new user().Update(
                whereClause: x => x.name.Contains("oscar") && items.Contains(x.id);
```
>Output

```sql
UPDATE user SET age=@age,name=@name,isActive=@isActive Where (([name] LIKE '%oscar%') AND ([id] IN (3,6,3,2)))
```

Example 3
***
```csharp
  var items = new List<int>() { 3, 6, 2 };
            var sql = new user().Update(
              whereClause: x => x.name.Contains("oscar") && items.Contains(x.id),
              selectColums: new List<string>() { "name" }
              );

```
>Output

```sql
UPDATE user SET name=@name Where (([name] LIKE '%oscar%') AND ([id] IN (3,6,2)))
```

Example 4
***
```csharp
 var items = new List<int>() { 3, 6, 3, 2 };
            var sql = new user().Delete(
            whereClause: x => x.name.Contains("oscar") && items.Contains(x.id));
```
>Output
```sql
DELETE FROM user Where (([name] LIKE '%oscar%') AND ([id] IN (3,6,3,2)))
```

Developer profile:  [LinkedIn](https://www.linkedin.com/in/oscar-itaba-4b5107a0/)
Email: [oscardybabaphd@gmail.com]