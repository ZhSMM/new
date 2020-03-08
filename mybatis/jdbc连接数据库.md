# jdbc连接数据库

## 步骤

1. 加载数据库驱动；
2. 创建并获取数据库连接；
3. 创建jdbc statement对象；
4. 设置sql语句；
5. 设置sql语句中的参数（使用preparedStatement）；
6. 通过statement执行sql并获取结果；
7. 对sql执行结果进行解析处理；
8. 释放资源(resultSet、preparedstatement、connection)。

## 缺点

1. 数据库连接，使用时就创建，不使用立即释放，对数据库进行频繁连接开启和关闭，造成数据库资源浪费，影响数据库性能。

   解决方法：使用数据库连接池管理数据库连接；

2. 将sql语句硬编码到java代码中，如果sql语句修改，需要重新编译java代码，不利于系统维护。

3. 向preparedStatement中设置参数，对占位符号位置和设置参数值，硬编码在java代码中，不利于系统维护。

4. 从resutSet中遍历结果集数据时，存在硬编码，将获取表的字段进行硬编码，不利于系统维护。

## 示例代码

```java
package top.songfang.jdbctest;

import java.sql.*;

public class JdbcTest {
    public static void main(String[] args) {
        // 连接
        Connection connection=null;
        // 预编译的Statement
        PreparedStatement preparedStatement=null;
        // 结果集
        ResultSet resultSet=null;

        try{
            // 加载驱动
            Class.forName("com.mysql.cj.jdbc.Driver");
            // 通过驱动管理类获取数据库连接
            connection= DriverManager.getConnection(
                    "jdbc:mysql://localhost:3306/test","root","1234");
            // 定义sql语句
            String sql="select * from psd where id=?";
            preparedStatement=connection.prepareStatement(sql);
            preparedStatement.setInt(1,2);
            resultSet=preparedStatement.executeQuery();
            while (resultSet.next()){
                System.out.println(resultSet.getString("id")+" "+
                        resultSet.getString("account")+" "+
                        resultSet.getString("password")+" "+
                        resultSet.getString("tip"));
            }
        }
        catch (Exception e){
            e.printStackTrace();
        }
        finally {
            if(resultSet!=null){
                try {
                    resultSet.close();
                }catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (preparedStatement!=null){
                try {
                    preparedStatement.close();
                }
                catch (SQLException e){
                    e.printStackTrace();
                }
            }
            if (connection!=null){
                try {
                    connection.close();
                }catch (SQLException e){
                    e.printStackTrace();
                }
            }

        }

    }
}

```



 