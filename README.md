# Voy.DALBase Readme
[TOC]
# English
Simple ORM framework for MySql/Sql Server（Will add more） using lambda expressions.
## How to use
### Instantiate
```C#
using Microsoft.Data.SqlClient; //Connect to the SqlServer database.
using MySqlConnector;   //Connect to the Mysql database.
using System.Data.Common;
using Voy.DALBase.Tools;

//When you need to connect to the Mysql database.
DbConnection conn = new MySqlConnection("Server=XXX.XXX.XXX.XXX; Port=XXXX; Database=XXXX; Uid=XXXX; Pwd=XXXXXXXX; SslMode=Preferred;"); 

//When you need to connect to the SqlServer database.
DbConnection conn = new SqlConnection("Server=XXX.XXX.XXX.XXX; Port=XXXX; Database=XXXX; Uid=XXXX; Pwd=XXXXXXXX; SslMode=Preferred;"); 

VDB vdb = new VDB(conn);   //normal mode.
VDBSingleton.Instance.VDB = new VDB(conn);  //singleton mode.
builder.Register(c => new VDBService(conn)).As<IVDB>().SingleInstance();    //Used to register IoC.
```
### Public Methods
|Description||Get SQL statement|Get parameters of SQL statement|Execute data operation command|Execute data query command|Execute scalar query command|
|-|-|:-:|:-:|:-:|:-:|:-:|
||Method|GetSQLString()|GetParams()|Execute()|GetData()|ExecuteScalar()|
|Create database|CreateDatabase()|✓|✓|✓|||
|Remove database|DropDatabase()|✓|✓|✓|||
|Get table information in the database|GetDatabaseTables()|✓|✓||✓|✓|
|Create data table|CreateTable()|✓|✓|✓|||
|Remove data table|DropTable()|✓|✓|✓|||
|Clear data table|TruncateTable()|✓|✓|✓|✓|✓|
|Get the data table structure|GetTableStructure()|✓|✓||||
|Insert data|Insert()|✓|✓|✓|||
|Delete data|Delete()|✓|✓|✓|||
|Update data|Update()|✓|✓|✓|||
|Query data|Select()|✓|✓||✓|✓|
### Insert
```C#
//Insert data with Lambda expressions.
var ins1 = vdb.Insert<User>(u => new User { Name = "Tom", Age = 30 });
int i1 = ins1.Execute();

//Insert data with entity objects. (<TClass> can be omitted)
var ins2 = vdb.Insert(new User { Name = "Jerry", Age = 40 });
int i2 = ins2.Execute();

//Insert multiple data.
var ins3 = vdb.Insert<User>(new List<User>
{
    new User { Name = "Doctor", Age = 10, CreateTime = DateTime.Now },
    new User { Name = "Bashful ", Age = 20, CreateTime = DateTime.Now },
    new User { Name = "Sleepy", Age = 50, CreateTime = DateTime.Now },
    new User { Name = "Sneezy", Age = 60, CreateTime = DateTime.Now },
    new User { Name = "Happy", Age = 70, CreateTime = DateTime.Now },
    new User { Name = "Dopey", Age = 80, CreateTime = DateTime.Now },
    new User { Name = "Grumpy", Age = 90, CreateTime = DateTime.Now },
});
int i3 = ins3.Execute();
```
+ Usage: Insert<TClass>(), the parameter cannot be empty. The name of the method to obtain the result: Execute().
+ You can use Lambda expressions and entity objects as parameters to insert one record at a time, or use List<TClass> type parameters to insert multiple records in batches.
+ Returns the auto-increment primary key value when inserting a record. If the primary key is not self-increasing, return 0; when inserting multiple records, return the number of affected rows.
+ Nullable properties will have a null value inserted, non-nullable properties will have a default value inserted.
### Update
```C#
//Update data with Lambda expressions.
var upd1 = vdb.Update<User>(u => new User { Id = 1, Name = "Sleepy", Age = 50 });
int u1 = upd1.Execute();

//Update data with expression + Where method.
var upd2 = vdb.Update<User>(u => new User { Name = "Sneezy", Age = 60 }).Where(u => u.Name == "Jerry");
int u2 = upd2.Execute();

//Update data with an anonymous object.
var upd3 = vdb.Update<User>(new { Id = 1, Name = "Happy", Age = 70 });
int u3 = upd3.Execute();

//Update data with anonymous object update + Where method.
var upd4 = vdb.Update<User>(new { Name = "Dopey", Age = 80 }).Where(u => u.Name == "Sneezy");
int u4 = upd4.Execute();
```
+ Usage: Update<TClass>(), the parameter cannot be empty. The name of the method to obtain the result: Execute().  
+ You can use: Lambda expressions, anonymous objects as parameters to modify a record. Entity objects are currently not supported, and using entity objects will not execute successfully. For forward compatibility of subsequent versions, the parameter type continues to use dynamic.
+ Any column can be updated when using the anonymous object parameter and the "Where()" method to update the record; when only using the object parameter to update, the value of the primary key field will not be updated as a filter condition.
+ The attribute with the same name as "TClass" in the anonymous object parameter will not appear in the generated SQL statement if it has the "DatabaseGeneratedOption.Computed" label. 
+ When the primary key field is not included in the column of the anonymous object, and the Where statement is not used for update filtering, the update operation will not be executed for data security.
### Delete
```C#
//Delete data with "Where" method.
var del1 = vdb.Delete<User>().Where(u => u.Name == "周八");
int d1 = del1.Execute();

//Delete multiple data with "Where" method.
List<int> ids = new List<int>{ 1, 2 };
var del2 = vdb.Delete<User>().Where(u => ids.ToArray().Contains(u.Id));
int d2 = del2.Execute();

//Delete multiple data with IList<TClass>. (<TClass> can be omitted, All properties need to be assigned a value to take effect.)
var del3 = vdb.Delete(new List<User>() {
    new User(){ Id = 1, Name = "Tom", Age = 2},
    new User(){ Id = 2, Name= "Jerry", Age = 1 }
});
int d3 = del3.Execute();
```
+ Usage method: Delete<TClass>(), the parameter can be empty. The name of the method to obtain the result: Execute().
+ You can use Lambda expressions and entity objects as parameters to delete records, or use the Where method as filter conditions to delete records.
+ When using an entity object as a parameter to filter records, if the object's property is a numeric default value (such as 0 for int), it will have no effect. For example, do not use "new User { Age = 0 }". 
+ When there are parameters and the Where statement is used for update filtering, the filter conditions will be combined with AND.
+ When there is no parameter and no Where statement is used for update filtering, the update operation will not be executed for data security.

### Select
```C#
//Get all fields
var sel1 = vdb.Select<User>()
    .Where<User>(u => u.Name == "Happy")
    .OrderBy(u => u.Id)
    .OrderBy(u => u.Age)
    .Page(1, 10);
var items = sel1.GetData();

//Get the specified fields
var sel2 = vdb.Select<User>(u => new { u.Id, u.Name })
    .Where<User>(u => u.Name == "Happy")
    .OrderByDesc(u => u.Id)
    .OrderBy(u => u.Age)
    .Page(1, 10);
var items = sel1.GetData();

//Multi-table query
var sel1 = vdb.Select<User, Order>((u, o) => new { u.Id, u.Name, o.Amount, o.CreateTime })
    .LeftJoin((u, o) => u.Id == o.UserId)
    .Where((u, o) => u.Name == "Happy")
    .OrderBy((u, o) => u.Id)
    .OrderBy((u, o) => u.Age)
    .Page(1, 10);
var items = sel1.GetData();
```
+ Support multi-level master-slave table query, support one-to-many, many-to-one, one-to-one query, and support a maximum of 16 tables at a time.
+ Multi-table queries must have a Join statement, and the parameters in the Join statement have no order requirements.
+ The multi-table query always returns the set of the first formal parameter, so in order to return the result correctly, the Property of the model class whose type is the sub-table mapping should be added to the model class mapped by the main table, Property can be a single model or a generic collection of List\<sub-table models>.

|Method|Parameter|Reusable times|Remark|
|-|-|:-:|-|
|Where()|Lambda expressions|∞|When used multiple times, the filter condition relationship between methods is And.|
|WhereByBase()|Lambda expressions|∞|When the type used for filtering is uncertain and common attributes are required (for example, when it is agreed that each class has an Id). WhereByBase<TClass, TBase> The two type parameters are required.|
|GroupBy()|Lambda expressions|∞|groups rows that have the same values into summary rows, like "find the number of customers in each country".|
|OrderBy()|Lambda expressions|∞||
|OrderByDesc()|Lambda expressions|∞||
|InnerJoin()|Lambda expressions|∞|selects records that have matching values in both tables.|
|LeftJoin()|Lambda expressions|∞|returns all records from the left table (table1), and the matching records from the right table (table2). The result is 0 records from the right side, if there is no match.|
|RightJoin()|Lambda expressions|∞|returns all records from the right table (table2), and the matching records from the left table (table1). The result is 0 records from the left side, if there is no match.|
|FullOutJoin()|Lambda expressions|∞|returns all records when there is a match in left (table1) or right (table2) table records.|
|Page()|int pageIndex, int pageSize|1|The total number of records can be obtained using the "Count" property.|

### Operators For Filter（Where Method）
|Operator|Description|Usage example|
|:-:|-|-|
|==||u.Name == "Johnny"|
|!=||u.Name != "Johnny"|
|<||u.Age < 30|
|<=||u.Age <= 30|
|>||u.Age > 30|
|>=||u.Age >= 30|
|&&||u.Name == "Johnny" && u.Age == 30|
|\|\|||u.Name == "Johnny" \|\| u.Age == 30|
|StartsWith||u.Name.StartsWith("J")|
|EndsWith||u.Name.EndsWith("y")|
|Contains||u.Name.Contains("a") <br>new[] { 1, 2, 3, 4 }.Contains(u.Id)<br>Enumerable.Contains<int>(new[] { 1, 2, 3, }, u.Age)<br>Enumerable.Contains<string>(new[] { "Tom", "Jerry" }, u.Name)|
+ Methods can also be used multiple times, and the order of use is not required. The relationship between the filter conditions executed by multiple methods is And.
+ If there is an "OR" relationship, you can use the static method "Tools.ExpressionTools.CreateExpression<T>()" to create more flexible filter conditions as parameters. For example:
```C#
var e1 = ExpressionTools.CreateExpression<User>(u => u.Id == 1 || u.Name.Contains("z"));
var e2 = ExpressionTools.CreateExpression<User>(u => u.Age > 20);

var exp = e1.And(e2);
var v1 = vdb.Select<User>().Where(exp);
```
### Agg Func
```C#
var v1 = vdb.Select<User>()
.IF(u => u.IsDeleted == 0 ? "Not deleted" : "Deleted", "Deleted status")
.IFNULL(u => u.Name ?? "Empty Name")
var result = v1.GetData();
```
|Method|Description|Parameter|Reusable times|
|-|-|-|:-:|
|If()|If the judgment condition is true, return the first value after "?", otherwise return the second value.|Lambda expressions, Temporary column name (nullable)|∞|
|IfNull()|If the judgment object is empty, return the value after "??".|Lambda expressions|∞|

### Agg Func
```C#
var v1 = vdb.Select<User>()
.SUM(u => u.Age,)
.COUNT(u => u.IsDeleted == 0)
var result = v1.GetData();
```
|Method|Description|Parameter|Reusable times|
|-|-|-|:-:|
|Avg()|Calculates the average of a column.|Lambda expressions|∞|
|Count|Counts the number of items in the collection.|Lambda expressions|∞|
|Max()|Calculates the maximum value of a column.|Lambda expressions|∞|
|Min()|Calculate the minimum value of a column.|Lambda expressions|∞|
|Sum()|Calculates the total value of a column.|Lambda expressions|∞|
### About Attribute For Model
+ Use the [Table] and [Column] tags on the class and attribute to map the specified data table and data column.
+ If no data table/data column name is specified, VDB will apply different naming rules to map class name -> data table name, attribute name -> data column name according to the mapped database brand.

|Database Brand|Table Naming Rules|Table Name Example|Column Naming Rules|Column Name Example|
|-|-|-|-|-|
|MySQL|Lowercase words separated by underscores|UserSetting->\`user_setting\`|Lowercase words separated by underscores|UserId->\`user_id\`|
|SQL Server|Pascal|UserSetting->[UserSetting]|Camel|UserId->[userId]|
|SQLite|Pascal|UserSetting->[UserSetting]|Camel|UserId->[userId]|
|Access|Pascal|UserSetting->[UserSetting]|Camel|UserId->[userId]|
|Oracle|Capitalized words separated by underscores|UserSetting->"USER_SETTING"|Capitalized words separated by underscores|UserId->"USER_ID"|
#### Example of use
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
        [Description("Name")]
        [Column("name", TypeName = "varchar(50)")]
        public string? Name { get; set; }

        [Description("Age")]
        public int? Age { get; set; }

        [Required]
        [DefaultValue(0)]
        [Description("ShowOrder")]
        public int? ShowOrder { get; set; }

        /// <summary>
        /// UserType.
        /// </summary>
        public UserType? UserType { get; set; }

        /// <summary>
        /// OrderList
        /// </summary>
        public List<Order>? Orders { get; set; } = new List<Order>();

        [Required]
        [DefaultValue(0)]
        [Description("IsDelete")]
        public sbyte? IsDeleted { get; set; }

        [Column(TypeName = "timestamp")]
        [Required]
        [DefaultValue("CURRENT_TIMESTAMP")]
        [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
        [Description("CreateTime")]
        public DateTime? CreateTime { get; set; }
    }
```
#### Table
|Name|Description|Usage example|Namespace|
|-|-|-|-|
|Table|Indicates the database table to which the class will be mapped.|[Table("user_team")]|System.ComponentModel.DataAnnotations.Schema|
#### Column
|Name|Description|Usage example|Namespace|
|-|-|-|-|
|Column|Indicates the database column to which the attribute will be mapped.|[Column("id", Order = 0, TypeName = "int")]|System.ComponentModel.DataAnnotations.Schema|
|Computed|When a row is inserted or updated, the database generates a value.|[DatabaseGenerated(DatabaseGeneratedOption.Computed)]|System.ComponentModel.DataAnnotations.Schema|
|Identity|Indicates that a property or class should be excluded from database mapping.|[DatabaseGenerated(DatabaseGeneratedOption.Identity)]|System.ComponentModel.DataAnnotations.Schema|
|None|Represents one or more properties that uniquely identify an entity.|[DatabaseGenerated(DatabaseGeneratedOption.None)]|System.ComponentModel.DataAnnotations.Schema|
|NotMapped|Specifying a data field value is required.|[NotMapped]|System.ComponentModel.DataAnnotations.Schema|
|Key|Represents one or more properties that uniquely identify an entity.|[Key]|System.ComponentModel.DataAnnotations|
|Required|Specifying a data field value is required.|[Required]|System.ComponentModel.DataAnnotations|
|DefaultValue|Specifies the default value for the property.|[DefaultValue("CURRENT_TIMESTAMP")]|System.ComponentModel|
|Description|Specifies a description for the property or event.|[Description("name")]|System.ComponentModel|
#### Description
1. An int attribute with Key and Identity tags, and the mapped data column is an auto-growth column.
1. The fields identified by Key and Required are required and do not need to be set repeatedly.
1. The Identity label (automatic growth) is only valid for int type columns. After the DefaultValue is set for non-int type columns, the Identity label will be ignored.
1. For attributes without a Column tag, the corresponding column name will be generated from the attribute name according to the database brand, and the corresponding column type and default maximum length will also be generated according to the data type of the attribute.
1. For attribute mapping columns with the Computed tag set, the default value will be calculated when inserting and updating, and the default value for which Computed is not set is only calculated when inserting.
1. If the value of the required item is not set when inserting data, and the item has no default value, the Insert statement will automatically increase the default value of the item type according to the content of "TypeName". If TypeName is not set, the closest field type will be assigned based on the attribute type.
1. When using the entity class to insert records, it is recommended to set all mapped attributes to nullable types. The reason is that the non-empty attributes in the entity class will automatically generate default values, and the fields with values will be added to the SQL statement.
1. When inserting a record using an object instance, if the value of the attribute mapped to the timestamp type field is not set, it cannot be executed because the minimum time is automatically generated. Workaround 1. Set the property as a nullable type. 2. Change the timestamp to time type. 3. Set the attribute value of the field mapping within the range allowed by the timestamp.
1. It is recommended to set the int type attribute (mostly seen in the Id field) to the self-growth (Identity tag) as the nullable type "int?", otherwise the field with a value of 0 will be included in the SQL insertion content, but it can be executed successfully .
1. When inserting in batches, the attributes mapped to mandatory fields with default values must be set or not set at all. Because there is an object in the set of parameters with this field set, this field will appear in the SQL statement, and other records without this field cannot be executed successfully due to lack of parameters.

### DBFirst
```C#
var result = vdb.GenerateCode(
        new string[] { "user" },
        language: ProgrammingLanguage.CSharp)
    .Model(
        nameSpace: "MyProject", 
        baseTypes: new string[] { "BaseModel" }, 
        ignoredColumns: new string[] { "id" })
    .ToFile();
```
+ You can use the VDB.GenerateCode() method to generate codes according to the structure of the data table.
+ Use the ToString() method to get the code in text form. The generated code is output as the characters contained in the Value of the Dictionary, and the Key in the Dictionary is the table name.
+ Use the ToFile() method to automatically save the code to the specified file directory, and the default path is the project folder (Models\IServices).
+ Support specifying programming languages such as C# and Visual Basic.
+ Support for generating data model classes and service interfaces.
+ Support specifying the data table to be mapped, or you can ignore this parameter and scan all data tables to create all data models.
+ Support specifying namespace name.
+ Support adding inherited base classes, or implemented interfaces.
+ Support specifying data columns that need to be ignored.

## Contact Us
***email:cnxl@hotmail.com, we will reply as soon as possible.***

# 中文
适用于MySql/Sql Server（将添加更多），可使用Lambda表达式的简单ORM框架。
## 如何使用
### 实例化
```C#
using Microsoft.Data.SqlClient; //连接到SqlServer数据库。
using MySqlConnector;   //连接到Mysql数据库。
using System.Data.Common;
using Voy.DALBase.Tools;

//当你需要连接到Mysql数据库。
DbConnection conn = new MySqlConnection("Server=XXX.XXX.XXX.XXX; Port=XXXX; Database=XXXX; Uid=XXXX; Pwd=XXXXXXXX; SslMode=Preferred;"); 

//当你需要连接到SqlServer数据库。
DbConnection conn = new SqlConnection("Server=XXX.XXX.XXX.XXX; Port=XXXX; Database=XXXX; Uid=XXXX; Pwd=XXXXXXXX; SslMode=Preferred;"); 

VDB vdb = new VDB(conn);   //普通模式。
VDBSingleton.Instance.VDB = new VDB(conn);  //单例模式。
builder.Register(c => new VDBService(conn)).As<IVDB>().SingleInstance();    //用于注册IoC中。
```

### 公共方法
|说明||获取SQL语句|获取SQL语句的参数|执行数据操作命令|执行数据查询命令|执行标量查询命令|
|-|-|:-:|:-:|:-:|:-:|:-:|
||方法|GetSQLString()|GetParams()|Execute()|GetData()|ExecuteScalar()|
|创建数据库|CreateDatabase()|✓|✓|✓|||
|移除数据库|DropDatabase()|✓|✓|✓|||
|获取数据库中的表信息|GetDatabaseTables()|✓|✓||✓|✓|
|创建数据表|CreateTable()|✓|✓|✓|||
|移除数据表|DropTable()|✓|✓|✓|||
|清空数据表|TruncateTable()|✓|✓|✓|✓|✓|
|获取数据表结构|GetTableStructure()|✓|✓||||
|插入数据|Insert()|✓|✓|✓|||
|删除数据|Delete()|✓|✓|✓|||
|更新数据|Update()|✓|✓|✓|||
|查询数据|Select()|✓|✓||✓|✓|
### 插入
```C#
//使用Lambda表达式插入数据。
var ins1 = vdb.Insert<User>(u => new User { Name = "张三", Age = 30 });

//使用实体对象插入数据。（可省略<TClass>）
var ins2 = vdb.Insert(new User { Name = "李四", Age = 40 });

//批量插入数据。
var ins3 = vdb.Insert<User>(new List<User>
{
    new User { Name = "刘一", Age = 10, CreateTime = DateTime.Now },
    new User { Name = "陈二", Age = 20, CreateTime = DateTime.Now },
    new User { Name = "王五", Age = 50, CreateTime = DateTime.Now },
    new User { Name = "赵六", Age = 60, CreateTime = DateTime.Now },
    new User { Name = "孙七", Age = 70, CreateTime = DateTime.Now },
    new User { Name = "周八", Age = 80, CreateTime = DateTime.Now },
    new User { Name = "吴九", Age = 90, CreateTime = DateTime.Now },
    new User { Name = "郑十", Age = 100, CreateTime = DateTime.Now },
});
```
+ 使用方法：Insert<TClass>()，参数不可为空。获取结果方法名称：Execute()。
+ 可以使用Lambda表达式、实体对象做为参数单次插入一条记录，也可以用List<TClass>型参数批量插入多条记录。
+ 插入一条记录时返回自增主键值。如果主键不是自增长，返回0。插入多条记录时返回影响行数。
+ 可空的属性将插入空值，不可空属性将插入默认值。

### 更新
```C#
//用Lambda表达式更新数据。
var upd1 = vdb.Update<User>(u => new User { Id = 1, Name = "王五", Age = 50 });

//用表达式 + Where方法更新数据。
var upd2 = vdb.Update<User>(u => new User { Name = "赵六", Age = 60 }).Where(u => u.Name == "李四");

//用匿名对象更新数据。
var upd3 = vdb.Update<User>(new { Id = 1, Name = "孙七", Age = 70 });

//用匿名对象更新 + Where方法更新数据。
var upd4 = vdb.Update<User>(new { Name = "周八", Age = 80 }).Where(u => u.Name == "赵六");
```
+ 使用方法：Update<TClass>()，参数不可为空。获取结果方法名称：Execute()。
+ 可以使用Lambda表达式、匿名对象作为参数修改一条记录，目前不支持实体对象，使用实体对象将不能成功执行。为了后续版本向前兼容，参数类型持续使用dynamic。
+ 使用匿名对象参数和Where()方法更新记录时可以更新包括主键在内的任何列；只使用对象参数更新时，主键字段的值将作为筛选条件不进行更新。
+ 匿名对象参数中与“TClass”中同名的属性，如果带有“DatabaseGeneratedOption.Computed”标签将不会出现在生成的SQL语句中。
+ 匿名对象的列中未包括主键字段，也没有使用Where语句进行更新筛选时，为了数据安全更新操作将不会被执行。

### 删除
```C#
//用Lambda表达式删除数据。
var del1 = vdb.Delete<User>(u => new User { Id = 1 });

//用Where方法删除数据。
var del2 = vdb.Delete<User>().Where(u => u.Name == "周八");

//用实体对象删除数据。（可省略<TClass>）
var del3 = vdb.Delete(new User() { Name = "周八" });

//用Lambda表达式 + Where方法删除数据。
var del4 = vdb.Delete<User>(u => new User { Id = 1 }).Where(u => u.Name == "周八");

//用实体对象 + Where方法删除数据。
var del5 = vdb.Delete<User>(new User { Id = 1 }).Where(u => u.Name == "周八");
```
+ 使用方法：Delete<TClass>()，参数可为空。获取结果方法名称：Execute()。
+ 可以使用Lambda表达式、实体对象做为参数删除记录，也可以用Where方法作为筛选条件删除记录。
+ 使用实体对象为参数筛选记录时，如果对象的属性是数值型的默认值（例如int的0）则无效果，例如不要使用“new User { Age = 0 }”。
+ 当有参数，也使用Where语句进行更新筛选时，筛选条件将进行AND合并。
+ 当无参数，也没有使用Where语句进行更新筛选时，为了数据安全更新操作将不会被执行。

### 查询
```C#
//获取全部字段
var sel1 = vdb.Select<User>()
    .Where<User>(u => u.Name == "王五")
    .OrderBy(u => u.Id)
    .OrderBy(u => u.Age)
    .Page(1, 10);
var items = sel1.GetData();

//获取指定字段
var sel2 = vdb.Select<User>(u => new { u.Id, u.Name })
    .Where<User>(u => u.Name == "王五")
    .OrderByDesc(u => u.Id)
    .OrderBy(u => u.Age)
    .Page(1, 10);
var items = sel1.GetData();

//多表查询
var sel1 = vdb.Select<User, Order>((u, o) => new { u.Id, u.Name, o.Amount, o.CreateTime })
    .LeftJoin((u, o) => u.Id == o.UserId)
    .Where((u, o) => u.Name == "张三")
    .OrderBy((u, o) => u.Id)
    .OrderBy((u, o) => u.Age)
    .Page(1, 10);
var items = sel1.GetData();
```
+ 支持多级主从表查询，支持一对多、多对一、一对一查询，最大可支持一次查询16个表。
+ 多表查询必须要有Join语句，Join语句中的参数无顺序的要求。
+ 多表查询总是返回第一个形参的集合，所以为了正确返回结果，应在主表映射的模型类中增加类型为子表映射的模型类的属性，属性可以是单一模型或是List<子表模型>的泛型集合。

|方法|说明|参数|可使用次数|备注|
|-|-|-|:-:|-|
|Where()|通过属性进行筛选|Lambda表达式|∞|多次使用时，方法之间的筛选条件关系是And。|
|WhereByBase()|通过基类的属性进行筛选|Lambda表达式|∞|用于筛选的类型不确定，又需要使用通用属性（例如约定每个类都有Id）的时候。WhereByBase<TClass, TBase>两个类型参数为必填。|
|GroupBy()|分组|Lambda表达式|∞||
|OrderBy()|升序排序|Lambda表达式|∞||
|OrderByDesc()|降序排序|Lambda表达式|∞||
|InnerJoin()|联合查询|Lambda表达式|∞|如果表中有至少一个匹配，则返回行。|
|LeftJoin()|左联合查询|Lambda表达式|∞|即使右表中没有匹配，也从左表返回所有的行。|
|RightJoin()|右联合查询|Lambda表达式|∞|即使左表中没有匹配，也从右表返回所有的行。|
|FullOutJoin()||Lambda表达式|∞|只要其中一个表中存在匹配，则返回行。|
|Page()|筛选结果的分页|int pageIndex, int pageSize|1|引用"Count"属性获得记录总数。|
### 过滤器的运算符（Where 方法）
|运算符|说明|使用示例|
|:-:|-|-|
|==||u.Name == "张三"|
|!=||u.Name != "张三"|
|<||u.Age < 30|
|<=||u.Age <= 30|
|>||u.Age > 30|
|>=||u.Age >= 30|
|&&||u.Name == "张三" && u.Age == 30|
|\|\|||u.Name == "张三" \|\| u.Age == 30|
|StartsWith||u.Name.StartsWith("张")|
|EndsWith||u.Name.EndsWith("三")|
|Contains||u.Name.Contains("张") <br>new[] { 1, 2, 3, 4 }.Contains(u.Id)<br>Enumerable.Contains<int>(new[] { 1, 2, 3, }, u.Age)<br>Enumerable.Contains<string>(new[] { "张三", "李四" }, u.Name)|
+ 以上方法可以多次使用，使用顺序无要求。多个方法执行的筛选条件之关系为And。
+ 如果有“OR”的关系可以使用静态方法“Tools.ExpressionTools.CreateExpression<T>()”来创建更灵活的筛选条件作为参数。例如：
```C#
var e1 = ExpressionTools.CreateExpression<User>(u => u.Id == 1 || u.Name.Contains("z"));
var e2 = ExpressionTools.CreateExpression<User>(u => u.Age > 20);

var exp = e1.And(e2);
var v1 = vdb.Select<User>().Where(exp);
```
### 条件函数
```C#
var v1 = vdb.Select<User>()
.IF(u => u.IsDeleted == 0 ? "未删除" : "已删除", "删除状态")
.IFNULL(u => u.Name ?? "空名字")
var result = v1.GetData();
```
|方法|说明|参数|可使用次数|
|-|-|-|:-:|
|If()|判断条件如果为真，则返回?后面的第一个值，否则返回第二个值。|Lambda表达式，临时列名（可选）|∞|
|IfNull()|判断对象如果为空，则返回??后面的值。|Lambda表达式|∞|

### 聚合函数
```C#
var v1 = vdb.Select<User>()
.Sum(u => u.Age,)
.Count(u => u.IsDeleted == 0)
var result = v1.GetData();
```
|方法|说明|参数|可使用次数|
|-|-|-|:-:|
|Avg()|计算某个列的平均值。|Lambda表达式|∞|
|Count()|统计集合中的项目数。|Lambda表达式|∞|
|Max()|计算列的最大值。|Lambda表达式|∞|
|Min()|计算列的最小值。|Lambda表达式|∞|
|Sum()|计算列的合计值。|Lambda表达式|∞|

### 关于Model的属性
+ 在类和属性上分别使用[Table]和[Column]标签可以映射指定的数据表和数据列。
+ 在不指定数据表/数据列名称的情况下，VDB会根据映射的数据库品牌，应用不同的命名规则来映射类名->数据表名、属性名->数据列名。

|数据库品牌|表命名规则|表名举例|列命名规则|列名举例|
|-|-|-|-|-|
|MySQL|小写字母与下划线|UserSetting->\`user_setting\`|小写字母与下划线|UserId->\`user_id\`|
|SQL Server|Pascal|UserSetting->[UserSetting]|Camel|UserId->[userId]|
|SQLite|Pascal|UserSetting->[UserSetting]|Camel|UserId->[userId]|
|Access|Pascal|UserSetting->[UserSetting]|Camel|UserId->[userId]|
|Oracle|大写字母与下划线|UserSetting->"USER_SETTING"|大写字母与下划线|UserId->"USER_ID"|
#### 使用示例
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
#### 数据表
|名称|说明|使用示例|命名空间|
|-|-|-|-|
|Table|表示类将映射到的数据库表。|[Table("user_team")]|System.ComponentModel.DataAnnotations.Schema|
#### 数据列
|名称|说明|使用示例|命名空间|
|-|-|-|-|
|Column|表示属性将映射到的数据库列。|[Column("id", Order = 0, TypeName = "int")]|System.ComponentModel.DataAnnotations.Schema|
|Computed|在插入或更新一个行时，数据库会生成一个值。|[DatabaseGenerated(DatabaseGeneratedOption.Computed)]|System.ComponentModel.DataAnnotations.Schema|
|Identity|在插入一个行时，数据库会生成一个值。|[DatabaseGenerated(DatabaseGeneratedOption.Identity)]|System.ComponentModel.DataAnnotations.Schema|
|None|数据库不生成值。|[DatabaseGenerated(DatabaseGeneratedOption.None)]|System.ComponentModel.DataAnnotations.Schema|
|NotMapped|表示应从数据库映射中排除属性或类。|[NotMapped]|System.ComponentModel.DataAnnotations.Schema|
|Key|表示唯一标识实体的一个或多个属性。|[Key]|System.ComponentModel.DataAnnotations|
|Required|指定数据字段值是必需的。|[Required]|System.ComponentModel.DataAnnotations|
|DefaultValue|指定属性的默认值。|[DefaultValue("CURRENT_TIMESTAMP")]|System.ComponentModel|
|Description|指定属性或事件的说明。|[Description("name")]|System.ComponentModel|
#### 说明
1. 设置了Key和Identity标签的int型属性，映射的数据列是自动增长列。
1. Key、Required标识的字段均为必填，无需对同一个属性重复设置这两个特性。
1. Identity标签（自动增长）只对int类型的列生效，非int类型列设置DefaultValue后，将忽略Identity标签。
1. 设置了Computed标签的属性映射列，在插入和更新时默认值均会被计算，没有设置Computed的默认值只在插入时计算。
1. 没有设置Column标签的属性，会根据数据库品牌，用属性名生成相应形式的列名，也会根据属性的数据类型生成相应的列类型及默认最大长度。
1. 插入数据时如果没有设置必填项的数值，该项目也没有默认值，Insert语句将根据“TypeName”的内容自动增加该项目类型的默认值。如果没有设置TypeName，则会根据属性类型分配最接近的字段类型。
1. 使用实体类插入记录时，推荐将映射的属性全部设置为可空类型。原因是实体类里的非空属性会自动产生默认值，有值的字段都会加入SQL语句中。
1. 在使用对象实例插入记录时，如果没有设置映射为时间戳类型字段的属性的值，会由于自动生成了最小时间而无法执行。解决方法1.将属性设置为可空类型。2.时间戳改为时间类型。3.设置该字段映射的属性值在时间戳允许的范围内。
1. 设置为自增长（Identity标签）的int类型的属性（多见于Id字段）建议设置为可空类型“int？”，否则会在sql插入内容中包括值为0的该字段，但是可以执行成功。
1. 批量插入时，有默认值的必填字段映射的属性要全部设置或全部不设置数值。因为参数的集合中有一个对象设置了该字段，就会在SQL语句中出现该字段，其他未设置该字段的记录则因缺少参数而不能执行成功。
### DBFirst
```C#
var result = vdb.GenerateCode(
        new string[] { "user" },
        language: ProgrammingLanguage.CSharp)
    .Model(
        nameSpace: "MyProject", 
        baseTypes: new string[] { "BaseModel" }, 
        ignoredColumns: new string[] { "id" })
    .ToFile();
```
+ 可以使用VDB.GenerateCode()方法,根据数据表的结构来生成代码。
+ 使用ToString()方法获得文字形式的代码。生成的代码输出为包含在Dictionary的Value中的字符，Dictionary中的Key是表名。
+ 使用ToFile()方法可以让代码自动保存到指定的文件目录中，默认路径是项目的文件夹中(Models\IServices)。
+ 支持指定使用C#、Visual Basic等编程语言。
+ 支持生成数据模型类、服务接口。
+ 支持指定要映射的数据表，也可以忽略这个参数而扫描全部数据表，创建全部的数据模型。
+ 支持指定命名空间名称。
+ 支持添加继承的基类，或实现的接口。
+ 支持指定需要忽略的数据列。

##联系我们
***email:cnxl@hotmail.com，我们将尽快回复。***
