# JavaDBCP
*.Properties文件读取配置；封装获取连接方法；创建数据库连接池
//新增一个类：1.读取*.Properties文件；
//2.创建一个获取数据连接池方法；3.创建一个连接池归还方法（即连接关闭方法）；
package dbcpJdbcTest;

import java.io.IOException;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

import org.apache.commons.dbcp.BasicDataSource;

import com.application.jdbc.applicationjdbc.ApplicationJdbc;

public class DBCPTest {
	static String driver;
	static String url;
	static String username;
	static String password;
	static int initialSize;
	static int maxActive;
	
	static {      //使用静态块方法将配置读取并加载使用
		try {
			Properties p = new Properties();
			InputStream in = ApplicationJdbc.class.getClassLoader().getResourceAsStream("db.properties");//db.properties文件需要在src/main/resource文件夹下放置
			p.load(in);
			driver=p.getProperty("jdbc.driver");
			url=p.getProperty("jdbc.url");
			username=p.getProperty("jdbc.username");
			password=p.getProperty("jdbc.password");
			initialSize=Integer.parseInt(p.getProperty("dbcp.initialsize"));
			maxActive=Integer.parseInt(p.getProperty("dbcp.maxactive"));
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	
	public static Connection getConnection() {    //获取连接池方法
		Connection conn = null;
		try {
			BasicDataSource bs = new BasicDataSource();
			bs.setDriverClassName(driver);
			bs.setUrl(url);
			bs.setUsername(username);
			bs.setPassword(password);
			bs.setInitialSize(initialSize);
			bs.setMaxActive(maxActive);
			conn = bs.getConnection();
			return conn;
		}catch(Exception e) {
			e.printStackTrace();
			throw new RuntimeException(e);
		}
	}
	public static void close() {      //归还连接到连接池方法
		Connection conn = DBCPTest.getConnection();
		try {
			conn.close();
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
}


//新建一个类用于测试上述方法类

package dbcpJdbcTest;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class dbcpJdbcTest {
	public static void main(String[] args) {
		Connection conn = DBCPTest.getConnection();
		try {
			Statement st = conn.createStatement();
			String sql_sel = "select * from user_name";
			ResultSet rs = st.executeQuery(sql_sel);
			while(rs.next()) {
				int id = rs.getInt("user_id");
				String name = rs.getString("user_name");
				String password = rs.getString("user_password");
				System.out.println("第"+id+"项用户名为："+name+"密码是"+password);
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}finally {
			DBCPTest.close();
		}
	}
}
//第*项用户名为：****密码是****  有数据显示测试成功！
