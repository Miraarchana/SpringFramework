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
```java
  @Component
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
