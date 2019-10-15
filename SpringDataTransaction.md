**Transaction Management in Spring**

supported by
  - JDBC module
  - ORM framework module (Hibernate, JPA, iBatis)
  
**JDBC module in Java:**
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
**JDBC module in Spring:**
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
**Using JDBCTemplate **
- set of methods provided by Spring for setting before query execution and after execution.
JDBCTemplate has no idea about the object that is to be mapped for custom type.
```java
  @Component
  public class JdbcDAOImpl{
    private DataSource datasource; // create getter and setter for autowiring
  
    private JdbcTemplate jdbcTemplate;
    
    public int getCustomerCount(){
        String sql=conn.prepareStatement("SELECT COUNT(*) from Customer");
        return jdbcTemplate.queryForInt(sql); //how to decide which method to be called - Using method given below in getCustomer()
    }
    
    public String getCustomer(String custId){
        String sql=conn.prepareStatement("SELECT custname from Customer where custId=?");
        return jdbcTemplate.queryForObject(sql,new Object[]{custId},String.class); //new object arr for arguments and datatype of return value is String
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
JDBCTemplate supports mapping result to custom object using *RowMapper* class.
But application must implement RowMapper class to use it.
------------------------------------------------------------------------------------------------------------------------

**DAO support**
- Any DAOImpl must implement a DAOSupport which will take care of Datasource and jdcTemplate
SimpleJdbcDAOSupport

```java
  public class JdbcDAOImpl implements SimpleJdbcDAOSupport{
     //member variables can be used to autowire by defining a Bean in configuration
     //<bean id="simplejdbcdaosupport" class="...JdbcDAOImpl">
     //<property name="datasource" red="datasource"></bean>
     
     public int getCustomerCount(){
        String sql=conn.prepareStatement("SELECT COUNT(*) from Customer");
        return this.jdbcTemplate.queryForInt(sql); //how to decide which method to be called - Using method given below in getCustomer()
    }
  }
```
  
**Using Hibernate with Spring**
Like JdbcTemplate - Hibernate has Session object which contains different methods for differenct return type.
Singleton Bean of SessionFactory - which enables one time creation (one object per application)  

```<bean id="sessionFactory" class="org.springframework.orm.hibernate3.annotation.AnotationSessionFactoryBean">
  <property name="datasource" ref="datasource"/>
  <property name="packagesToScan" value="<package that contains model"/>
  <property name="hibernateProperties">
    <props>
      <prop key="dialect">org.hibernate.dialect.<DBdialect></prop>
    </props>
  </property>
</bean>
```
Note: AnotationSessionFactoryBean  for Annotated model class
Annotate Model class
```java
   import javax.persistence.Entity;
   import javax.persistence.Id;
    @Entity
          public class Customer{
            @Id
            private int custId;
            .
            .
        
     }
 ```
```java
  import org.hibernate.Query;
  // stereotype to mark the behavior of this class
  @Repository 
  public class HibernateDAOImpl{
     @Autowired
     private SessionFactory sessionFactory;
     
     public void setSessionFactory(SessionFactory sessFactory){
        sessionFactory= sessFactory;
     }
     public SessionFactory getSessionFactory(){
        return sessionFactory;
     }
     public int getCustomerCount(){
        String hql="select count(*) from Customer";
        Query query = getSessionFactory().openSession().createQuery(hql);
        return( (Long)query.uniqueResult()).intValue();
     }
  }
  ```
