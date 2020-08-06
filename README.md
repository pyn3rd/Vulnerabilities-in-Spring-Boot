# Vulnerabilities in Spring Boot

### 0x01 Spring Boot Actuator

Actuator endpoints allow you to monitor and interact with your Spring application. Spring Boot includes a number of built-in endpoints and you can also add your own. For example the health endpoint provides basic application health information. The following endpoints are available: 
	•	/autoconfig - Displays an auto-configuration report showing all auto-configuration candidates and the reason why they 'were' or 'were not' applied.
	•	/beans - Displays a complete list of all the Spring beans in your application.
	•	/configprops - Displays a collated list of all @ConfigurationProperties.
	•	/dump - Performs a thread dump.
	•	/env - Exposes properties from Spring's ConfigurableEnvironment.
	•	/health - Shows application health information (a simple 'status' when accessed over an unauthenticated connection or full message details when authenticated).
	•	/info - Displays arbitrary application info.
	•	/metrics - Shows 'metrics' information for the current application.
	•	/mappings - Displays a collated list of all @RequestMapping paths.
	•	/shutdown - Allows the application to be gracefully shutdown (not enabled by default).
	•	/trace - Displays trace information (by default the last few HTTP requests).
  
  
### 0x02 Spring Boot Jolokia RCE


### 0x03 Spring Boot H2 Database RCE involving JNDI injection


pom.xml

``
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>

<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
    <version>1.4.199</version>
</dependency>
``


### 0x03 Spring Boot H2 Database RCE involving JNDI injection
