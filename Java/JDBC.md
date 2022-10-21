# JDBC连接配置

``` java
final String driverName = "com.microsoft.sqlserver.jdbc.SQLServerDriver";
        // 连接服务器和数据库studentcourse和建立连接
        final String dbURL = "jdbc:sqlserver://localhost:1433; DatabaseName=StudentDb";
        final String userName = "sa"; // 默认用户名
        final String userPwd = "lgj"; // 密码
        Connection Conn = null;

        try {
            Class.forName(driverName);
            Conn=DriverManager.getConnection(dbURL, userName, userPwd);
            System.out.println("连接成功");
            
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (Conn != null)
                    Conn.close();
            } // 对记录操作完毕都要关闭数据库连接
            catch (SQLException e) {
                e.printStackTrace();
            }
        }

```



## Statsment接口和PreparedStatsment接口

![image-20200517100646718](E:%5C%E7%AC%94%E8%AE%B0%5CJava%5C%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6%E5%A4%B9%5Cimage-20200517100646718.png)







### Statement

```java
 String sql="";
          Statement stmt =Conn.createStatement();
          String sql="insert into Java values('12','323','26','55') ";
          stmt.execute(sql);
```

### PreparedStatement

```java
 sql="insert into Java values(?,?,?,?)";//  ？占位符
            PreparedStatement pstmt= Conn.prepareStatement(sql);
            //加值,可以使用Object，避免字符不匹配
            pstmt.setString(1, "12");
            pstmt.setString(2, "12");
            pstmt.setString(3, "12");
            pstmt.setString(4, "12");
            
            //插入记录
            pstmt.execute();
```

### ResultSet

```java
sql="select 姓名,住址 from Java where 住址=? ";
            PreparedStatement pstmt= Conn.prepareStatement(sql);
            pstmt.setObject(1, "323");//把住址为‘323’的记录取出来
            ResultSet rs=pstmt.executeQuery();
            
            //插入记录
            pstmt.execute();
            //返回行数
            while(rs.next()) {
                System.out.println(rs.getString(1)+"----"+rs.getString(2));//
            }
           //连接成功
            12 ----323      
```

### Batch

```java
    Conn.setAutoCommit(false);//手动提交
            tmt =Conn.createStatement();
            for(int i=0;i<2000;i++) {
                tmt.addBatch("insert into Java (姓名,住址,职务) values('罗 "+i+"','33','dasd')");
            }
            tmt.executeBatch();
            Conn.commit();
            long end =System.currentTimeMillis();
            System.out.println("插入2000条数据耗时"+(end-start));
```

### 事物回滚

```java
 Conn.setAutoCommit(false);
            pstmt= Conn.prepareStatement("insert into Java1 values(?,?)");
            pstmt.setObject(1, "罗国军");
            pstmt.setObject(2,"asd");
            pstmt.execute();
            System.out.println("插入个用户，罗国军");
            
            Thread.sleep(6000);//睡6秒
            
            pstmt1= Conn.prepareStatement("insert into Java1 values(?,?,?)");//多插入一个执行时失败，不对数据表插入值
            pstmt1.setObject(1, "张三");
            pstmt1.setObject(2,"dada");
            pstmt1.execute();
            System.out.println("插入一个用户，张三");
            
            Conn.commit();
        } catch (SQLException e) {
            
            e.printStackTrace();
            try {
                Conn.rollback();//执行回滚，插入不成功时
            } catch (SQLException e1) {
                // TODO Auto-generated catch block
                e1.printStackTrace();
            }


```



## JDBC_CRM原理

### 使用Object[]和List<Object[]>来封装一条或多条数据

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

import com.zxc.JDBCUtil;
/*
* 测试使用Object[]数组来封装一条记录
* 测试使用List<Object[]>来存储多条记录
*/
public class TestObject {
public static void main(String[] args) {
    Connection  conn =null;
    ResultSet rs=null;
    PreparedStatement ps=null;
    
    List<Object[]> list= new ArrayList<Object[]>();
    conn=JDBCUtil.JDBC();//调用之前封装好的JDBC工具类
    System.out.println("连接成功");
    try {
        ps =conn.prepareStatement("select name,salary,age from emp where age>?");
        ps.setObject(1, 18);
        rs=ps.executeQuery();
        
        while(rs.next()) {
        Object []objs=new Object[3];
//            System.out.println(rs.getString(1)+"--"+rs.getInt(2)+"--"+rs.getInt(3));
        
        objs[0]=rs.getString(1);
        objs[1]=rs.getInt(2);
        objs[2]=rs.getInt(3);
        list.add(objs);//将数据添加到List里面
        }
    } catch (SQLException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
    for(Object [] objs:list) {
    System.out.println(""+objs[0]+objs[1]+objs[2]);
    }
}
}


```



### 使用Map<String,Object>和List<Map<String,Object>>来封装一条或多条数据

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.swing.RowFilter;

import com.zxc.JDBCUtil;
/*
* 测试使用Map来封装一条记录
* 测试使用List<Map>来存储多条记录
*/
public class TestMap {
public static void main(String[] args) {
    Connection  conn =null;
    ResultSet rs=null;
    PreparedStatement ps=null;
    List<Map<String,Object>> list=new ArrayList<Map<String,Object>>();
//    List<Object[]> list= new ArrayList<Object[]>();
//    Map<String,Object> map=new HashMap<String, Object>();
    conn=JDBCUtil.JDBC();
    System.out.println("连接成功");
    try {
        ps =conn.prepareStatement("select name,salary,age from emp where age>?");
        ps.setObject(1, 10);
        rs=ps.executeQuery();
        
        while(rs.next()) {
        Map<String,Object> row=new HashMap<String, Object>();
        row.put("name", rs.getString(1));//put属性
        row.put("salary", rs.getInt(2));
        row.put("age", rs.getInt(3));
        list.add(row);//将数据添加到List里面
        
        }
    } catch (SQLException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
    //遍历map
    
for(Map<String,Object> row:list) {
for(String key:row.keySet()) {
    System.out.print(key+"--"+row.get(key)+"\t");
}
    System.out.println();
}
}
}
```



### 使用Javabean和List<Emp>来封装一条或多条数据

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.swing.RowFilter;

import com.zxc.JDBCUtil;
/*
* 测试使用Javabean来封装一条记录
* 测试使用List<Javabean>来存储多条记录
*/
public class TestJavabean {
public static void main(String[] args) {
    Connection  conn =null;
    ResultSet rs=null;
    PreparedStatement ps=null;
    Emp emp=null;//在外面定义的类
    conn=JDBCUtil.JDBC();
    List<Emp> list=new ArrayList<Emp>();
    System.out.println("连接成功");
    try {
        ps =conn.prepareStatement("select name,salary,age from emp where age>?");
        ps.setObject(1, 10);
        rs=ps.executeQuery();
        
        while(rs.next()) {
        emp=new Emp(rs.getString(1), rs.getInt(2),rs.getInt(3));
        list.add(emp);//添加到list
        
        }
    } catch (SQLException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
遍历//
    for(Emp emps:list) {
    System.out.println(emps.getName()+"--"+emps.getSalary()+"--"+emps.getAge());
    }
}
}

```

