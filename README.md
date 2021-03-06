# SqlSugar 4.X  API

In addition to EF, the most powerful ORM

Support：MySql、SqlServer、Sqlite、Oracle、postgresql

## Contactinfomation  
Email:610262374@qq.com 
QQ Group:225982985

## Nuget 

.net Install：
```ps1
Install-Package sqlSugar 
```
.net core Install：
```ps1
Install-Package sqlSugarCore
```

## SqlSugar's 16 Functions
There are 16 methods under SqlSugarClient

![输入图片说明](http://www.codeisbug.com/_theme/ueditor/utf8-net/net/upload/image/20190429/6369214497126656989458119.jpg "")

 

## Create SqlSugarClient
All operations are based on SqlSugarClient
```cs
public  List<Student> GetStudentList()
{
    var db= GetInstance();
    var list= db.Queryable<Student>().ToList();//Search
    return list;
}

/// <summary>
/// Create SqlSugarClient
/// </summary>
/// <returns></returns>
private SqlSugarClient GetInstance()
{
    SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
        {
            ConnectionString = "Server=.xxxxx",
            DbType = DbType.SqlServer,
            IsAutoCloseConnection = true,
            InitKeyType = InitKeyType.Attribute
        });
    //Print sql
    db.Aop.OnLogExecuting = (sql, pars) =>
    {
        Console.WriteLine(sql + "\r\n" + db.Utilities.SerializeObject(pars.ToDictionary(it => it.ParameterName, it => it.Value)));
        Console.WriteLine();
    };
    return db;
}

public class Student
{
    [SugarColumn(IsPrimaryKey = true, IsIdentity = true]
    public int Id { get; set; }
    public int? SchoolId { get; set; }
    public string Name { get; set; }
}
```
 
 

##  1. Queryable
We use it to query
 
```cs
//easy
var getAll = db.Queryable<Student>().ToList();
var getAllNoLock = db.Queryable<Student>().With(SqlWith.NoLock).ToList();
var getByPrimaryKey = db.Queryable<Student>().InSingle(2);
var sum = db.Queryable<Student>().Sum(it=>it.Id);
var isAny = db.Queryable<Student>().Where(it=>it.Id==-1).Any();
var isAny2 = db.Queryable<Student>().Any(it => it.Id == -1);
var getListByRename = db.Queryable<School>().AS("Student").ToList();
var getByWhere = db.Queryable<Student>().Where(it => it.Id == 1 || it.Name == "a").ToList();
var getByFuns = db.Queryable<Student>().Where(it => SqlFunc.IsNullOrEmpty(it.Name)).ToList();
var group = db.Queryable<Student>().GroupBy(it => it.Id).Select(it =>new { id = SqlFunc.AggregateCount(it.Id) }).ToList();

//Page
var page = db.Queryable<Student>().ToPageList(pageIndex, pageSize, ref totalCount);

//page join
var pageJoin = db.Queryable<Student, School>((st, sc) =>new JoinQueryInfos(JoinType.Left,st.SchoolId==sc.Id))
.ToPageList(pageIndex, pageSize, ref totalCount);

//top 5
var top5 = db.Queryable<Student>().Take(5).ToList();

//join Order By (order by st.id desc,sc.id desc)
var list4 = db.Queryable<Student, School>((st, sc) =>new  JoinQueryInfos(JoinType.Left,st.SchoolId==sc.Id))
.OrderBy(st=>st.Id,OrderByType.Desc)
.OrderBy((st,sc)=>sc.Id,OrderByType.Desc)
.Select<ViewModelStudent>().ToList();

```
[<font color=red>View more >> </font>](https://github.com/sunkaixuan/SqlSugar/wiki/1.Queryable) 
 


 ##  2. Updateable
We use it to Update
 ```cs
//update reutrn Update Count
var t1= db.Updateable(updateObj).ExecuteCommand();

//Only  update  Name 
var t3 = db.Updateable(updateObj).UpdateColumns(it => new { it.Name }).ExecuteCommand();

//Ignore  Name and TestId
var t4 = db.Updateable(updateObj).IgnoreColumns(it => new { it.Name, it.TestId }).ExecuteCommand();

//update List<T>
var t7 = db.Updateable(updateObjs).ExecuteCommand();

//Where By Expression
var t9 = db.Updateable(updateObj).Where(it => it.Id == 1).ExecuteCommand();

 ```
 [<font color=red>View more >> </font>](https://github.com/sunkaixuan/SqlSugar/wiki/2.Updateable) 

 
##  3. Insertable
We use it to Insert
 ```cs
//Insert reutrn Insert Count
var t2 = db.Insertable(insertObj).ExecuteCommand();

//Insert reutrn Identity Value
var t3 = db.Insertable(insertObj).ExecuteReutrnIdentity();

//Only  insert  Name 
var t4 = db.Insertable(insertObj).InsertColumns(it => new { it.Name,it.SchoolId }).ExecuteReutrnIdentity();

//Ignore TestId
var t5 = db.Insertable(insertObj).IgnoreColumns(it => new { it.Name, it.TestId }).ExecuteReutrnIdentity();

//Insert List<T>
var s9 = db.Insertable(insertObjs).InsertColumns(it => new { it.Name }).ExecuteCommand();
```
 [<font color=red>View more >> </font>](https://github.com/sunkaixuan/SqlSugar/wiki/3.Insertable) 
 
##  4. Deleteable
We use it to Delete

 ```cs
var t1 = db.Deleteable<Student>().Where(new Student() { Id = 1 }).ExecuteCommand();

//use lock
var t2 = db.Deleteable<Student>().With(SqlWith.RowLock).ExecuteCommand();


//by primary key
var t3 = db.Deleteable<Student>().In(1).ExecuteCommand();

//by primary key array
var t4 = db.Deleteable<Student>().In(new int[] { 1, 2 }).ExecuteCommand();

//by expression
var t5 = db.Deleteable<Student>().Where(it => it.Id == 1).ExecuteCommand();
 ```


 ##  5. SqlQueryable
```cs
var list = db.SqlQueryable<Student>("select * from student").ToPageList(1, 2);
var list2 = db.SqlQueryable<Student>("select * from student").Where(it=>it.Id==1).ToPageList(1, 2);
var list3= db.SqlQueryable<Student>("select * from student").Where("id=@id",new { id=1}).ToPageList(1, 2);
``` 
 
  ##  6. SaveQueues
  Perform multiple operations together with transactions
```cs
var db = GetInstance();
db.Insertable<Student>(new Student() { Name = "a" }).AddQueue();
db.Insertable<Student>(new Student() { Name = "b" }).AddQueue();
db.SaveQueues(); 

db.Insertable<Student>(new Student() { Name = "a" }).AddQueue();
db.Insertable<Student>(new Student() { Name = "b" }).AddQueue();
db.Insertable<Student>(new Student() { Name = "c" }).AddQueue();
db.Insertable<Student>(new Student() { Name = "d" }).AddQueue();
var ar = db.SaveQueuesAsync(); 

db.Queryable<Student>().AddQueue();
db.Queryable<School>().AddQueue();
db.AddQueue("select * from student where id=@id", new { id = 1 }); 
var result2 = db.SaveQueues<Student, School, Student>();  
```

##  7.Ado
db.Ado.MethodName，Look at the following example
```cs
var dt=db.Ado.GetDataTable("select * from table where id=@id and name=@name",new List<SugarParameter>(){
  new SugarParameter("@id",1),
  new SugarParameter("@name",2)
});
var dt=db.Ado.GetDataTable("select * from table where id=@id and name=@name",new{id=1,name=2});

//Use Stored Procedure
var dt2 = db.Ado.UseStoredProcedure().GetDataTable("sp_school",new{name="张三",age=0});//  GetInt SqlQuery<T>  等等都可以用
var nameP= new SugarParameter("@name", "张三");
var ageP= new SugarParameter("@age", null, true);//isOutput=true
var dt2 = db.Ado.UseStoredProcedure().GetDataTable("sp_school",nameP,ageP);
```
 
 ##  8.Saveable
 Insert or Update
```cs
db.Saveable<Student>(entity).ExecuteReturnEntity();
db.Saveable<Student>(new Student() { Name = "" })
                  .InsertColumns(it=>it.Name)
                  .UpdateColumns(it=>new { it.Name,it.CreateTime }
                  .ExecuteReturnEntity();

```
 
  ##  9.EntityMain
  ```cs
var entityInfo=db.EntityMaintenance.GetEntityInfo<Student>();
foreach (var column in entityInfo.Columns)
{
    Console.WriteLine(column.ColumnDescription);
}
```
  ##  10.DbMain
   ```cs
  var tables = db.DbMaintenance.GetTableInfoList();
  foreach (var table in tables)
  {
        Console.WriteLine(table.Description);
  }
  ```
  

  ##  11.Aop
  ```cs
db.Aop.OnLogExecuted = (sql, pars) => //SQL executed event
{
 
};
db.Aop.OnLogExecuting = (sql, pars) => //SQL executing event (pre-execution)
{
 
};
db.Aop.OnError = (exp) =>//SQL execution error event
{
                 
};
db.Aop.OnExecutingChangeSql = (sql, pars) => //SQL executing event (pre-execution,SQL script can be modified)
{
    return new KeyValuePair<string, SugarParameter[]>(sql,pars);
};

```

  ##  12.QueryFilter
  ```cs

 //gobal filter
var db = GetInstance();
var sql = db.Queryable<Student>().ToSql();
//SELECT [ID],[SchoolId],[Name],[CreateTime] FROM [STudent]  WHERE  isDelete=0 


public static SqlSugarClient GetInstance()
{
    SqlSugarClient db = new SqlSugarClient(new ConnectionConfig() {xxx);
            db.QueryFilter.Add(new SqlFilterItem()
             {
                 FilterValue = filterDb =>
                 {
                     return new SqlFilterResult() { Sql = " isDelete=0" };
                 }
             });
            return db;
}
 ```
  ##  13.DbFirst
  ```cs
var db = GetInstance();
//Create all class
db.DbFirst.CreateClassFile("c:\\Demo\\1");

//Create student calsss
db.DbFirst.Where("Student").CreateClassFile("c:\\Demo\\2");
//Where(array)

//Mapping name
db.MappingTables.Add("ClassStudent", "Student");
db.MappingColumns.Add("NewId", "Id", "ClassStudent");
db.DbFirst.Where("Student").CreateClassFile("c:\\Demo\\3");

//Remove mapping
db.MappingTables.Clear();

//Create class with default value
db.DbFirst.IsCreateDefaultValue().CreateClassFile("c:\\Demo\\4", "Demo.Models");


//Mapping and Attribute
db.MappingTables.Add("ClassStudent", "Student");
db.MappingColumns.Add("NewId", "Id", "ClassStudent");
db.DbFirst.IsCreateAttribute().Where("Student").CreateClassFile("c:\\Demo\\5");


```
  ##  14.CodeFirst
```cs
db.CodeFirst.SetStringDefaultLength(100).BackupTable().InitTables(typeof(CodeTable),typeof(CodeTable2)); //change entity backupTable
db.CodeFirst.SetStringDefaultLength(100).InitTables(typeof(CodeTable), typeof(CodeTable2));
```
  ##  15.Utilities
  ```cs
var list = db.Utilities.DataTableToList(datatable);
  ```

  ##  16.SimpleClient
```cs
var db = GetInstance();
var sdb = db.GetSimpleClient<Student>();
sdb.GetById(1);
sdb.GetList();
sdb.DeleteById(1);
sdb.Update(obj);
 ```
  

# Code generator
https://github.com/sunkaixuan/SoEasyPlatform

# More APi 中文文档：
http://www.codeisbug.com/Doc/8
