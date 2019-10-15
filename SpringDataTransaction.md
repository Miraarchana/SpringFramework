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

**Transaction Management**
----------------------------------------------------------------------------------------------------------------------------------------
[https://www.javainuse.com/spring/boot-transaction]
~~~What are Database Transactions?
A Database transaction is a single logical unit of work which accesses and possibly modifies the contents of a database.
Begin Transaction
  Query1
  Query2
  Query3
Commit Transaction
~~~
What are ApplicationTransaction?
An application transaction is a sequence of application actions that are considered as a single logical unit by the application

example: newBankAccount is a single logical unit service will call CustomerDetails (Persist Customer Details) & AccountDetails (Persist Account Details) services.
If AccountDetails service fails CustomerDetails must also be rollback. (by default the CustomerDetails will be committed leading to error in data layer).

**Transaction Management in SpringBoot is AOP: (Cross-cutting functionality)**
To avoid such situation **Transaction Management is implemented in SpringBoot by *@Transactional* annotation**- This will create a proxy around the logical unit to begintransaction and commit on successful completion of the logical unit instead of every service.
Note: In SpringBoot like MySql by default autocommit is turned on.

BankAccountServiceImpl.java
```java
  @Service
  public class BankAccountServiceImpl implements BankAccountService{
    @Autowired
    CustomerDetailsService customerService;
    @Autowired
    AccountDetailsService accountService;
    
    @Override
    @Transactional
    public void newBankAccount(Customer customer, AccountDetails accountDetails){
      //proxy - begin transaction
      customerService.insertCustomer(customer);
      accountService.insertAccountDetails(accountDetails);
      //proxy-commit transaction
    }
```
CustomerDetailsServiceImpl.java
```java
  @Service
  public class CustomerDetailsServiceImpl implements CustomerDetailsService{
    @Autowired
    CustomerDetailsDAO customerDAO;
    
    @Override
    public void insertCustomer(Customer customer){
       customerDAO.insertCustomer(customer);
    }
```
AccountDetailsServiceImpl.java
```java
  @Service
  public class AccountDetailsServiceImpl implements AccountDetailsService{
    @Autowired
    AccountDetailsDAO accountDAO;
    
    @Override
    public void insertAccountDetails(AccountDetails accountDetails){
      accountDAO.insertAccountDetails(accountDetails);
    }
```
**Transaction Propagation**
----------------------------
Transaction Propagation indicates whether the calling service will or will not participate in transaction and how it behaves if the callling service is already created a transaction.
In above example if CustomerDetailsService is directly called commit and rollback may not be handled.
Adding @Transactional annotation to these services individually will solve this issue.

**Transaction Propagation talks about whether a service called from another service executes its own transaction or transcation of calling service.**
In the example:
CustomerDetailsService can create its own transaction to execute or it can participate in the parent service BankAccountDetails service transaction.
CustomerDetailsServiceImpl.java
```java
  @Service
  public class CustomerDetailsServiceImpl implements CustomerDetailsService{
    @Autowired
    CustomerDetailsDAO customerDAO;
    
    @Override
    @Transactional
    public void insertCustomer(Customer customer){
       customerDAO.insertCustomer(customer);
    }
```
AccountDetailsServiceImpl.java
```java
  @Service
  public class AccountDetailsServiceImpl implements AccountDetailsService{
    @Autowired
    AccountDetailsDAO accountDAO;
    
    @Override
    @Transactional
    public void insertAccountDetails(AccountDetails accountDetails){
      accountDAO.insertAccountDetails(accountDetails);
    }
``` 
**Transaction Propagation Scenarios:**
----------
**REQUIRED** 
  - Default
    example: 
    case1: If the CustomerDetailsService called directly - creates new transaction
    case2: If the CustomerDetailsService called from BankAccountService.
      1. If calling service has transaction makes use of it (in eg., BankAccountService has a transaction)
      2. Else it creates its own
    ```java
      @Override
      @Transactional(propagation=Propagation.REQUIRED)
      public void insertCustomer(Customer customer){
         customerDAO.insertCustomer(customer);
      }
    ```
    **SUPPORT** 
    example: 
    case1: If the CustomerDetailsService called directly - DOES NOT CREATE
    case2: If the CustomerDetailsService called from BankAccountService.
      1. If calling service has transaction makes use of it (in eg., BankAccountService has a transaction)
      2. Else it DOES NOT CREATE
    ```java
      @Override
      @Transactional(propagation=Propagation.SUPPORT)
      public void insertCustomer(Customer customer){
         customerDAO.insertCustomer(customer);
      }
    ```
    **REQUIRES_NEW** 
    example: 
    case1: If the CustomerDetailsService called directly - CREATES ITS OWN TRANSACTION
    case2: If the CustomerDetailsService called from BankAccountService.
      1. If calling service has transaction does not make use of it (in eg., BankAccountService has a transaction) and creates one.
      2. Else it CREATEs one.
    ```java
      @Override
      @Transactional(propagation=Propagation.REQUIRED_NEW)
      public void insertCustomer(Customer customer){
         customerDAO.insertCustomer(customer);
      }
    ```
    **NOT_SUPPORTED** 
   example: 
    case1: If the CustomerDetailsService called directly - DOES NOT CREATE
    case2: If the CustomerDetailsService called from BankAccountService.
      1. If calling service has transaction does not make use of it (in eg., BankAccountService has a transaction). Runs without  transaction.
      2. Else it DOES NOT CREATE
    ```java
      @Override
      @Transactional(propagation=Propagation.NOT_SUPPORTED)
      public void insertCustomer(Customer customer){
         customerDAO.insertCustomer(customer);
      }
    ```
    **NEVER** 
example: 
    case1: If the CustomerDetailsService called directly - DOES NOT CREATE
    case2: If the CustomerDetailsService called from BankAccountService.
      1. If calling service has transaction makes use of it (in eg., BankAccountService has a transaction). 
      2. Else it DOES NOT CREATE - RUN WITHOUT TRANSACTION.
    ```java
      @Override
      @Transactional(propagation=Propagation.NEVER)
      public void insertCustomer(Customer customer){
         customerDAO.insertCustomer(customer);
      }
    ```
   **MANDATORY**
example: 
    case1: If the CustomerDetailsService called directly - THROW AN EXCEPTION
    case2: If the CustomerDetailsService called from BankAccountService.
      1. If calling service has transaction does not make use of it (in eg., BankAccountService has a transaction). THROWS AN EXCEPTION
      2. Else if calling service does not have a transaction - THROWS AN EXCEPTION
    ```java
      @Override
      @Transactional(propagation=Propagation.MANDATORY)
      public void insertCustomer(Customer customer){
         customerDAO.insertCustomer(customer);
      }
    ```

**Transcational RollBack**
------------------------
    In case of checked exception the previous operations done by a service cannot be stopped from saving even when Transcation is yet to be committed.
  @Transactional(rollbackFor=InvalidAccountTypeException.class) must be specified in parent service class    

**Transaction Isolation**
-----------------------
It is a database state when two concurrent transaction act on the same database entity.
Isolation levels :ACID(Atomicity, Concurrency, Isolation, Durability)

Transaction Isolation at SQL level:

~~
    //Show existing transaction isolation level if mysql version >= 8
    SELECT @@TRANSACTION_ISOLATION;

     //Show existing transaction isolation level if mysql version < 8
    SELECT @@TX_ISOLATION;

    //Set transaction isolation level to serializable. Using same syntax 
    //we can set it to other isolation level.
    SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;

    //By default auto commit is enabled for mysql transaction. So we will disable it.
    SET AUTOCOMMIT=0;

    //Start transaction
    BEGIN

    //Commit transaction
    COMMIT
~~
**SERIALIZABLE:**
If two transcation executing concurrently is considered to be as serial execution where one transaction gets committed and the next is executed. This is **total isolation**
low performance and deadlock might occur
**REPEATABLE_READ:**
Till first transaction is committed, existing records cannnot be modified by second transaction but new records can be added by second  transaction. After second is committed, newl added records get reflected in first transaction which is still not committed
**READ_COMMITTED:**
Before first transaction is committed, second can modify existing record or add new record and commit. These data will be reflectd in first transaction which is not committed.

Dirty Reads 
T1 and T2 runs concurrently, T1 modifies record and T2 reads records, but T1 rollbacks the changes for the record and commits. T2 will be having a wrong data.
Non_repeatable Reads
T1 and T2 runs concurrently, T1 reads record and T2 modifies records before T1 has been committed, but T1 reads the records again they will be different. 
Phantom Reads

| Isolation Level  | Dirty Reads                                                                  | Non_Repetable Read                                                         | Phantom Read                                                   |
|------------------|------------------------------------------------------------------------------|----------------------------------------------------------------------------|----------------------------------------------------------------|
| Serializable     | Not possible                                                                 | Not possible                                                               | Not possible                                                   |
| Repeatable_Read  | Not possible                                                                 | Not possible                                                               | Possible As T2 can insert records even if T1 is not committed. |
| Read_Committed   | Not possible                                                                 | Possible As T2 can modify  existing records even if first is not committed | Possible As T2 can insert records even if T1 is not committed. |
| Read_Uncommitted | Reading data when T1 is not committed If T1 rollback T2 will have wrong data | Possible                                                                   | Possible                                                       |


~~~ // Using Transactional annotation we can define any isolation level supported by the underlying database.
	@Transactional(isolation = Isolation.SERIALIZABLE)
~~~

Note:
----
[https://dzone.com/articles/spring-transaction-management] - talks about Programmatic Transaction Management and Declarative Transaction Management.
