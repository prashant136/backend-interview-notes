
Profiling = provide a way to handle different configs for different environments (dev, test, prod) in spring app.

there are so many other configurations, which are different in diff envs.
- url
- port,
- connection timeout value
- retry (in prod -> retry = 3 )

application.properties parent file hota hai en sab ke liye - 
application-{profile1}.properties
application-{profile2}.properties
application-{profileN}.properties

ex - application-dev.properties, application-test.properties, application-prod.properties. 

suppose there is test profile active, 
if application-test.properties file agar nhi milta then iske parent application.properties me dekhega.
if application-test.properties file me koi properties nhi milta, then iske parent application.properties me dekhega. agar application.properties me bhi nhi hai toh error dega.


**how to set profile** - define  "spring.profiles.active=prod/test/dev"  at application.properties.
we can define multiple config also - spring.profiles.active=dev, prod

**Dynamic Configuration -**

1. while app starts ( command ) - 

```
mvn spring-boot:run -Dspring-boot.run.profiles=test // it will fire test env even if we define spring.profiles.active=prod at application.properties.
```


2. add profile in pom.xml -
```
<profiles>
	<profile>
		<id>local</id>
		<properties>
			<spring-boot.run.profiles>dev</spring-boot.run.profiles>
		</properties>
	</profile>
	<profile>
		<id>production</id>
		<properties>
			<spring-boot.run.profiles>prod</spring-boot.run.profiles>
		</properties>
	</profile>
</profiles>

... then run mvn spring-boot:run -Pproduction (pick by <id>  </id>)
```


@Profile - used to conditionally activate or register Spring beans based on the currently active application environment (such as `dev`, `test`, or `prod`). then iit inject the Bean.

let's say in some environment you want certain Bean to be injected and in other environment you don't want to inject those beans.

@Profile("prod")
@Profile("!prod") // Loads for any profile that is NOT production
@Profile("prod & cloud") // Active only if both 'prod' and 'cloud' are active
@Profile("dev | test") // Active if either 'dev' or 'test' is active



