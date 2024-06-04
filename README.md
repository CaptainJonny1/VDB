## Voy.DALBase Readme
&ensp;&ensp;&ensp;&ensp;Voy.DALBase is a lightweight and universal ORM Library developed based on .NET Standard. It supports the construction and development of multiple databases, multiple .NET frameworks, multiple programming languages, and multiple operating system projects.

+ Database support: MySql, Sql Server, SQLite (more will be added).
+ Programming language support: C#, VB.Net, F#.
+ .NET version support:

|.NET implementation|Version support|
|-|-|
|.NET and .NET Core|2.0、2.1、2.2、3.0、3.1、5.0、6.0、7.0|
|.NET Framework|4.6.1 2、4.6.2、4.7、4.7.1、4.7.2、4.8、4.8.1|
|Mono|5.4、6.4|
|Xamarin.iOS|10.14、12.16|
|Xamarin.Mac|3.8、5.16|
|Xamarin.Android|8.0、10.0|
|Universal Windows Platform|10.0.16299, TBD
|Unity|2018.1|
### Install
||Command|
|-|-|
|.NET CLI|```dotnet add package Voy.DALBase```|
|Package Manager|```NuGet\Install-Package Voy.DALBase```|
|PackageReference|```<PackageReference Include="Voy.DALBase" />```|
|Paket CLI|```paket add Voy.DALBase```|
### Overview of functionality
|Namespace|Overview|
|-|-|
|Voy.DALBase|Use expressions to operate data, operate data tables (Code First), operate databases, and transaction operations.|
|Voy.DALBase.Tools|Automatically create (DB First) Model, data warehouse, data warehouse interface, WebAPIController, insert registration Ioc code dependency injection, singleton factory, create or merge expressions according to the data table structure.|
|Voy.DALBase.Utils|Enumeration for reference.|
|Voy.Toolkit.Common[^1]|Perform data inspection, conversion, encryption and decryption, creation, acquisition, manipulation, analysis and other operations.|
|Voy.Tools[^2]|Based on Voy.DALBase and Voy.Toolkit.Common, duplicate codes are automatically generated based on templates that can be modified by yourself.|
### How to use
#### SQL Statement expansion method list
|Method|`GetSQLString()`|`GetParams()`|`Execute()`|`GetData()`|`ExecuteScalar()`|
|-|:-:|:-:|:-:|:-:|:-:|
|`Insert()`|✓|✓|✓|||
|`Delete()`|✓|✓|✓|||
|`Update()`|✓|✓|✓|||
|`Select()`|✓|✓||✓|✓|
|`TableIsExist()`|✓|✓||✓||
|`CreateTableIfNotExist()`|✓|✓|✓|||
|`DropTable()`|✓|✓|✓|||
|`TruncateTable()`|✓|✓|✓|||
|`GetTableStructure()`|✓|✓||✓||
|`DatabaseIsExist()`|✓|✓||✓||
|`CreateDatabaseIfNotExist()`[^3]|✓|✓|✓|||
|`DropDatabase()`[^3]|✓|✓|✓|||
|`GetDatabaseTables()`|✓|✓||✓||
#### SQL Clause expansion method list
|Method|Parameter|Reusable times|Remark|
|-|-|:-:|-|
|`Where()`|Lambda expressions|∞|When used multiple times, the filter condition relationship between methods is And.|
|`WhereByBase()`|Lambda expressions|∞|When the type used for filtering is uncertain and common attributes are required (for example, when it is agreed that each class has an Id). WhereByBase<TClass, TBase> The two type parameters are required.|
|`FromQuery()`|SelectEntity|∞|From clause can be used to specify a sub-query expression in SQL. |
|`GroupBy()`|Lambda expressions|∞|groups rows that have the same values into summary rows, like "find the number of customers in each country".|
|`OrderBy()`|Lambda expressions|∞||
|`OrderByDesc()`|Lambda expressions|∞||
|`InnerJoin()`|Lambda expressions|∞|selects records that have matching values in both tables.|
|`LeftJoin()`|Lambda expressions|∞|returns all records from the left table (table1), and the matching records from the right table (table2). The result is 0 records from the right side, if there is no match.|
|`RightJoin()`|Lambda expressions|∞|returns all records from the right table (table2), and the matching records from the left table (table1). The result is 0 records from the left side, if there is no match.|
|`FullOutJoin()`|Lambda expressions|∞|returns all records when there is a match in left (table1) or right (table2) table records.|
|`Page()`|int pageIndex, int pageSize|1|The total number of records can be obtained using the "Count" property.<br>***Note: SQL Server requires version 2012 and higher, and OrderBy sorting is required.***|
+ Methods can also be used multiple times, and the order of use is not required. The relationship between the filter conditions executed by multiple methods is And.
+ If there is an "OR" relationship, you can use the static method "Tools.ExpressionTools.CreateExpression<T>()" to create more flexible filter conditions as parameters. For example:
```C#
var e1 = ExpressionTools.CreateExpression<User>(u => u.Id == 1 || u.Name.Contains("z"));
var e2 = ExpressionTools.CreateExpression<User>(u => u.Age > 20);

var exp = e1.And(e2);
var v1 = vdb.Select<User>().Where(exp);
```
#### Operators For Filter（Where Method）
|Operator|Usage example|Description|
|:-:|-|-|
|`==`|`u.Name == "Johnny"`||
|`!=`|`u.Name != "Johnny"`||
|`<`|`u.Age < 30`||
|`<=`|`u.Age <= 30`||
|`>`|`u.Age > 30`||
|`>=`|`u.Age >= 30`||
|`&&`|`u.Name == "Johnny" && u.Age == 30`||
|`\|\|`|`u.Name == "Johnny" \|\| u.Age == 30`||
|`StartsWith`|`u.Name.StartsWith("J")`||
|`EndsWith`|`u.Name.EndsWith("y")`||
|`Contains`|`u.Name.Contains("a")`<br>`new[] { 1, 2, 3, 4 }.Contains(u.Id)`<br>`Enumerable.Contains<int>(new[] { 1, 2, 3, }, u.Age)`<br>`Enumerable.Contains<string>(new[] { "Tom", "Jerry" }, u.Name)`||
|`Substring`|`u.Name.Substring(0, 1) == "Tom"`||
|`IndexOf`|`u.Name.IndexOf("Tom", 1) == 1`||

#### Instantiate
```C#
using Microsoft.Data.SqlClient; //Connect to SqlServer database.
using MySqlConnector;           //Connect to Mysql database.
using System.Data.OleDb;        //Connect to Access database.
using System.Data.SQLite;       //Connect to SQLite database.
using System.Data.Common;
using Voy.DALBase.Tools;

//When you need to connect to Mysql database.
DbConnection conn = new MySqlConnection("Server=XXX.XXX.XXX.XXX; Port=XXXX; Database=XXXX; Uid=XXXX; Pwd=XXXXXXXX; SslMode=Preferred;"); 

//When you need to connect to SqlServer database.
DbConnection conn = new SqlConnection("Server=XXX.XXX.XXX.XXX; Port=XXXX; Database=XXXX; Uid=XXXX; Pwd=XXXXXXXX; TrustServerCertificate=true;"); 

//When you need to connect to SQLite database.
string dbPath = @"D:\db1.sqlite";
SQLiteConnection.CreateFile(dbPath);
DbConnection connDB = new SQLiteConnection($"Data Source={dbPath}");

VDB vdb = new VDB(conn);   //normal mode.
VDBSingleton.Instance.VDB = new VDB(conn);  //singleton mode.
```
#### Insert
+ Use Lambda expressions and entity objects as parameters to insert a single record at a time, or use List<TClass> type parameters to insert multiple records in batches and return the number of inserted records.
+ Returns the auto-incrementing primary key value when inserting a record. If the primary key is not auto-increasing, 0 is returned. Returns the number of affected rows when inserting multiple records.
+ Nullable properties will have a null value inserted, non-nullable properties will have a default value inserted.
```C#
//Insert data with Lambda expressions.
int? ins1 = vdb.Insert<User>(u => new User { Name = "Tom", Age = 30 }).Execute();

//Insert data with entity objects. (<TClass> can be omitted)
int? ins2 = vdb.Insert(new User { Name = "Jerry", Age = 40 }).Execute();

//Insert multiple data.
int? ins3 = vdb.Insert<User>(new List<User>
{
    new User { Name = "Doctor", Age = 10, CreateTime = DateTime.Now },
    new User { Name = "Bashful", Age = 20, CreateTime = DateTime.Now },
    new User { Name = "Sleepy", Age = 50, CreateTime = DateTime.Now },
    new User { Name = "Sneezy", Age = 60, CreateTime = DateTime.Now },
    new User { Name = "Happy", Age = 70, CreateTime = DateTime.Now },
    new User { Name = "Dopey", Age = 80, CreateTime = DateTime.Now },
    new User { Name = "Grumpy", Age = 90, CreateTime = DateTime.Now },
}).Execute();
```
#### Update
+ Use Lambda expressions and anonymous objects as parameters to modify records and return the number of updated records. Entity objects are not currently supported and will not execute successfully using entity objects. For forward compatibility with subsequent versions, parameter types continue to use `dynamic`.
+ Supports common multi-type parameter updates, and also supports operations such as addition, subtraction, multiplication, and division for numerical fields based on their own numerical values. **Notice! This kind of operation requires actively adding a Where statement to control the update scope, otherwise the entire table will be updated.**
+ When updating records using anonymous object parameters and the `Where()` method, you can update any column including the primary key; when updating only using object parameters, the value of the primary key field will be used as a filter condition and will not be updated.
+ When the column of the anonymous object does not include the primary key field and the `Where()` method is not used for update filtering, the update operation will not be performed for data security.
+ Properties with the same name as those in `TClass` in anonymous object parameters will not appear in the generated SQL statement if they have the `[DatabaseGeneratedOption.Computed]` tag.
```C#
//Update data with Lambda expressions.
int? upd1 = vdb.Update<User>(u => new User { Id = 1, Name = "Sleepy", Age = 50 }).Execute();

//Update data with expression + Where method.
int? upd2 = vdb.Update<User>(u => new User { Name = "Sneezy", Age = 60 }).Where(u => u.Name == "Happy").Execute();

//Update data with an anonymous object.
int? upd3 = vdb.Update<User>(new { Id = 1, Name = "Happy", Age = 70 }).Execute();

//Update data with anonymous object update + Where method.
int? upd4 = vdb.Update<User>(new { Name = "Dopey", Age = 80 }).Where(u => u.Name == "Grumpy").Execute();

//Update data with Lambda expressions, and perform operations such as addition, subtraction, multiplication, and division based on your own value. Notice! This kind of operation requires actively adding a Where statement to control the update scope, otherwise the entire table will be updated.
int? upd5 = vdb.Update<User>(u => (u.Number + 1) & (u.Age - 2) & (u.Order * 3)).Where(u => u.Id == 1);
```
#### Delete
+ Use Lambda expressions and entity objects as parameters to delete records, or use the `Where()` method as a filter condition to delete records and return the number of deleted records.
+ When using entity objects to filter records as parameters, if the object's attribute is a numerical default value (such as 0 for int), it will have no effect. For example, do not use `new User { Age = 0 }`.
+ When there are parameters and `Where()` is also used for update filtering, the filtering conditions will be AND combined.
+ When there are no parameters and `Where()` is not used for update filtering, the update operation will not be executed for data security.
```C#
//Delete data with "Where" method.
int? del1 = vdb.Delete<User>().Where(u => u.Name == "Tom").Execute();

//Delete multiple data with "Where" method.
List<int> ids = new List<int>{ 1, 2 };
int? del2 = vdb.Delete<User>().Where(u => ids.ToArray().Contains(u.Id)).Execute();

//Delete multiple data with IList<TClass>. (<TClass> can be omitted, All properties need to be assigned a value to take effect.)
int? del3 = vdb.Delete(new List<User>() {
    new User(){ Id = 1, Name = "Tom", Age = 2},
    new User(){ Id = 2, Name= "Jerry", Age = 1 }
}).Execute();
```
#### Transaction
```C#
try
{
    vdb.BeginTransaction();
    int? result = vdb.Insert<User>(u => new User { Name = "NewUser1", Age = 30 }).Execute();
    int newId = result is null ? 0 : (int)result;
    var query = vdb.Insert<Order>(new List<Order>
    {
            new Order { UserId=newId, Product = "NewProduct1", Amount = (decimal)10.10 },
            new Order { UserId=newId, Product = "NewProduct2", Amount = (decimal)20.20 },
            new Order { UserId=newId, Product = "NewProduct3", Amount = (decimal)30.30 },
            new Order { UserId=newId, Product = "NewProduct4", Amount = (decimal)40.40 },
            new Order { UserId=newId, Product = "NewProduct5", Amount = (decimal)50.50 },
    }).Execute();
    vdb.Commit();
}
catch (Exception ex)
{
    vdb.Rollback();
    //Write to log;
}
```
#### Query
+ Use Lambda expressions as parameters to query records, supporting single-table and multi-table queries.
##### Single table query
```C#
//Get all fields
IEnumerable<User> sel1 = vdb.Select<User>()
         .Where<User>(u => u.Age > 20).OrderBy(u => u.Age).Page(1, 10).GetData();

//Get the specified field
IEnumerable<User> sel2 = vdb.Select<User>(u => new { u.Id, u.Name })
         .Where<User>(u => u.Age > 20).OrderBy(u => u.Age).Page(1, 10).GetData();
```
##### Single table Self-linking query
```C#
IEnumerable<User> sel1 = vdb.Select<User, User, User>()
        .LeftJoin((u, u1, u2) => u.CreatorId == u1.Id)
        .LeftJoin((u, u1, u2) => u.UpdaterId == u2.Id)
        .Where<User>((u, u1, u2) => u.Age > 20)
        .OrderBy((u, u1, u2) => u.Age)
        .Page(1, 10)
        .GetData();
```
```C#
public class User
{
    public int Id { get; set; }
    ...
    [ForeignKey("Creator")]
    public int CreatorId { get; set; }

    [ForeignKey("Updater")]
    public int UpdaterId { get; set; }

    public User Creator { get; set; }

    public User Updater { get; set; }
}
```
##### Multi-table query
+ Multi-table query supports one-to-many, many-to-one, and one-to-one queries, and up to 8 tables can be queried at the same time.
+ Multi-table query will return the set of the first formal parameter, so in order to obtain the correct return result, you should add the navigation property of the model class of the sub-table mapping to the model class of the main table mapping. For one-to-many, it is `List< Generic collection of child table models>`, one-to-one is a single model (if there are multiple attributes of the same type, you need to use the `[ForeignKey()]` tag to specify the navigation attributes).
```C#
IEnumerable<User> sel1 = vdb.Select<User, User, User, Order>()
    .LeftJoin((u, u1, u2, o) => u.CreatorId == u1.Id)
    .LeftJoin((u, u1, u2, o) => u.UpdaterId == u2.Id)
    .LeftJoin((u, u1, u2, o) => u.Id == o.UserId && o.IsDeleted == 0)
    .Where((u, u1, u2, o) => u.Age > 20)
    .OrderBy((u, u1, u2, o) => u.Age)
    .GetData();
```
```C#
public class User
{
    public int Id { get; set; }
    ...

    [ForeignKey("Creator")]
    public int CreatorId { get; set; }

    [ForeignKey("Updater")]
    public int UpdaterId { get; set; }

    public User Creator { get; set; }

    public User Updater { get; set; }

    public List<Order> Orders { get; set; }
}
```
##### Multi-table paging query
+ This should be achieved using FromQuery subquery. For example:
```C#
IEnumerable<User> result = vdb.Select<User, Order>()
     .FromQuery(vdb.Select<User>(u => new { u.Id, u.Name })
                 .Where(u => u.IsDeleted == 0)
                 .OrderBy(u => u.ShowOrder)
                 .Page(1, 10))
     .LeftJoin((u, o) => u.Id == o.UserId)
     .GetData();
```
##### Union primary key query
```C#
IEnumerable<User> result = vdb.Select<User, Order>()
     .LeftJoin((u, o) => u.Id == o.UserId && u.Name == o.UserName)
     .GetData();
```
##### Multi-column query
Setting the second parameter `filterColumns` of the Where method to a new expression can achieve a more convenient multi-column query, so that the filtering conditions can be applied to all columns specified in the expression. For example:
```C#
IEnumerable<User> result = vdb.Select<User, Order>()
     .Where(u => u.Name.Contains("Tianjin"), u => new { u.Name, u.Introduction })
     .GetData();
```
##### Recursive query[^3]
Suitable for querying infinite hierarchical table structures. For example, the table contains the ParentId field, which is used to record the superior of the current record.
+ Use the first expression parameter of "InnerJoin" to set the relationship between the identity field and its parent identity field, if the field representing the parent Id in the expression is recursive upward to the left of the equals sign and downward recursive to the right of the equals sign, and set the condition for the recursion to end in the second expression parameter.
+ The third expression is an optional parameter. If this parameter is set to a field mapping attribute, VDB will use this attribute as a condition to convert the query results into a tree structure.
+ The result of the spanning tree structure requires that the Model mapped by the main table contains a generic List collection attribute of the same type as the main table.
+ Applicable to SQLServer2005, MySql8.0 (Released in 2018), SQLite 3.8.3 (Released in 2014-02-03) and newer versions.
+ Maximum recursion depth: SQLServer has no limit on the recursion depth. It is 4,294,967,295 for MySQL and 1000 for SQLite.
```C#
     [Table("user")]
     public class User
     {
         ···
         public int Id { get; set; }
         public int ParentId { get; set; }
         ...
         public List<User> Users { get; set; }
         public List<Order> Orders { get; set; }
     }
```
```C#
IEnumerable<User> result = vdb.Select<User, Order>((u, o) => new { u.Id, u.ParentId, u.Name, u.ShowOrder, o.Product })
     .LeftJoin((u, o) => o.UserId == u.Id)
     .InnerJoin((u) => u.Id == u.ParentId, u => u.Id == 1, u => u.ParentId)
     .GetData();
```
##### Aggregation function
```C#
public class User
{
        public int Id { get; set; }
        public bool IsDeleted { get; set; }
        [NotMapped]
        public int SumForDeletedEmployeeAge { get; set; }
        [NotMapped]
        public int CountForDeletedEmployee { get; set; }
}
```
```C#
IEnumerable<User> result = vdb.Select<User>()
     .Sum(u => u.Age, "SumForDeletedEmployeeAge")
     .Count(u => u.Id, "CountForDeletedEmployee")
     .Where(u => u.IsDeleted == 0)
     .GetData();
```
|Method|Description|Parameters|Number of uses|
|-|-|-|:-:|
|`Avg()`|Calculate the average of a column. |Lambda expression、 Temporary column name mapped to a property of the same name in the Model (optional)|∞|
|`Count()`|Counts the number of items in the collection. |Lambda expression(recommended to select the identity column)、 Temporary column name mapped to a property of the same name in the Model (optional)|∞|
|`Max()`|Computes the maximum value of a column. |Lambda expression、 Temporary column name mapped to a property of the same name in the Model (optional)|∞|
|`Min()`|Computes the minimum value of a column. |Lambda expression、 Temporary column name mapped to a property of the same name in the Model (optional)|∞|
|`Sum()`|Computes the total value of a column. |Lambda expression、 Temporary column name mapped to a property of the same name in the Model (optional)|∞|
##### Conditional function
```C#
IEnumerable<User> result = vdb.Select<User>()
     .If(u => u.IsDeleted == 0 ? "Not deleted" : "Deleted", "Deleted status")
     .IfNull(u => u.Name ?? "Empty name")
     .GetData();
```
|Method|Description|Parameters|Number of uses|
|-|-|-|:-:|
|`If()`|If the judgment condition is true, return the first value after ?, otherwise return the second value. |Lambda expression, temporary column name (optional) |∞|
|`IfNull()`|If the judgment object is empty, the value after ?? will be returned. |Lambda expression|∞|
#### Data model
+ Without specifying the data table/data column name, VDB will apply different naming rules to map class name -> data table name, attribute name -> data column name according to the mapped database brand.
+ Use the `[Table]` and `[Column]` tags on the class and attribute respectively to specify the mapped data table and data column.
##### Default naming rules
|Database Brand|Table Naming Rules|Table Name Examples|Column Naming Rules|Column Name Examples|
|-|-|-|-|-|
|MySQL|Lowercase letters and underscores|`UserSetting`->`` `user_setting` ``|Lowercase letters and underscores|`UserId`->`` `user_id` ``|
|SQL Server|Pascal|`UserSetting`->`[UserSetting]`|Camel|`UserId`->`[userId]`|
|SQLite|Pascal|`UserSetting`->`'UserSetting'`|Lowercase letters and underscores|`UserId`->`` `user_id` ``|
|Access|Pascal|`UserSetting`->`[UserSetting]`|Camel|`UserId`->`[userId]`|
|Oracle|Capital letters and underscores|`UserSetting`->`"USER_SETTING"`|Capital letters and underscores|`UserId`->`"USER_ID"`|
##### Instructions for using data table attribute labels
|Name|Description|
|-|-|
|`[Table]`|Represents the database table to which the class will be mapped. <br>Usage Example：`[Table("user_team")]`<br>Namespace：System.ComponentModel.DataAnnotations.Schema|
##### Instructions for using data column attribute labels
|Name|Description|
|-|-|
|`[Column]`|Represents the database column to which the attribute will be mapped. <br>Usage Example：`[Column("id", Order = 0, TypeName = "int")`]<br>Namespace：System.ComponentModel.DataAnnotations.Schema|
|`[Computed]`|When a row is inserted or updated, the database generates a value. <br>Usage Example：`[DatabaseGenerated(DatabaseGeneratedOption.Computed)]`<br>Namespace：System.ComponentModel.DataAnnotations.Schema|
|`[DefaultValue]`|Specifies the default value of the attribute. <br>Usage Example：`[DefaultValue("CURRENT_TIMESTAMP")]`<br>Namespace：System.ComponentModel|
|`[Description]`|Specifies the description of the property or event. <br>Usage Example：`[Description("name")]`<br>Namespace：System.ComponentModel|
|`[ForeignKey]`|Specifies which property is the foreign key in the relationship. On the foreign key property of the dependent class, pass in the name of the navigation property. <br>Usage Example：`[ForeignKey("User")]`<br>Namespace：System.ComponentModel.DataAnnotations.Schema|
|`[Identity]`|When a row is inserted, the database generates a value. <br>Usage Example：`[DatabaseGenerated(DatabaseGeneratedOption.Identity)]`<br>Namespace：System.ComponentModel.DataAnnotations.Schema|
|`[Key]`|Represents one or more attributes that uniquely identify an entity. <br>Usage Example：`[Key]`<br>Namespace：System.ComponentModel.DataAnnotations|
|`[None]`|The database does not generate a value. <br>Usage Example：`[DatabaseGenerated(DatabaseGeneratedOption.None)]`<br>Namespace：System.ComponentModel.DataAnnotations.Schema|
|`[NotMapped]`| Indicates that a property or class should be excluded from database mapping. <br>Usage Example：`[NotMapped]`<br>Namespace：System.ComponentModel.DataAnnotations.Schema|
|`[Required]`|The specified data field value is required. <br>Usage Example：`[Required]`<br>Namespace：System.ComponentModel.DataAnnotations|
##### Usage examples
```C#
     /// <summary>
     /// User
     /// </summary>
     [Table("user")]
     public class User
     {
         [Column(Order = 0)]
         [Key]
         [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
         public int Id { get; set; }

         [Required]
         [Description("Name.")]
         [Column("name", TypeName = "varchar(50)")]
         public string? Name { get; set; }

         [Description("Age.")]
         public int? Age { get; set; }

         [Required]
         [DefaultValue(0)]
         [Description("Display order.")]
         public int? ShowOrder { get; set; }

         /// <summary>
         /// user type.
         /// </summary>
         public UserType? UserType { get; set; }

         /// <summary>
         /// Order List.
         /// </summary>
         public List<Order>? Orders { get; set; } = new List<Order>();

         [Required]
         [DefaultValue(0)]
         [Description("Whether logical deletion.")]
         public sbyte? IsDeleted { get; set; }

         [Column(TypeName = "timestamp")]
         [Required]
         [DefaultValue("CURRENT_TIMESTAMP")]
         [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
         [Description("Creation time.")]
         public DateTime? CreateTime { get; set; }
     }
```
1. The int attributes of the `[Key]` and `[Identity]` labels are set, and the mapped data columns are auto-growing columns.
1. The fields identified by `[Key]` and `[Required]` are required. There is no need to repeatedly set these two properties for the same attribute.
1. The `[Identity]` tag (automatic growth) only takes effect for int type columns. After setting DefaultValue for non-int type columns, the `[Identity]` tag will be ignored.
1. For attribute mapping columns with the `[Computed]` tag set, the default value will be calculated during both insertion and update. The default value without `[Computed]` will only be calculated during insertion.
1. If the attribute of the `[Column]` tag is not set, the attribute name will be used to generate the column name in the corresponding form according to the database brand, and the corresponding column type and default maximum length will also be generated according to the data type of the attribute.
1. If the value of a required item is not set when inserting data, and the item does not have a default value, the Insert statement will automatically increase the default value of the item type based on the content of `TypeName`. If `TypeName` is not set, the closest field type will be assigned based on the attribute type.
1. When using entity classes to insert records, it is recommended to set all mapped attributes to nullable types. The reason is that non-null attributes in entity classes will automatically generate default values, and fields with values will be added to the SQL statement.
1. When inserting records using an object instance, if the value of the attribute mapped to a timestamp type field is not set, execution will fail because the minimum time is automatically generated. Solution 1. Set the property to a nullable type. 2. Change the timestamp to time type. 3. Set the attribute value mapped by this field to be within the range allowed by the timestamp.
1. It is recommended to set the int type attribute (mostly found in the Id field) that is set to auto-increment (using the `[Identity]` tag) to the nullable type "int?", otherwise the sql insertion content will include a value of 0 This field, however, can be executed successfully.
1. When inserting in batches, all attributes mapped to required fields with default values must be set or none of the values must be set. Because there is an object in the parameter collection that has this field set, this field will appear in the SQL statement, and other records that do not have this field set will not be executed successfully due to lack of parameters.
1. `[ForeignKey]` example:
```C#
public class User
{
     public int Id { get; set; }
     ...
     public List<Order> Orders { get; set; }
}

public class Order
{
     public int OrderId { get; set; }
     [ForeignKey("Seller")]
     public int SellerId { get; set; }
     [ForeignKey("Buyer")]
     public int BuyerId { get; set; }
     ...
     public User Seller { get; set; }
     public User Buyer { get; set; }
}
```
#### Manipulate databases and data tables
+ Use Lambda expressions to operate databases and data tables.
##### Create a data table based on Model (Code First)
```C#
int? result = vdb.CreateTableIfNotExist<User>().Execute();
```
##### Remove data table
```C#
int? result = vdb.DropTable<User>().Execute();
```
##### Clear data table
```C#
int? result = vdb.TruncateTable<User>().Execute();
```
##### Check if the data table exists
```C#
bool result = vdb.TableIsExist<User>().GetData();
```
##### Get the data table schema
```C#
IEnumerable<dynamic> result = vdb.GetTableStructure("user").GetData();
```

#### Create data warehouse supporting files based on data tables
+ Create .NET source files at the specified location through the code tool (`CodeTool`) object.
+ You can specify parameters such as the language used in the source file (C#, VB.net, F#), whether to overwrite the file with the same name (not overwritten by default), and the saving location.
##### Instantiation tool
```C#
var codeTool = new CodeTool(conn);
codeTool.Language = ProgrammingLanguage.CSharp; //If not specified, C# will be used.
codeTool.NameSpace = "MyDemo"; // If not specified, the current project name will be used.
```
|Parameters of CodeTool|Description|Type|Default value|
|-|-|-|-|
|`Connection`|Data Connection|`DbConnection`|None|
|`Tables`|Name of data table|`string[]`|All tables|
|`Types`|Generate the corresponding class name based on the data table name|`string[]`|None|
|`IgnoredColumns`|Columns to be ignored|`string[]`|None|
|`NameSpace`|Namespace|`string`|Current project name|
|`BaseTypes`|Base class or interface|`string[]`|None|
|`Language`|The programming language used to generate content|`ProgrammingLanguage`|CSharp|
##### Create Model based on data table (DB First)
```C#
var result1 = codeTool.CreateModel().ToFile();
```
##### Create warehouse interface based on the data table
```C#
var result2 = codeTool.CreateIRepository().ToFile();
```
##### Create warehouse based on the data table
```C#
var result3 = codeTool.Creatorepository().ToFile();
```
##### Insert registration Ioc code dependency injection
```C#
var result4 = codeTool.RegisterIoC().InsertFile();
```
##### Create WebAPIController based on the data table
```C#
var result5 = codeTool.CreateWebAPIController().ToFile();
```
|Method|Method description|Parameters|Parameter description|Type|Default value|
|-|-|-|-|-|-|
|`ToCodeString()`|Output the generated results to a string|None||||
|`ToFile()`|Save the generated results to a file|`overwrite`|Whether to overwrite the file with the same name|`bool`|`false`|
|||`targetPath`|The path to save the generated file|`string`|The corresponding folder in the current working directory of the application|
|`InsertFile()`|Insert the generated results into an existing file|`targetPath`|The saving path of the file where the code needs to be inserted|`string`|After the `var app = builder.Build();` statement in the `Program.cs` file in the current working directory of the application|
## Contact information
***email:cnxl@hotmail.com, we will reply as soon as possible.***

---
## Voy.DALBase使用说明
&ensp;&ensp;&ensp;&ensp;Voy.DALBase 是一个基于.NET Standard开发的轻巧通用ORM Library，支持多种数据库、多种.NET框架、多种编程语言、多种操作系统项目的构建和研发。

+ 数据库支持：MySql、Sql Server、SQLite（将添加更多）。
+ 编程语言支持：C#、VB.Net、F#。
+ .NET版本支持：

|.NET 实现|版本支持|
|-|-|
|.NET 和.NET Core|2.0、2.1、2.2、3.0、3.1、5.0、6.0、7.0|
|.NET Framework|4.6.1 2、4.6.2、4.7、4.7.1、4.7.2、4.8、4.8.1|
|Mono|5.4、6.4|
|Xamarin.iOS|10.14、12.16|
|Xamarin.Mac|3.8、5.16|
|Xamarin.Android|8.0、10.0|
|通用 Windows 平台|10.0.16299，待定
|Unity|2018.1|
### 安装
||命令|
|-|-|
|.NET CLI|```dotnet add package Voy.DALBase```|
|Package Manager|```NuGet\Install-Package Voy.DALBase```|
|PackageReference|```<PackageReference Include="Voy.DALBase" />```|
|Paket CLI|```paket add Voy.DALBase```|
### 功能概述
|命名空间|说明|
|-|-|
|Voy.DALBase|使用表达式操作数据，操作数据表(Code First)，操作数据库，事务操作。|
|Voy.DALBase.Tools|根据数据表结构自动创建（DB First）Model、数据仓库、数据仓库接口、WebAPIController、插入注册Ioc代码依赖注入，单例工厂，创建或合并表达式。|
|Voy.DALBase.Utils|引用枚举。|
|Voy.Toolkit.Common[^1]|对数据进行检查、转换、加密解密、创建、获取、操作、解析等操作。|
|Voy.Tools[^2]|以Voy.DALBase、Voy.Toolkit.Common为基础，根据可自行修改的模板自动生成重复代码。|

[^1]: 需要安装 [Voy.Toolkit.Common](https://www.nuget.org/packages/Voy.Toolkit.Common/)。
[^2]: 需要安装 [Voy.Toolks](https://www.nuget.org/packages/Voy.Tools/)。
### 使用
#### SQL 语句扩展方法列表
|方法说明|`GetSQLString()`<br>获取SQL语句|`GetParams()`<br>获取SQL语句的参数<br>|执行数据操作命令<br>`Execute()`|执行数据查询命令<br>`GetData()`|执行标量查询命令<br>`ExecuteScalar()`|
|-|:-:|:-:|:-:|:-:|:-:|
|`Insert()`<br>插入数据|✓|✓|✓|||
|`Delete()`<br>删除数据|✓|✓|✓|||
|`Update()`<br>更新数据|✓|✓|✓|||
|`Select()`<br>查询数据|✓|✓||✓|✓|
|`TableIsExist()`<br>检查数据表是否存在|✓|✓||✓||
|`CreateTableIfNotExist()`<br>创建数据表|✓|✓|✓|||
|`DropTable()`<br>移除数据表|✓|✓|✓|||
|`TruncateTable()`<br>清空数据表|✓|✓|✓|||
|`GetTableStructure()`<br>获取数据表架构信息|✓|✓||✓||
|`DatabaseIsExist()`<br>检查数据库是否存在|✓|✓||✓||
|`CreateDatabaseIfNotExist()`[^3]<br>创建数据库|✓|✓|✓|||
|`DropDatabase()`[^3]<br>移除数据库|✓|✓|✓|||
|`GetDatabaseTables()`<br>获取数据库架构信息|✓|✓||✓||

[^3]: Access不适用
#### SQL 子句扩展方法列表
|方法|说明|参数|可使用次数|备注|
|-|-|-|:-:|-|
|`Where()`|通过属性进行筛选|Lambda表达式|∞|多次使用时，方法之间的筛选条件关系是And。|
|`WhereByBase()`|通过基类的属性进行筛选|Lambda表达式|∞|用于筛选的类型不确定，又需要使用通用属性（例如约定每个类都有Id）的时候。WhereByBase<TClass, TBase>两个类型参数为必填。|
|`FromQuery()`|子查询|SelectEntity|∞|用于指定SQL中的子查询表达式。|
|`GroupBy()`|分组|Lambda表达式|∞||
|`OrderBy()`|升序排序|Lambda表达式|∞||
|`OrderByDesc()`|降序排序|Lambda表达式|∞||
|`InnerJoin()`|联合查询|Lambda表达式|∞|如果表中有至少一个匹配，则返回行。|
|`LeftJoin()`|左联合查询|Lambda表达式|∞|即使右表中没有匹配，也从左表返回所有的行。|
|`RightJoin()`|右联合查询|Lambda表达式|∞|即使左表中没有匹配，也从右表返回所有的行。|
|`FullOutJoin()`||Lambda表达式|∞|只要其中一个表中存在匹配，则返回行。|
|`Page()`|筛选结果的分页|int pageIndex, int pageSize|1|引用"Count"属性获得记录总数。<br>***注意：SQLServer要求版本2012及更高，必须有OrderBy排序。***|
+ 以上方法可以多次使用，使用顺序无要求。多个方法执行的筛选条件之关系为And。
+ 如果有“OR”的关系可以使用静态方法`Voy.DALBase.Tools.ExpressionTools.CreateExpression<T>()`来创建更灵活的筛选条件作为参数。例如：
```C#
var e1 = ExpressionTools.CreateExpression<User>(u => u.Id == 1 || u.Name.Contains("z"));
var e2 = ExpressionTools.CreateExpression<User>(u => u.Age > 20);

var exp = e1.And(e2);
var v1 = vdb.Select<User>().Where(exp);
```
#### 过滤器的运算符（Where 方法）
|运算符|使用示例|说明|
|:-:|-|-|
|`==`|`u.Name == "张三"`||
|`!=`|`u.Name != "张三"`||
|`<`|`u.Age < 30`||
|`<=`|`u.Age <= 30`||
|`>`|`u.Age > 30`||
|`>=`|`u.Age >= 30`||
|`&&`|`u.Name == "张三" && u.Age == 30`||
|`\|\|`|`u.Name == "张三" \|\| u.Age == 30`||
|`StartsWith`|`u.Name.StartsWith("张")`||
|`EndsWith`|`u.Name.EndsWith("三")`||
|`Contains`|`u.Name.Contains("张")`<br>`new[] { 1, 2, 3, 4 }.Contains(u.Id)`<br>`Enumerable.Contains<int>(new[] { 1, 2, 3, }, u.Age)`<br>`Enumerable.Contains<string>(new[] { "张三", "李四" }, u.Name)`||
|`Substring`|`u.Name.Substring(0, 1) == "张"`||
|`IndexOf`|`u.Name.IndexOf("张", 1) == 1`||

#### 实例化
```C#
using Microsoft.Data.SqlClient; //连接到SqlServer数据库。
using MySqlConnector;           //连接到Mysql数据库。
using System.Data.OleDb;        //连接到Access数据库。
using System.Data.SQLite;       //连接到SQLite数据库。
using System.Data.Common;
using Voy.DALBase.Tools;

//当你需要连接到Mysql数据库。
DbConnection conn = new MySqlConnection("Server=XXX.XXX.XXX.XXX; Port=XXXX; Database=XXXX; Uid=XXXX; Pwd=XXXXXXXX; SslMode=Preferred;"); 

//当你需要连接到SqlServer数据库。
DbConnection conn = new SqlConnection("Server=XXX.XXX.XXX.XXX; Port=XXXX; Database=XXXX; Uid=XXXX; Pwd=XXXXXXXX; TrustServerCertificate=true;"); 

//当你需要连接到SQLite数据库。
string dbPath = @"D:\db1.sqlite";
SQLiteConnection.CreateFile(dbPath);
DbConnection connDB = new SQLiteConnection($"Data Source={dbPath}");

VDB vdb = new VDB(conn);   //普通模式。
VDBSingleton.Instance.VDB = new VDB(conn);  //单例模式。
```
#### 插入数据
+ 使用Lambda表达式、实体对象做为参数单次插入单一记录，也可以用List<TClass>型参数批量插入多条记录，返回插入的记录数量。
+ 插入一条记录时返回自增主键值，如果主键不是自增长返回0。插入多条记录时返回影响行数。
+ 可空的属性将插入空值，不可空属性将插入默认值。
```C#
//使用Lambda表达式插入数据。
int? ins1 = vdb.Insert<User>(u => new User { Name = "张三", Age = 30 }).Execute();

//使用实体对象插入数据。（可省略<TClass>）
int? ins2 = vdb.Insert(new User { Name = "李四", Age = 40 }).Execute();

//批量插入数据。
int? ins3 = vdb.Insert<User>(new List<User>
{
    new User { Name = "刘一", Age = 10, CreateTime = DateTime.Now },
    new User { Name = "陈二", Age = 20, CreateTime = DateTime.Now },
    new User { Name = "王五", Age = 50, CreateTime = DateTime.Now },
    new User { Name = "赵六", Age = 60, CreateTime = DateTime.Now },
    new User { Name = "孙七", Age = 70, CreateTime = DateTime.Now },
    new User { Name = "周八", Age = 80, CreateTime = DateTime.Now },
    new User { Name = "吴九", Age = 90, CreateTime = DateTime.Now },
    new User { Name = "郑十", Age = 100, CreateTime = DateTime.Now },
}).Execute();
```
#### 更新数据
+ 使用Lambda表达式、匿名对象作为参数修改记录，返回更新的记录数量。目前不支持实体对象，使用实体对象将不能成功执行。为了后续版本向前兼容，参数类型继续使用`dynamic`。
+ 支持常见的多类型参数更新，也支持用于数值字段以自己数值为基数进行的加减乘除等运算。**注意！此种操作需要主动添加Where语句控制更新范围，否则将全表更新。**
+ 使用匿名对象参数和`Where()`方法更新记录时可以更新包括主键在内的任何列；只使用对象参数更新时，主键字段的值将作为筛选条件不进行更新。
+ 当匿名对象的列中未包括主键字段，也没有使用`Where()`方法进行更新筛选时，为了数据安全更新操作将不会被执行。
+ 匿名对象参数中与`TClass`中同名的属性，如果带有`[DatabaseGeneratedOption.Computed]`标签将不会出现在生成的SQL语句中。
```C#
//用Lambda表达式更新数据。
int? upd1 = vdb.Update<User>(u => new User { Id = 1, Name = "王五", Age = 50 }).Execute();

//用表达式 + Where方法更新数据。
int? upd2 = vdb.Update<User>(u => new User { Name = "赵六", Age = 60 }).Where(u => u.Name == "李四").Execute();

//用匿名对象更新数据。
int? upd3 = vdb.Update<User>(new { Id = 1, Name = "孙七", Age = 70 }).Execute();

//用匿名对象更新 + Where方法更新数据。
int? upd4 = vdb.Update<User>(new { Name = "周八", Age = 80 }).Where(u => u.Name == "赵六").Execute();

//用Lambda表达式更新数据，以自己数值为基数进行的加减乘除等运算。注意！此种操作需要主动添加Where语句控制更新范围，否则将全表更新。
int? upd5 = vdb.Update<User>(u => (u.Number + 1) & (u.Age - 2) & (u.Order * 3)).Where(u => u.Id == 1);
```
#### 删除数据
+ 使用Lambda表达式、实体对象做为参数删除记录，也可以用`Where()`方法作为筛选条件删除记录，返回删除的记录数量。
+ 使用实体对象为参数筛选记录时，如果对象的属性是数值型的默认值（例如int的0）则无效果，例如不要使用`new User { Age = 0 }`。
+ 当有参数，也使用`Where()`进行更新筛选时，筛选条件将进行AND合并。
+ 当无参数，也没有使用`Where()`进行更新筛选时，为了数据安全更新操作将不会被执行。
```C#
//用Where方法删除数据。
int? del1 = vdb.Delete<User>().Where(u => u.Name == "张三").Execute();

//用Where方法批量删除数据。
List<int> ids = new List<int>{ 1, 2 };
int? del2 = vdb.Delete<User>().Where(u => ids.ToArray().Contains(u.Id)).Execute();

//使用 IList<TClass> 批量删除数据。 （<TClass>可以省略，所有属性都需要赋值才能生效。）
int? del3 = vdb.Delete(new List<User>() {
    new User(){ Id = 1, Name = "张三", Age = 2},
    new User(){ Id = 2, Name= "李四", Age = 1 }
}).Execute();
```
#### 使用事务
```C#
try
{
    vdb.BeginTransaction();
    int? result = vdb.Insert<User>(u => new User { Name = "NewUser1", Age = 30 }).Execute();
    int newId = result is null ? 0 : (int)result;
    var query = vdb.Insert<Order>(new List<Order>
    {
            new Order { UserId=newId, Product = "NewProduct1", Amount = (decimal)10.10 },
            new Order { UserId=newId, Product = "NewProduct2", Amount = (decimal)20.20 },
            new Order { UserId=newId, Product = "NewProduct3", Amount = (decimal)30.30 },
            new Order { UserId=newId, Product = "NewProduct4", Amount = (decimal)40.40 },
            new Order { UserId=newId, Product = "NewProduct5", Amount = (decimal)50.50 },
    }).Execute();
    vdb.Commit();
}
catch (Exception ex)
{
    vdb.Rollback();
    //Write to log;
}
```
#### 查询数据
+ 使用Lambda表达式做为参数查询记录，支持单表和多表查询。
##### 单表查询
```C#
//获取全部字段
IEnumerable<User> sel1 = vdb.Select<User>()
        .Where<User>(u => u.Age > 20).OrderBy(u => u.Age).Page(1, 10).GetData();

//获取指定字段
IEnumerable<User> sel2 = vdb.Select<User>(u => new { u.Id, u.Name })
        .Where<User>(u => u.Age > 20).OrderBy(u => u.Age).Page(1, 10).GetData();
```
##### 单表自联查询
```C#
IEnumerable<User> sel1 = vdb.Select<User, User, User>()
        .LeftJoin((u, u1, u2) => u.CreatorId == u1.Id)
        .LeftJoin((u, u1, u2) => u.UpdaterId == u2.Id)
        .Where<User>((u, u1, u2) => u.Age > 20)
        .OrderBy((u, u1, u2) => u.Age)
        .Page(1, 10)
        .GetData();
```
```C#
public class User
{
    public int Id { get; set; }
    ...
    [ForeignKey("Creator")]
    public int CreatorId { get; set; }

    [ForeignKey("Updater")]
    public int UpdaterId { get; set; }

    public User Creator { get; set; }

    public User Updater { get; set; }
}
```
##### 多表查询
+ 多表查询支持一对多、多对一、一对一查询，最多可同时查询8个表。
+ 多表查询将返回第一个形参的集合，所以为了获得正确的返回结果，应在主表映射的模型类中增加类型为子表映射的模型类的导航属性，一对多时是`List<子表模型>`的泛型集合，一对一时是单一模型（如果有多个同类属性，需要使用`[ForeignKey()]`标签指定导航属性）。
```C#
IEnumerable<User> sel1 = vdb.Select<User, User, User, Order>()
    .LeftJoin((u, u1, u2, o) => u.CreatorId == u1.Id)
    .LeftJoin((u, u1, u2, o) => u.UpdaterId == u2.Id)
    .LeftJoin((u, u1, u2, o) => u.Id == o.UserId && o.IsDeleted == 0)
    .Where((u, u1, u2, o) => u.Age > 20)
    .OrderBy((u, u1, u2, o) => u.Age)
    .GetData();
```
```C#
public class User
{
    public int Id { get; set; }
    ...
    
    [ForeignKey("Creator")]
    public int CreatorId { get; set; }

    [ForeignKey("Updater")]
    public int UpdaterId { get; set; }

    public User Creator { get; set; }

    public User Updater { get; set; }

    public List<Order> Orders { get; set; }
}
```
##### 多表的分页查询
+ 应使用FromQuery子查询来实现。例如：
```C#
IEnumerable<User> result = vdb.Select<User, Order>()
    .FromQuery(vdb.Select<User>(u => new { u.Id, u.Name })
                .Where(u => u.IsDeleted == 0)
                .OrderBy(u => u.ShowOrder)
                .Page(1, 10))
    .LeftJoin((u, o) => u.Id == o.UserId)
    .GetData();
```
##### 联合主键查询
```C#
IEnumerable<User> result = vdb.Select<User, Order>()
    .LeftJoin((u, o) => u.Id == o.UserId && u.Name == o.UserName)
    .GetData();
```
##### 多列查询
将Where方法的第二个参数`filterColumns`设置为new表达式可以实现更方便的多列查询，这样可以将过滤条件应用到表达式中指定的全部列。例如：
```C#
IEnumerable<User> result = vdb.Select<User, Order>()
    .Where(u => u.Name.Contains("天津"), u => new { u.Name, u.Introduction })
    .GetData();
```
##### 递归查询[^3]
适用于查询无限分级的表结构。例如表中含有ParentId字段，用来记录当前记录的上级。
+ 使用“InnerJoin”的第一个表达式参数中设置标识字段与其父标识字段的关系，如果表达式中代表父Id的字段在等于号左侧为向上递归，在等于号右侧为向下递归，并在第二个表达式参数中设置递归结束的条件。
+ 第三个表达式为可选参数，如果设置该参数为一个字段映射的属性，VDB将使用该属性作为条件，将查询结果转化为树结构。
+ 生成树结构的结果要求主表映射的Model中包含有主表同类型的泛型List集合属性。
+ 适用于SQLServer2005、MySql8.0（2018年发布）、SQLite 3.8.3（2014-02-03发布）及更新版本。
+ 最大递归深度：SQLServer没有递归深度的限制，MySQL为4,294,967,295，SQLite为1000。
```C#
    [Table("user")]
    public class User
    {
        ···
        public int Id { get; set; }
        public int ParentId { get; set; }
        ...
        public List<User> Users { get; set; }        
        public List<Order> Orders { get; set; }
    }
```
```C#
IEnumerable<User> result = vdb.Select<User, Order>((u, o) => new { u.Id, u.ParentId, u.Name, u.ShowOrder, o.Product })
    .LeftJoin((u, o) => o.UserId == u.Id)
    .InnerJoin((u) => u.Id == u.ParentId, u => u.Id == 1, u => u.ParentId)
    .GetData();
```
##### 聚合函数
```C#
public class User
{
        public int Id { get; set; }
        public bool IsDeleted { get; set; }
        [NotMapped]
        public int SumForDeletedEmployeeAge { get; set; }
        [NotMapped]
        public int CountForDeletedEmployee { get; set; }
}
```
```C#
IEnumerable<User> result = vdb.Select<User>()
     .Sum(u => u.Age, "SumForDeletedEmployeeAge")
     .Count(u => u.Id, "CountForDeletedEmployee")
     .Where(u => u.IsDeleted == 0)
     .GetData();
```
|方法|说明|参数|可使用次数|
|-|-|-|:-:|
|`Avg()`|计算某个列的平均值。|Lambda表达式、映射到Model中同名属性的临时列名（可选）|∞|
|`Count()`|统计集合中的项目数。|Lambda表达式（建议选择标识列）、映射到Model中同名属性的临时列名（可选）|∞|
|`Max()`|计算列的最大值。|Lambda表达式、映射到Model中同名属性的临时列名（可选）|∞|
|`Min()`|计算列的最小值。|Lambda表达式、映射到Model中同名属性的临时列名（可选）|∞|
|`Sum()`|计算列的合计值。|Lambda表达式、映射到Model中同名属性的临时列名（可选）|∞|
##### 条件函数
```C#
IEnumerable<User> result = vdb.Select<User>()
    .If(u => u.IsDeleted == 0 ? "未删除" : "已删除", "删除状态")
    .IfNull(u => u.Name ?? "空名字")
    .GetData();
```
|方法|说明|参数|可使用次数|
|-|-|-|:-:|
|`If()`|判断条件如果为真，则返回?后面的第一个值，否则返回第二个值。|Lambda表达式，临时列名（可选）|∞|
|`IfNull()`|判断对象如果为空，则返回??后面的值。|Lambda表达式|∞|

#### 数据模型
+ 在不指定数据表/数据列名称的情况下，VDB会根据映射的数据库品牌，应用不同的命名规则来映射类名->数据表名、属性名->数据列名。
+ 在类和属性上分别使用`[Table]`和`[Column]`标签可以指定映射的数据表和数据列。
##### 默认命名规则
|数据库品牌|表命名规则|表名举例|列命名规则|列名举例|
|-|-|-|-|-|
|MySQL|小写字母与下划线|`UserSetting`->`` `user_setting` ``|小写字母与下划线|`UserId`->`` `user_id` ``|
|SQL Server|Pascal|`UserSetting`->`[UserSetting]`|Camel|`UserId`->`[userId]`|
|SQLite|Pascal|`UserSetting`->`'UserSetting'`|小写字母与下划线|`UserId`->`` `user_id` ``|
|Access|Pascal|`UserSetting`->`[UserSetting]`|Camel|`UserId`->`[userId]`|
|Oracle|大写字母与下划线|`UserSetting`->`"USER_SETTING"`|大写字母与下划线|`UserId`->`"USER_ID"`|
##### 数据表特性标签使用说明
|名称|说明|
|-|-|
|`[Table]`|表示类将映射到的数据库表。<br>使用示例：`[Table("user_team")]`<br>命名空间：System.ComponentModel.DataAnnotations.Schema|
##### 数据列特性标签使用说明
|名称|说明|
|-|-|
|`[Column]`|表示属性将映射到的数据库列。<br>使用示例：`[Column("id", Order = 0, TypeName = "int")`]<br>命名空间：System.ComponentModel.DataAnnotations.Schema|
|`[Computed]`|在插入或更新一个行时，数据库会生成一个值。<br>使用示例：`[DatabaseGenerated(DatabaseGeneratedOption.Computed)]`<br>命名空间：System.ComponentModel.DataAnnotations.Schema|
|`[DefaultValue]`|指定属性的默认值。<br>使用示例：`[DefaultValue("CURRENT_TIMESTAMP")]`<br>命名空间：System.ComponentModel|
|`[Description]`|指定属性或事件的说明。<br>使用示例：`[Description("name")]`<br>命名空间：System.ComponentModel|
|`[ForeignKey]`|指定哪个属性是关系中的外键。在依赖类的外键属性上，传入导航属性的名称。<br>使用示例：`[ForeignKey("User")]`<br>命名空间：System.ComponentModel.DataAnnotations.Schema|
|`[Identity]`|在插入一个行时，数据库会生成一个值。<br>使用示例：`[DatabaseGenerated(DatabaseGeneratedOption.Identity)]`<br>命名空间：System.ComponentModel.DataAnnotations.Schema|
|`[Key]`|表示唯一标识实体的一个或多个属性。<br>使用示例：`[Key]`<br>命名空间：System.ComponentModel.DataAnnotations|
|`[None]`|数据库不生成值。<br>使用示例：`[DatabaseGenerated(DatabaseGeneratedOption.None)]`<br>命名空间：System.ComponentModel.DataAnnotations.Schema|
|`[NotMapped]`|表示应从数据库映射中排除属性或类。<br>使用示例：`[NotMapped]`<br>命名空间：System.ComponentModel.DataAnnotations.Schema|
|`[Required]`|指定数据字段值是必需的。<br>使用示例：`[Required]`<br>命名空间：System.ComponentModel.DataAnnotations|
##### 使用示例
```C#
    /// <summary>
    /// 用户
    /// </summary>
    [Table("user")]
    public class User
    {
        [Column(Order = 0)]
        [Key]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        public int Id { get; set; }

        [Required]
        [Description("姓名。")]
        [Column("name", TypeName = "varchar(50)")]
        public string? Name { get; set; }

        [Description("年龄。")]
        public int? Age { get; set; }

        [Required]
        [DefaultValue(0)]
        [Description("显示顺序。")]
        public int? ShowOrder { get; set; }

        /// <summary>
        /// 用户类型。
        /// </summary>
        public UserType? UserType { get; set; }

        /// <summary>
        /// 订单列表。
        /// </summary>
        public List<Order>? Orders { get; set; } = new List<Order>();

        [Required]
        [DefaultValue(0)]
        [Description("是否逻辑删除。")]
        public sbyte? IsDeleted { get; set; }

        [Column(TypeName = "timestamp")]
        [Required]
        [DefaultValue("CURRENT_TIMESTAMP")]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        [Description("创建时间。")]
        public DateTime? CreateTime { get; set; }
    }
```
1. 设置了`[Key]`和`[Identity]`标签的int型属性，映射的数据列是自动增长列。
1. `[Key]`、`[Required]`标识的字段均为必填，无需对同一个属性重复设置这两个特性。
1. `[Identity]`标签（自动增长）只对int类型的列生效，非int类型列设置DefaultValue后，将忽略`[Identity]`标签。
1. 设置了`[Computed]`标签的属性映射列，在插入和更新时默认值均会被计算，没有设置`[Computed]`的默认值只在插入时计算。
1. 没有设置`[Column]`标签的属性，会根据数据库品牌，用属性名生成相应形式的列名，也会根据属性的数据类型生成相应的列类型及默认最大长度。
1. 插入数据时如果没有设置必填项的数值，该项目也没有默认值，Insert语句将根据`TypeName`的内容自动增加该项目类型的默认值。如果没有设置`TypeName`，则会根据属性类型分配最接近的字段类型。
1. 使用实体类插入记录时，推荐将映射的属性全部设置为可空类型。原因是实体类里的非空属性会自动产生默认值，有值的字段都会加入SQL语句中。
1. 在使用对象实例插入记录时，如果没有设置映射为时间戳类型字段的属性的值，会由于自动生成了最小时间而无法执行。解决方法1.将属性设置为可空类型。2.时间戳改为时间类型。3.设置该字段映射的属性值在时间戳允许的范围内。
1. 设置为自增长（使用了`[Identity]`标签）的int类型的属性（多见于Id字段）建议设置为可空类型“int？”，否则会在sql插入内容中包括值为0的该字段，但是可以执行成功。
1. 批量插入时，有默认值的必填字段映射的属性要全部设置或全部不设置数值。因为参数的集合中有一个对象设置了该字段，就会在SQL语句中出现该字段，其他未设置该字段的记录则因缺少参数而不能执行成功。
1. `[ForeignKey]`示例：
```C#
public class User
{
    public int Id { get; set; }
    ...
    public List<Order> Orders { get; set; }
}

public class Order
{
    public int OrderId { get; set; }
    [ForeignKey("Seller")]
    public int SellerId { get; set; }
    [ForeignKey("Buyer")]
    public int BuyerId { get; set; }
    ...
    public User Seller { get; set; }
    public User Buyer { get; set; }
}
```
#### 操作数据库和数据表
+ 使用Lambda表达式操作数据库和数据表。
##### 根据Model创建数据表（Code First）
```C#
int? result = vdb.CreateTableIfNotExist<User>().Execute();
```
##### 移除数据表
```C#
int? result = vdb.DropTable<User>().Execute();
```
##### 清空数据表
```C#
int? result = vdb.TruncateTable<User>().Execute();
```
##### 检查数据表是否存在
```C#
bool result = vdb.TableIsExist<User>().GetData();
```
##### 获取数据表架构
```C#
IEnumerable<dynamic> result = vdb.GetTableStructure("user").GetData();
```

#### 根据数据表创建数据仓库配套文件
+ 通过代码工具（`CodeTool`）对象，在指定位置创建.NET源文件。
+ 可指定源文件中使用的语种（C#、VB.net、F#）、是否覆盖同名文件（默认不覆盖）和保存位置等参数。
##### 实例化工具
```C#
var codeTool = new CodeTool(conn);
codeTool.Language = ProgrammingLanguage.CSharp; //如不指定则使用C#。
codeTool.NameSpace = "MyDemo";  // 如不指定则使用当前项目名称。
```
|CodeTool 的参数|说明|类型|默认值|
|-|-|-|-|
|`Connection`|数据连接|`DbConnection`|无|
|`Tables`|数据表的名称|`string[]`|全部表|
|`Types`|根据数据表名称生成相应的类名|`string[]`|无|
|`IgnoredColumns`|需要忽略的列|`string[]`|无|
|`NameSpace`|命名空间|`string`|当前项目名称|
|`BaseTypes`|基类或接口|`string[]`|无|
|`Language`|生成内容使用的编程语言|`ProgrammingLanguage`|CSharp|
##### 根据数据表创建Model（DB First）
```C#
var result1 = codeTool.CreateModel().ToFile();
```
##### 根据数据表创建仓库接口
```C#
var result2 = codeTool.CreateIRepository().ToFile();
```
##### 根据数据表创建仓库
```C#
var result3 = codeTool.Creatorepository().ToFile();
```
##### 插入注册Ioc代码依赖注入
```C#
var result4 = codeTool.RegisterIoC().InsertFile();
```
##### 根据数据表创建WebAPIController
```C#
var result5 = codeTool.CreateWebAPIController().ToFile();
```
|方法|方法说明|参数|参数说明|类型|默认值|
|-|-|-|-|-|-|
|`ToCodeString()`|将生成结果输出至字符串|无||||
|`ToFile()`|将生成结果保存至文件|`overwrite`|是否覆盖同名文件|`bool`|`false`|
|||`targetPath`|生成文件的保存路径|`string`|应用程序当前工作目录下的相应文件夹|
|`InsertFile()`|将生成结果插入已有文件|`targetPath`|需要插入代码的文件的保存路径|`string`|应用程序当前工作目录下的`Program.cs`文件的`var app = builder.Build();`语句之后|
### 联系方式
***email:cnxl@hotmail.com，我们将尽快回复。***
