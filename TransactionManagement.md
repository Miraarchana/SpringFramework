
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

Note: Any database transaction, by defintion must be **ACID**
1. **Atomic** - either entire transaction or nothing occurs. No partial commit
2. **Consistent** - Any change made during a transaction must adhere to database constraints like data integrity (primary key violation)
3. **Isolation** - One transaction changes must not affect other transaction
4. **Durability** - guarantee that committed transaction will survive permanently.

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

1. **SERIALIZABLE:**
    	- If two transcation executing concurrently is __*considered to be as serial execution*__ where one transaction gets committed and the next is executed. 
	- This is **total isolation**.
   	- creates low performance and deadlock might occur
2. **REPEATABLE_READ:**
    	- Till first transaction is committed, existing records __*cannnot be modified*__ by second transaction but __*new records can be added*__ by second  transaction. 
	- After second is committed, newly added records get reflected in first transaction which is still not committed
3. **READ_COMMITTED:**
    	- Before first transaction is committed, second __*can modify existing record or add new record and commit*__. 
	- These __*data will be reflected in first transaction which is not committed*__.
4. **READ_UNCOMMITTED**
	- When two concurrent transactions are executed, before the first transaction is committed, second __*can alter existing or add new records*___.
	- Even when __*second is not committed we still can see the changes reflected when first executes a select again*__.

**Dirty Reads**
  - Transaction read data that are not committed by other Transaction.
    ex., T1 and T2 runs concurrently, T1 modifies record and T2 reads records, but T1 rollbacks the changes for the record and commits. T2 will be having a wrong data.
   - read UNCOMMITED data from another transaction
**Non_repeatable Reads**
  - Performing the same query twice may return different data.
    eg., T1 and T2 runs concurrently, T1 reads record and T2 modifies records before T1 has been committed, but T1 reads the records again they will be different. 
  - read COMMITTED data from an UPDATE query from another transaction
**Phantom Reads**
  - Perform same query twice for a range may return different data.
  - read COMMITTED data from an INSERT or DELETE query from another transaction
  
___Non_Repetable read vs Phantom Read__
  - Non Repeatable is for single row
  - Phantom is for a range of records.
  
| Isolation Level  | Dirty Reads                                                                  | Non_Repetable Read                                                         | Phantom Read                                                   |
|------------------|------------------------------------------------------------------------------|----------------------------------------------------------------------------|----------------------------------------------------------------|
| Serializable     | Not possible                                                                 | Not possible                                                               | Not possible                                                   |
| Repeatable_Read  | Not possible                                                                 | Not possible                                                               | Possible As T2 can insert records even if T1 is not committed. |
| Read_Committed   | Not possible                                                                 | Possible As T2 can modify  existing records even if first is not committed | Possible As T2 can insert records even if T1 is not committed. |
| Read_Uncommitted | Reading data when T1 is not committed If T1 rollback T2 will have wrong data | Possible                                                                   | Possible                                                       |

Benefits of different isolation levels
 - READ_UNCOMMITTED prevents nothing. It's the zero isolation level
-  READ_COMMITTED prevents just one, i.e. Dirty reads
-  REPEATABLE_READ prevents two anomalies: Dirty reads and Non-repeatable reads
-  SERIALIZABLE prevents all three anomalies: Dirty reads, Non-repeatable reads and Phantom reads

Isolation Level in SpringBoot:
	- is the default level in underlying database. i.e., If underlying database is MySQL, the isolation level is REPETABLE_READ.

~~~ // Using Transactional annotation we can define any isolation level supported by the underlying database.
	@Transactional(isolation = Isolation.SERIALIZABLE)
~~~

Note:
----
[https://dzone.com/articles/spring-transaction-management] - talks about Programmatic Transaction Management and Declarative Transaction Management.
