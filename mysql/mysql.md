# MySql架构

## 连接MySql大概过程

![image-20210413151039244](img/image-20210413151039244.png)

> `JDBC`流程如下
>
> ~~~java
>  public static void main(String[] args) throws SQLException, ClassNotFoundException {
>         Class.forName("com.mysql.cj.jdbc.Driver");
>         Connection conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/imooc", "root", "root");
>         Statement statement = conn.createStatement();
>         statement = statement;
>         String sql = "insert into user(loginName,userName,password,sex)values('tom123','tom','123456',1)";
>         int result = statement.executeUpdate(sql);
>   }
> ~~~



网络连接由线程处理：当数据库服务器连接池接受到网络请求时，会分配一个工作线程去进行处理

## MySql处理流程

![image-20210414111703139](img/image-20210414111703139.png)





### SQL接口：负责处理接收到的SQL语句

​	MySql工作线程从网络连接中读取出SQL语句后，会将其交给SQL接口去执行。

​	**SQL接口**：`是一套执行SQL语句的接口，专门用于执行我们发送给MySql的增删改查语句`

### 查询解析器：让MySql看懂sql语句

​	**Parser(查询解析器)**：`按照既定的sql语法规则，对sql语句进行解析，理解sql语句要做什么事情`

### 查询优化器：选择最优查询路径

​	**Optimizer(查询优化器)**：`针对编写的sql，生成所谓查询路径树，从中选择一条最优路径出来`

​	相当于告诉你，你应该按照什么样的步骤和顺序，去执行那些操作，然后一步一步的把sql语句给完成

### 执行器：根据执行计划调用存储引擎接口

​	`执行器会根据优化器生成的一套执行计划，然后不停的调用存储引擎的各种接口去完成sql语句的执行计划`

### 存储引擎接口：真正执行sql语句

​	MySql的架构设计中，SQL接口，解析器，优化器都是通用的，仅是一套组件而已。但存储引擎是由各种各样的，如InnoDB，MyISAM等，我们是可以选择使用那种存储引擎来负责具体sql执行的。

​	`存储引擎就是执行sql语句的，他会按照一定步骤去查询内存缓存数据，更新磁盘数据，查询磁盘数据等等`





# InnoDB

​	以一条update语句为例，初步了解InnoDB存储引擎的架构设计

​	update user set name = 'xxx' where id = 10	这条sql首先会通过数据库连接发送到MySql上，然后经过SQL接口，解析器，优化器，执行器几个环节，解析SQL语句，生成执行计划，接着由执行器负责计划的执行，调用InnoDB存储引擎的接口去执行

## 更新语句大致流程

---



### 	缓存池（Buffer Pool）

​		引擎执行更新语句时，会先判断id=10的数据是否存在于缓冲池中。如果不存在，则会从磁盘中加载数据，并且会对这行记录加独占锁

> Buffer Pool是一个放在内存中的组件，非常重要。其中放有缓存的数据，便于以后查询时，可以考虑直接从缓冲中获取。

### 	undo日志

​		如果id为10的数据name为zhangsan，先将其更新为xxx。那需要把这条记录原来的值（zhangsan，id为10）这些信息，写入到undo日志文件中去

> 考虑到未来可能要回滚数据，需要将更新前的值写入到undo日志文件中