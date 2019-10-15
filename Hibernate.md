**Hibernate**
  -ORM Tool
  -used in data layer where we have to persist application data to database
  -implements JPA(Java Persistence API) means it implements set of rules which helps in decoupling ORM tool with another without any change in application.
  
Without Hibernate
- Mapping member variables to column manually
- Mapping relationship
- Handling datatypes
- Managing changes to Object state
    for ex., An user information is retreived from database, an information like emailID is updated by application. The changes has to be manually handled to get reflected in database.
    
    
Steps involved in Using Hibernate API
1. Create a SessionFactory - for application to get hold of session objects (created once)
2. Create session from Session Factory- whenever a transaction to be performed
3. Use the session to save or other transaction with model objects

Mapping Model class and adding the this class name into SessionFactory mapping in hibernate-configurtion.xml
```java
public class HibernateTestApplication{
  UserDetails user = new UserDetails();
  user.setName("nasdm");
  user.setAge(30);
  
  SessionFactory sessFactory = new Configuration().configure().buildSessionFactory();
  Session session = sessFactory.openSession();
  session.beginTransaction();
  session.save(user);
  session.getTransaction().commit();
}
```
Note: hbm2dll.auto property by default mentioned as create- is drop-create for every startup
changing to update - drops and updates only if there is any change in schema

*[http://www.techferry.com/articles/hibernate-jpa-annotations.html]

