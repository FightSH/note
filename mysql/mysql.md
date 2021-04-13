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



### SQL接口：负责处理接收到的SQL语句

​	但MySql工作线程从网络连接中读取出SQL语句后，会将其交给SQL接口

[^]: 即MySql提供的执行SQL语句的接口

去执行。

​	



​	

### 查询解析器



### 查询优化器



### 存储引擎接口



### 执行器