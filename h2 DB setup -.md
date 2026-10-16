
open link - http://localhost:8080/h2-console

application.yml -

```yaml
spring:  
  
  datasource:  
    driver-class-name: org.h2.Driver  
    url: jdbc:h2:mem:redisDemoDB  
    username: sa  
    password: password  
  
  h2:  
    console:  
      enabled: true  
      path: /h2-console
```

use

```
Driver Class : org.h2.Driver
JDBC URL     : jdbc:h2:mem:redisDemoDB
User Name    : sa
Password     : password
```