**Transaction Management in Spring**

supported by
  - JDBC module
  - ORM framework module (Hibernate, JPA, iBatis)
  
JDBC module in Java:
```java
  public class JdbcDAOImpl{
    public Customer getCustomer(String custID){
      Connection conn = null;
      try{
          String driver = "org.apache.derby.jdbc.ClientDriver";
          Class.forName(driver).newInstance();
          conn = DriverManager.getConnection("jdbc:derby....localhost..");
          PreparedStatement pstmt=conn.prepareStatement("SELECT * from Customer where id =?");
          pstmt.setInt(1,custID);
          ResultSet rs= pstmt.executeQuery();
          if(rs.next()){
            Customer cust = new Customer(custID,rs.getString("name"));
          }
          rs.close();
          pstmt.close();
          return cust;
        }
        catch(Exception e){
          throw new RuntimeException(e);
        }
        finally{
          try{
            conn.close();
          }catch(SQLException e){}
        }
      }
  }
```
JDBC module in Spring:
  - advantage DataSource is created common to all implementation and we can easily replace another data source implementation class in configuration without modifying the code.
  
```java
  @Component
  public class JdbcDAOImpl{
    @Autowired
    private DataSource datasource; // create getter and setter for autowiring
    
    public Customer getCustomer(String custID){
      Connection conn = null;
      try{
          //connecting to DB
          //create datasource bean *DriverManagerDataSource* in configuration file and driverclass and url is specified in it.
          //getConnection
          conn = datasource.getConnection("jdbc:derby....localhost..");
          //Preparing the SQL statement
          PreparedStatement pstmt=conn.prepareStatement("SELECT * from Customer where id =?");
          pstmt.setInt(1,custID);
          //Exceuting the statement
          ResultSet rs= pstmt.executeQuery();
          //parsing the result
          if(rs.next()){
            Customer cust = new Customer(custID,rs.getString("name"));
          }
          //closing
          rs.close();
          pstmt.close();
          return cust;
        }
        catch(Exception e){
          throw new RuntimeException(e);
        }
        finally{
          try{
            conn.close();
          }catch(SQLException e){}
        }
      }
  }
```
Using JDBCTemplate - set of methods provided by Spring for setting before query execution and after execution.

```java
  @Component
  public class JdbcDAOImpl{
    private DataSource datasource; // create getter and setter for autowiring
  
    private JdbcTemplate jdbcTemplate = new JdbcTemplate();
    
    public int getCustomerCount(String custID){
        String sql=conn.prepareStatement("SELECT COUNT(*) from Customer");
        return jdbcTemplate.queryForInt(sql); //how to decide which method to be called - Using 
    }
    public DataSource getDatasource(){
        return datasource;
    }
    
    @Autowired
    public void setDataSource(DataSource datasource){//datasource setter is used to pass datasource to JDBCTemplate
        this.jdbcTemplate = new JdbcTemplate(datasource);
    }
    
  }
```
