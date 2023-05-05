# Voy.DALBase Readme
Simple ORM framework for MySql/Sql Server（Will add more） using lambda expressions.
(You can contact my email:cnxl@hotmail.com)
## How to use
### Instantiate
```C#
using Microsoft.Data.SqlClient; //Connect to the SqlServer database.
using MySqlConnector;   //Connect to the Mysql database.
using System.Data.Common;

//When you need to connect to the Mysql database.
DbConnection conn = new MySqlConnection("Server=XXX.XXX.XXX.XXX; Port=XXXX; Database=XXXX; Uid=XXXX; Pwd=XXXXXXXX; SslMode=Preferred;"); 

//When you need to connect to the SqlServer database.
DbConnection conn = new SqlConnection("Server=XXX.XXX.XXX.XXX; Port=XXXX; Database=XXXX; Uid=XXXX; Pwd=XXXXXXXX; SslMode=Preferred;"); 

VDB vdb = new VDB(conn));
```

### Public Methods

|Method Name|Description|GetSQLString()|GetParams()|Execute()|GetData()|ExecuteScalar()|
|-|-|:-:|:-:|:-:|:-:|:-:|
|方法名称|说明|获取SQL语句|获取SQL语句的参数|执行数据操作命令|执行数据查询命令|执行标量查询命令|
|CreateDatabase|创建数据库|✓|✓|✓|||
|DropDatabase|移除数据库|✓|✓|✓|||
|GetDatabaseTables|获取数据库中的表信息|✓|✓||✓|✓|
|CreateTable|创建数据表|✓|✓|✓|||
|DropTable|移除数据表|✓|✓|✓|||
|TruncateTable|清空数据表|✓|✓|✓|✓|✓|
|GetTableStructure|获取数据表结构|✓|✓||||
|Insert|插入数据|✓|✓|✓|||
|Delete|删除数据|✓|✓|✓|||
|Update|更新数据|✓|✓|✓|||
|Select|查询数据|✓|✓||✓|✓|

### Insert
```C#
//Insert data with Lambda expressions.
//使用Lambda表达式插入数据。
var ins1 = vdb.Insert<User>(u => new User { Name = "张三", Age = 30 });
int i1 = ins1.Execute();

//Insert data with entity objects. (<TClass> can be omitted)
//使用实体对象插入数据。（可省略<TClass>）
var ins2 = vdb.Insert(new User { Name = "李四", Age = 40 });
int i2 = ins2.Execute();

//Insert multiple data.
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
int i3 = ins3.Execute();
```
+ Usage: Insert<TClass>(), the parameter cannot be empty. The name of the method to obtain the result: Execute().  
使用方法：Insert<TClass>()，参数不可为空。获取结果方法名称：Execute()。
+ You can use Lambda expressions and entity objects as parameters to insert one record at a time, or use List<TClass> type parameters to insert multiple records in batches.  
可以使用Lambda表达式、实体对象做为参数单次插入一条记录，也可以用List<TClass>型参数批量插入多条记录。
+ Returns the auto-increment primary key value when inserting a record. If the primary key is not self-increasing, return 0; when inserting multiple records, return the number of affected rows.  
插入一条记录时返回自增主键值。如果主键不是自增长，返回0；插入多条记录时返回影响行数。
+ Nullable properties will have a null value inserted, non-nullable properties will have a default value inserted.  
可空的属性将插入空值，不可空属性将插入默认值。

### Update
```C#
//Update data with Lambda expressions.
//用Lambda表达式更新数据。
var upd1 = vdb.Update<User>(u => new User { Id = 1, Name = "王五", Age = 50 });
int u1 = upd1.Execute();

//Update data with expression + Where method.
//用表达式 + Where方法更新数据。
var upd2 = vdb.Update<User>(u => new User { Name = "赵六", Age = 60 }).Where(u => u.Name == "李四");
int u2 = upd2.Execute();

//Update data with an anonymous object.
//用匿名对象更新数据。
var upd3 = vdb.Update<User>(new { Id = 1, Name = "孙七", Age = 70 });
int u3 = upd3.Execute();

//Update data with anonymous object update + Where method.
//用匿名对象更新 + Where方法更新数据。
var upd4 = vdb.Update<User>(new { Name = "周八", Age = 80 }).Where(u => u.Name == "赵六");
int u4 = upd4.Execute();
```
+ Usage: Update<TClass>(), the parameter cannot be empty. The name of the method to obtain the result: Execute().  
使用方法：Update<TClass>()，参数不可为空。获取结果方法名称：Execute()。
+ You can use: Lambda expressions, anonymous objects as parameters to modify a record. Entity objects are currently not supported, and using entity objects will not execute successfully. For forward compatibility of subsequent versions, the parameter type continues to use dynamic.
可以使用：Lambda表达式、匿名对象作为参数修改一条记录。目前不支持实体对象，使用实体对象将不能成功执行。为了后续版本向前兼容，参数类型持续使用dynamic。
+ Any column can be updated when using the anonymous object parameter and the "Where()" method to update the record; when only using the object parameter to update, the value of the primary key field will not be updated as a filter condition.
使用匿名对象参数和Where()方法更新记录时可以更新任何列；只使用对象参数更新时，主键字段的值将作为筛选条件不进行更新。
+ The attribute with the same name as "TClass" in the anonymous object parameter will not appear in the generated SQL statement if it has the "DatabaseGeneratedOption.Computed" label. 
匿名对象参数中与“TClass”中同名的属性，如果带有“DatabaseGeneratedOption.Computed”标签将不会出现在生成的SQL语句中。
+ When the primary key field is not included in the column of the anonymous object, and the Where statement is not used for update filtering, the update operation will not be executed for data security.
匿名对象的列中未包括主键字段，同时没有使用Where语句进行更新筛选时，为了数据安全更新操作将不会被执行。

### Delete
```C#
//Delete data with "Where" method.
//用Where方法删除数据。
var del1 = vdb.Delete<User>().Where(u => u.Name == "周八");
int d1 = del1.Execute();

//Delete multiple data with "Where" method.
//用Where方法批量删除数据。
List<int> ids = new List<int>{ 1, 2 };
var del2 = vdb.Delete<User>().Where(u => ids.ToArray().Contains(u.Id));
int d2 = del2.Execute();

//Delete multiple data with IList<TClass>. (<TClass> can be omitted)
//用集合参数批量删除数据。（可省略<TClass>，需要给所有属性赋值才能生效。）
var del3 = vdb.Delete(new List<User>() {
    new User(){ Id = 1, Name = "Tom", Age = 2},
    new User(){ Id = 2, Name= "Jerry", Age = 1 }
});
int d3 = del3.Execute();
```
+ Usage method: Delete<TClass>(), the parameter can be empty. The name of the method to obtain the result: Execute().
使用方法：Delete<TClass>()，参数可为空。获取结果方法名称：Execute()。
+ You can use Lambda expressions and entity objects as parameters to delete records, or use the Where method as filter conditions to delete records.
可以使用Lambda表达式、实体对象做为参数删除记录，也可以用Where方法作为筛选条件删除记录。
+ When using an entity object as a parameter to filter records, if the object's property is a numeric default value (such as 0 for int), it will have no effect. For example, do not use "new User { Age = 0 }". 
使用实体对象为参数筛选记录时，如果对象的属性是数值型的默认值（例如int的0）则无效果，例如不要使用“new User { Age = 0 }”。
+ When there are parameters and the Where statement is used for update filtering, the filter conditions will be combined with AND.
当有参数，同时使用Where语句进行筛选时，筛选条件将进行AND合并。
+ When there is no parameter and no Where statement is used for update filtering, the update operation will not be executed for data security.
当无参数，同时没有使用Where语句进行筛选时，为了数据安全更新操作将不会被执行。

### Select
```C#
//Get all fields
//获取全部字段
var sel1 = vdb.Select<User>()
    .Where<User>(u => u.Name == "王五")
    .OrderBy(u => u.Id)
    .OrderBy(u => u.Age)
    .Page(1, 10);

//Get the specified fields
//获取指定字段
var sel2 = vdb.Select<User>(u => new { u.Id, u.Name })
    .Where<User>(u => u.Name == "王五")
    .OrderByDesc(u => u.Id)
    .OrderBy(u => u.Age)
    .Page(1, 10);
```
|Method Name|Description|Parameter|Reusable times|Remark|
|-|-|-|:-:|-|
|方法|说明|参数|可使用次数|备注|
|InnerJoin|联合查询|Lambda表达式|∞|如果表中有至少一个匹配，则返回行。|
|LeftJoin|左联合查询|Lambda表达式|∞|即使右表中没有匹配，也从左表返回所有的行。|
|RightJoin|右联合查询|Lambda表达式|∞|即使左表中没有匹配，也从右表返回所有的行。|
|FullOutJoin||Lambda表达式|∞|只要其中一个表中存在匹配，则返回行。|
|Where|通过属性进行筛选|Lambda表达式|∞|多次使用时，方法之间的筛选条件关系是And。|
|WhereByBase|通过基类的属性进行筛选|Lambda表达式|∞|用于筛选的类型不确定，又需要使用通用属性（例如约定每个类都有Id）的时候。WhereByBase<TClass, TBase>两个类型参数为必填。|
|GroupBy|分组|Lambda表达式|∞||
|OrderBy|升序排序|Lambda表达式|∞||
|OrderByDesc|降序排序|Lambda表达式|∞||
|Page|筛选结果的分页|int pageIndex, int pageSize|1|引用Total属性获得记录总数。|

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
|Contains||u.Name.Contains("a") <br>new[] { 1, 2, 3, 4 }.Contains(u.Id)<br>Enumerable.Contains<int>(new[] { 1, 2, 3, }, u.Age)<br>Enumerable.Contains<string>(new[] { "张三", "李四" }, u.Name)|
+ Methods can also be used multiple times, and the order of use is not required. The relationship between the filter conditions executed by multiple methods is And.方法也可以多次使用，使用顺序无要求。多个方法执行的筛选条件之关系为And。
+ If there is an "OR" relationship, you can use the static method "Tools.ExpressionTools.CreateExpression<T>()" to create more flexible filter conditions as parameters. 如果有“OR”的关系可以使用静态方法“Tools.ExpressionTools.CreateExpression<T>()”来创建更灵活的筛选条件作为参数。For example:
```C#
var e1 = ExpressionTools.CreateExpression<User>(u => u.Id == 1 || u.Name.Contains("z"));
var e2 = ExpressionTools.CreateExpression<User>(u => u.Age > 20);

var exp = e1.And(e2);
var v1 = vdb.Select<User>().Where(exp);
```

### Agg
```C#
var v1 = vdb.Select<User>()
.SUM(u => u.Age,)
.COUNT(u => u.IsDeleted == 0)
var result = v1.GetData();
```
|Method Name|Description|Parameter|Reusable times|
|-|-|-|:-:|
|方法|说明|参数|可使用次数|
|Avg|计算某个列的平均值。|Lambda表达式|∞|
|Count|统计集合中的项目数。|Lambda表达式|∞|
|Max|计算列的最大值。|Lambda表达式|∞|
|Min|计算列的最小值。|Lambda表达式|∞|
|Sum|计算列的合计值。|Lambda表达式|∞|

## About Attribute For Model
### Table
|Name|说明|Description|Usage example|Namespace|
|-|-|-|-|-|
|Table|表示类将映射到的数据库表。|Indicates the database table to which the class will be mapped.|[Table("user_team")]|System.ComponentModel.DataAnnotations.Schema|
### Column
|Name|说明|Description|Usage example|Namespace|
|-|-|-|-|-|
|Column|表示属性将映射到的数据库列。|Indicates the database column to which the attribute will be mapped.|[Column("id", Order = 0, TypeName = "int")]|System.ComponentModel.DataAnnotations.Schema|
|Computed|在插入或更新一个行时，数据库会生成一个值。|When a row is inserted or updated, the database generates a value.|[DatabaseGenerated(DatabaseGeneratedOption.Computed)]|System.ComponentModel.DataAnnotations.Schema|
|Identity|在插入一个行时，数据库会生成一个值。|Indicates that a property or class should be excluded from database mapping.|[DatabaseGenerated(DatabaseGeneratedOption.Identity)]|System.ComponentModel.DataAnnotations.Schema|
|None|数据库不生成值。|Represents one or more properties that uniquely identify an entity.|[DatabaseGenerated(DatabaseGeneratedOption.None)]|System.ComponentModel.DataAnnotations.Schema|
|NotMapped|表示应从数据库映射中排除属性或类。|Specifying a data field value is required.|[NotMapped]|System.ComponentModel.DataAnnotations.Schema|
|Key|表示唯一标识实体的一个或多个属性。|Represents one or more properties that uniquely identify an entity.|[Key]|System.ComponentModel.DataAnnotations|
|Required|指定数据字段值是必需的。|Specifying a data field value is required.|[Required]|System.ComponentModel.DataAnnotations|
|DefaultValue|指定属性的默认值。|Specifies the default value for the property.|[DefaultValue("CURRENT_TIMESTAMP")]|System.ComponentModel|
|Description|指定属性或事件的说明。|Specifies a description for the property or event.|[Description("name")]|System.ComponentModel|

+ 有Key和Identity标签的int型属性，映射的数据列是自动增长列。
+ Required、Key标识的字段均为必填，无需重复设置。
+ 没有设置Column标签的属性，会根据数据库品牌，用属性名生成相应形式的列名，也会根据属性的数据类型生成相应的列类型及默认最大长度。
+ Identity标签（自动增长）只对int类型的列生效，非int类型列设置DefaultValue后，将忽略Identity标签。
+ 设置了Computed标签的属性映射列，在插入和更新时默认值均会被计算，没有设置Computed的默认值只在插入时计算。
+ 插入数据时如果没有设置必填项的数值，该项目也没有默认值，Insert语句将根据“TypeName”的内容自动增加该项目类型的默认值。如果没有设置TypeName，则会根据属性类型分配最接近的字段类型。
+ 使用实体类插入记录时，推荐将映射的属性全部设置为可空类型。原因是实体类里的非空属性会自动产生默认值，有值的字段都会加入SQL语句中。
+ 在使用对象实例插入记录时，如果没有设置映射为时间戳类型字段的属性的值，会由于自动生成了最小时间而无法执行。解决方法1.将属性设置为可空类型。2.时间戳改为时间类型。3.设置该字段映射的属性值在时间戳允许的范围内。
+ 设置为自增长（Identity标签）的int类型的属性（多见于Id字段）建议设置为可空类型“int？”，否则会在sql插入内容中包括值为0的该字段，但是可以执行成功。
+ 批量插入时，有默认值的必填字段映射的属性要全部设置或全部不设置数值。因为参数的集合中有一个对象设置了该字段，就会在SQL语句中出现该字段，其他未设置该字段的记录则因缺少参数而不能执行成功。

+ An int attribute with Key and Identity tags, and the mapped data column is an auto-growth column.
+ The fields identified by Required and Key are required and do not need to be set repeatedly.
+ For attributes without a Column tag, the corresponding column name will be generated from the attribute name according to the database brand, and the corresponding column type and default maximum length will also be generated according to the data type of the attribute.
+ The Identity label (automatic growth) is only valid for int type columns. After the DefaultValue is set for non-int type columns, the Identity label will be ignored.
+ For attribute mapping columns with the Computed tag set, the default value will be calculated when inserting and updating, and the default value for which Computed is not set is only calculated when inserting.
+ If the value of the required item is not set when inserting data, and the item has no default value, the Insert statement will automatically increase the default value of the item type according to the content of "TypeName". If TypeName is not set, the closest field type will be assigned based on the attribute type.
+ When using the entity class to insert records, it is recommended to set all mapped attributes to nullable types. The reason is that the non-empty attributes in the entity class will automatically generate default values, and the fields with values will be added to the SQL statement.
+ When inserting a record using an object instance, if the value of the attribute mapped to the timestamp type field is not set, it cannot be executed because the minimum time is automatically generated. Workaround 1. Set the property as a nullable type. 2. Change the timestamp to time type. 3. Set the attribute value of the field mapping within the range allowed by the timestamp.
+ It is recommended to set the int type attribute (mostly seen in the Id field) to the self-growth (Identity tag) as the nullable type "int?", otherwise the field with a value of 0 will be included in the SQL insertion content, but it can be executed successfully .
+ When inserting in batches, the attributes mapped to mandatory fields with default values must be set or not set at all. Because there is an object in the set of parameters with this field set, this field will appear in the SQL statement, and other records without this field cannot be executed successfully due to lack of parameters.
