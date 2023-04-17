# 4 JDBC和连接池  

## 4.1 JDBC概述

JDBC是为访问不同的数据库提供了统一的接口。Java程序员使用JDBC可以连接任何提供了JDBC驱动程序的数据库系统，从而完成对数据库的各种操作。  

### 4.1.1 JDBC程序编写步骤   

 （1）注册驱动：加载Driver类     

```
Driver driver = new com.mysql.cj.jdbc.Driver();
```

 （2）获取链接：得到Connection      

```
String url = "jdbc:mysql://localhost:3306/db02";   
Properties properties = new Properties();   
properties.setProperty("user", "root");   
properties.setProperty("password", "123456");   
Connection connect = driver.connect(url, properties);
```


 （3）执行增删改查：发送SQL给mysql执行    

```
String sql = "update actor set name = '周星驰' where id = 1";    
Statement statement = connect.createStatement();    
int rows = statement.executeUpdate(sql); 
```


 （4）释放资源：关闭相关连接      

```
statement.close();   
connect.close();     
```

### 4.1.2 获取数据库连接的方式   

 （1）静态加载：    

```
 Driver driver = new com.mysql.cj.jdbc.Driver();  
```


 （2）动态加载（使用反射）：更加灵活，方便  

```
Class<?> aClass = Class.forName("com.mysql.cj.jdbc.Driver");
Driver driver = (Driver) aClass.getDeclaredConstructor().newInstance();
```

 （3）使用DriverManager替代Driver进行统一管理   

```
Class<?> aClass = Class.forName("com.mysql.cj.jdbc.Driver");
Driver driver = (Driver) aClass.getDeclaredConstructor().newInstance();    
DriverManager.registerDriver(driver);
String url = "jdbc:mysql://localhost:3306/db02";
Connection conn = DriverManager.getConnection(url,"root","123456"); 
```

 （4）使用Class.forName 自动完成注册驱动     
 tips：注册驱动在Driver进行静态加载的时候就进行了,所以无需显式注册   

```
//使用反射完成注册驱动
Class<?> aClass = Class.forName("com.mysql.cj.jdbc.Driver");   
String url = "jdbc:mysql://localhost:3306/db02";   
Connection connection = DriverManager.getConnection(url, "root", "123456");    
```


 （5）利用配置文件读取相关信息：（常用）   

```
Properties properties = new Properties();   
properties.load(new FileInputStream("src\\mysql.properties")); 
String user = properties.getProperty("user"); 
String password = properties.getProperty("password"); 
String driver = properties.getProperty("driver"); 
String url = properties.getProperty("url"); 
Class.forName(driver); 
Connection connection = DriverManager.getConnection(url, user, password); 
```

## 4.2 Resultset 结果集

（1）表示数据库结果集的数据表，通过执行**查询数据库的语句**生成
（2） ResultSet保持一个光标指向某数据行，最开始位于第一行之前（不是第一行）      
（3）next方法移动到下一行，没有下一行时返回false     
（4）previous方法移动到上一行，没有上一行时返回false   
（5）getXXX（列的索引|列名）：索引从1开始，返回对应列的值，接收类型是XXX    
（6）getObject：统一返回Object   

## 4.3 建立连接后，对数据进行访问的方式     

### 4.3.1 statement:存在SQL注入风险    

SQL注入：利用某些系统没有对用户输入的数据进行充分的检查，而在用户输入数据中注入非法的SQL语句段或命令，恶意攻击数据库        

tips：现在基本不使用statement

### 4.3.2 PreparedStatement：经过预处理的statement    

 （1）SQL语句中的参数可以用“？”表示，调用SetXXX（占位符，参数值）方法设置这些参数。占位符（表示是第几个参数）从1开始。        

```
preparedStatement.setString(1, admin_name);
```

 （2）executeQuery返回ResultSet    
 （3）executeUpdate返回受影响的行数        

## 4.4 JDBC相关API小结

（1）DriverManager : getConnection(url, user, pwd);

（2）Connection： createStatement（）

（3）Statement：

​		  executeUpdate(sql)：执行dml语句

​		  executeQuery(sql)：执行查询语句

​		  execute(sql)：执行任意语句，返回布尔值

（4）PreparedStatement：  

​		  executeUpdate(sql)：执行dml语句

​		  executeQuery(sql)：执行查询语句

​		  execute(sql)：执行任意语句，返回布尔值

​		  setXXX(占位符索引，占位符值)：解决SQL注入问题

​		  setObject(占位符索引，占位符值)

（5）ResultSet：  		  

​		  next()：下移

​		  previous()：上移

​		  getXXX(列的索引||列名)：返回对应值，接收类型是XXX

​		  getObject(列的索引||列名)：返回对应值，接收类型是Object

## 4.5 事务

（1）JDBC当一个Connection创建时，是默认自动提交事务，不能回滚。  
（2）多个SQL语句作为整体执行，需要使用事务
（3）调用Connection的setAutoCommit(false):取消自动提交
（4）最后执行成功后，调用Connection的commit（）
（5）执行中出现异常时，可以调用Connection的rollback（）方法进行回滚（回滚整个事务）。   

## 4.6 批处理

（1）需要成批处理数据时，采用批处理机制，比单独提交  
（2）addBatch（）：添加需要批量处理的SQL语句或参数
（3） executeBatch（）：批量处理语句
（4）clearBatch（）：清空批量处理包的语句
（5）JDBC连接Mysql时，要使用批处理功能，在url中加参数：

```
?rewriteBatchedStatements=true;  
```

（6） elementDate(ArrayList数组)存放预处理的sql语句，初始大小为10，满后按照1.5倍扩容。  

## 4.7 数据库连接池

（1）传统获取连接的方式，不能控制创建的连接数量，可能导致内存泄露、Mysql崩溃
（2）缓冲池：缓冲池中放入一定数量的连接，需要建立连接时，只需从缓冲池中取出一个，使用完毕后再放回去（该连接可以复用）。    
（3）当应用程序向连接池请求的连接数量超过最大连接数量时，会加入等待队列。
（4）JDBC的数据库连接池使用javax.sql.DataSource来表示，是一个接口        

### 4.7.1 使用Druid数据库连接池

```
Properties properties = new Properties(); 
//需要使用Druid配置文件
properties.load(new FileInputStream("src\\druid.properties")); 
DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);  
Connection connection = dataSource.getConnection(); 
connection.close(); 
```

## 4.8 Apache——DBUtils

现有resultSet存在的问题：    
 （1）结果集合和connection是关联的，如果关闭connection就不能使用结果集    
 （2）结果集不利于数据的管理   
 （3）使用返回信息不方便     

### 4.8.1  查询 

```
//返回多行多列：
List<Actor> list = queryRunner.query(connection, sql, new BeanListHandler<>(Actor.class), 1);
```

```
//返回一行多列  
Actor actor = queryRunner.query(connection, sql, new BeanHandler<>(Actor.class), 1); 
```

```
//返回一行一列（即某个对象）
Object object = queryRunner.query(connection, sql, new ScalarHandler(), 1);
```

tips：1是问号的值，可以传多个参数     

### 4.8.2 javaBean（即和数据库对应的java的类）

（1） javaBean和表的数据类型要对应
（2）字段名必须相同（也可以和别名相同）
（3） javaBean的类型用包装类，不要用基本数据类型   

## 4.9 Javaweb的简单框架

（1）AppView：界面层

（2）Service：业务层，调用相应的Dao完成综合的需求

（3）Dao：完成对某表的增删改查操作

（4）domain：即javaBean，与数据库的表相对应的类

（5）utils：工具类
