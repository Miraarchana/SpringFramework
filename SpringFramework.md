**Spring Framework**
-------------------------------------------------------------------------------------------------------------------------------------


**IoC - Inversion of Control : **
is a principle of software engineering in which the control of objects and portion of program is transferred to container or framework

Example:
Below does Configuration using XML file.
```java
public class GameService {
	
  //	Score score= new Score();   //does not use IOC
  
	//IoC container will call ApplicationContext (Control is delegated to Spring Container to create and maintain that bean)
	//in traditional programming, custom code makes call to library. IoC enables framework/container to control flow of program
	//and makes call to custom code.
	ApplicationContext context = new ClassPathXmlApplicationContext("spring-context.xml");
	Score score = context.getBean("thescore",Score.class);

}
```
For configuration using Annotations, the class that acts as source of bean definitions must be annotated with **@Configuration** which can be added to more than one class. **@Bean** for define a bean.
	Bean scopes:
		- Singleton - Spring checks whether a definition of bean exist in cache before creating one.
		- Prototype - A new bean instance is returned by container for each method call
		
The advantages of this architecture are:

1. decoupling the execution of a task from its implementation 
2. making it easier to switch between different implementations
3. greater modularity of a program
4. greater ease in testing a program by isolating a component or mocking its dependencies and allowing components to communicate through contracts

IoC can be acheived by Strategy DP, Service Locator pattern, Factory pattern and DI.

**DI - Dependency Injection**
Design Pattern that IoC uses to inject 
The act of connecting objects with other objects is done by assembler rather of object doing by itself
	- Constructor-Based DI - recommended for mandatory ones.
	```java
			@Configuration
			public class AppConfig {

			    @Bean
			    public Item item1() {
				return new ItemImpl1();
			    }

			    @Bean
			    public Store store() {
				return new Store(item1());
			    }
			}
			```
			```XML
			<bean id="item1" class="org.baeldung.store.ItemImpl1" /> 
			<bean id="store" class="org.baeldung.store.Store"> 
			    <constructor-arg type="ItemImpl1" index="0" name="item" ref="item1" /> 
			</bean>```
	- Setter based DI - recommended for optional ones
		``java
			@Bean
			public Store store() {
			    Store store = new Store();
			    store.setItem(item1());
			    return store;
			}
			```
		```XML
		<bean id="store" class="org.baeldung.store.Store">
		    <property name="item" ref="item1" />
		</bean>
		```
	- Field based DI
	
**Spring IoC Container**
In Spring, IoC is represented by an interface *ApplicationContext*
This container is responsible of instantiating, configuring, and assembling objects known as beans, as well as managing the life-cycle
	Implementations for *ApplicationContext*
		- ClassPathXmlApplicationContext - Standalone applications
		- FileSystemXmlApplicationContext - Standalone applications
		- WebApplicationContext - Web applications
