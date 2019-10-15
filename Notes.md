**Spring Framework**
-------------------------------------------------------------------------------------------------------------------------------------


IoC - Inversion of Control : is a principle of software engineering in which the control of objects and portion of program is transferred to container or framework

Example:
```
public class GameService {
	
  //	Score score= new Score();   //does not use IOC
  
	//IoC container will call ApplicationContext (Control is delegated to Spring Container to create and maintain that bean)
	//in traditional programming, custom code makes call to library. IoC enables framework/container to control flow of program
	//and makes call to custom code.
	ApplicationContext context = new ClassPathXmlApplicationContext("spring-context.xml");
	Score score = context.getBean("thescore",Score.class);

}
```
