---
layout: default
title: Spring
parent: Primer
nav_order: 2
permalink: docs/primer/spring/
---

# Spring
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## What is Spring?

Released in [October 2002](https://en.wikipedia.org/wiki/Spring_Framework), [Spring](https://spring.io/) gained popularity due to its simplicity and practicality.  Alternative technologies, such as [Java EE](https://en.wikipedia.org/wiki/Java_Platform,_Enterprise_Edition), were quite complicated and required lots of boilerplate code.  Furthermore, these alternative technologies were hard to test as they relied on the application server.  It was not easy to test the application without spinning up an application server.

Similar to Java, the word _Spring_ is overloaded and it is used to mean different things.  This can be a bit confusing.  Spring [comprise several projects](https://spring.io/projects), such as [Spring Data](https://spring.io/projects/spring-data) or [Spring Security](https://spring.io/projects/spring-security) to name just two.  [Spring Framework](https://spring.io/projects/spring-framework) is one of the first projects that itself comprise other modules, such as [Spring MVC](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html)

## How can we use Spring?

Spring comprise several projects and each project can be used independently.  For example, we can use Spring Data independent from the other projects, including the [Spring Framework](https://spring.io/projects/spring-framework), in our application.

Spring projects can be added to an application like any other dependency.  For example, the following example adds the [Spring Data JDBC](https://spring.io/projects/spring-data-jdbc) to the dependencies.

```groovy
dependencies {
  implementation 'org.springframework.data:spring-data-jdbc:2.0.1.RELEASE'
}
```

This will allow us to use [`JdbcTemplate`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html) to interact with the database in a safe manner within our application.

## What are Spring beans?

"_In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC container are called beans. A bean is an object that is instantiated, assembled, and otherwise managed by a Spring IoC container. Otherwise, a bean is simply one of many objects in your application. Beans, and the dependencies among them, are reflected in the configuration metadata used by a container._"<br/>
([reference](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-introduction))

The _Spring IoC container_ is a module from the _Spring Framework_ and these two are used interchangeably.

Let's see Spring beans through an example.  The following image shows a web application that contains five classes.

![Web Application Layout]({{ '/assets/images/Web-Application-Layout.png' | absolute_url }})

The application comprises, two REST controllers, a service and to two repositories, connected as shown above.  The REST controllers do not talk to the repositories directly, but they do this through the service.  The repositories connect to the database to read and persist data.

### How will we create and connect these classes together?

We can initialise each class when our application starts and manage the lifecycle of each class.  We also need to connect the classes together and make sure that each class is initialised once and not multiple times.  For example, both REST controllers make use of the same service and not two separate instances.  This is quite important especially if we need to coordinate things.

Alternatively, we can make use from Spring Framework and its IoC container to initialise and handle the lifecycle of our classes.  Each of our five classes will be considered as a Spring Bean as this is initialised and managed by the Spring Framework.

In summary, a Spring Bean is an object that is managed by the Spring Framework.

## What is the difference between Spring Framework and Spring Boot?

[Phillip Webb](https://spring.io/team/pwebb), [Spring Boot](https://spring.io/projects/spring-boot) co-creator, gave a very good analogy when introducing Spring Boot.  Consider the following image.

![Spring Framework and Spring Boot Analogy]({{ '/assets/images/Spring-Framework-Boot-Analogy.png' | absolute_url }})
([Reference](https://twitter.com/phillip_webb/status/641444531867680768?lang=en))

The Spring Framework provides us with lots of great ingredients that we need to make a delicious cake (or Spring based application).  Spring Boot takes all these great ingredients and bakes the cake for us.  Spring Boot is very customisable, and we can change (_almost_) anything we like.  Furthermore, Spring Boot uses a set of ingredients (dependencies) that work well together.  This ensures that our cake (or Spring based application), does not taste funny.

The Spring Framework and Spring Boot are two popular projects from the many projects within the Spring ecosystem.  Spring Framework provides _Inversion of Control_, also referred to as _Dependency Injection_ amongst other things, while Spring Boot simplifies the usage and integration of various parts of an application.

The Spring Framework can be used together with Spring Boot or without it.  Consider the following application.

[ThoughtWorks](https://www.thoughtworks.com/) has offices in 14 countries across the globe.  Consider a simple web application that returns the list of countries where ThoughtWorks has an office.  This application contains one [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) endpoint that returns the list of countries.

```bash
$ curl http://localhost:8080/countries
```

The above request returns the list of countries in [JSON](https://www.json.org/) format, similar to the following example.

```json
[
  {"name":"Australia"},
  {"name":"Brazil"},
  {"name":"Canada"},
  {"name":"Chile"},
  {"name":"China"},
  {"name":"Ecuador"},
  {"name":"Germany"},
  {"name":"India"},
  {"name":"Italy"},
  {"name":"Singapore"},
  {"name":"Spain"},
  {"name":"Thailand"},
  {"name":"United Kingdom"},
  {"name":"United States of America"}
]
```

The [repository](https://github.com/albertattard/spring-with-and-without-boot) contains two applications, one built without Spring Boot and the other with Spring Boot.  Both application do the same thing, that is, implement one REST endpoint that lists the countries where ThoughtWorks has an office in.

Do not worry if all of this is new, as each part will be explained in more depth at a later stage.  The scope of this example is to simply show the difference between an application without Spring Boot and the equivalent application with Spring Boot.  Unfortunately we had to bring in some things that were not yet introduced.

Let's compare both applications.

### Spring Framework without Spring Boot

The solution that does not make use of Spring Boot has the following folder structure.

```bash
$ tree spring-without-boot
spring-without-boot
├── Dockerfile
├── build.gradle
└── src
    └── main
        ├── java
        │   └── demo
        │       └── noboot
        │           ├── ContactUsController.java
        │           └── Country.java
        └── webapp
            └── WEB-INF
                ├── spring-servlet.xml
                └── web.xml

7 directories, 6 files
```

Two XML files are required to configure our application.

1. The [`spring-servlet.xml` file](https://github.com/albertattard/spring-with-and-without-boot/blob/master/spring-without-boot/src/main/webapp/WEB-INF/spring-servlet.xml) configures the Spring Framework

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xmlns:mvc="http://www.springframework.org/schema/mvc"
          xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/mvc
           http://www.springframework.org/schema/mvc/spring-mvc.xsd">

     <!-- Provide support for component scanning -->
     <context:component-scan base-package="demo.noboot"/>

     <!--Provide support for conversion, formatting and validation -->
     <mvc:annotation-driven/>

   </beans>
   ```

   The above file sets the base package and enables annotation processing.

1. The [`web.xml` file](https://github.com/albertattard/spring-with-and-without-boot/blob/master/spring-without-boot/src/main/webapp/WEB-INF/web.xml) is typical to all [Servlet based applications](https://javaee.github.io/servlet-spec/)

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns="http://java.sun.com/xml/ns/javaee"
            xsi:schemaLocation="
             http://java.sun.com/xml/ns/javaee
             http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
            id="spring-without-boot"
            version="3.0">

     <display-name>Spring without Boot</display-name>

     <servlet>
       <servlet-name>spring</servlet-name>
       <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
       <load-on-startup>1</load-on-startup>
     </servlet>

     <servlet-mapping>
       <servlet-name>spring</servlet-name>
       <url-pattern>/</url-pattern>
     </servlet-mapping>
   </web-app>
   ```

   The above file set Spring's [`DispatcherServlet`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html) as the catch all servlet.

Our application does not have a `main()` method.  We cannot run this application using `java -jar`, as we usually do with a FatJAR application.  This application requires a webserver, such as [Apache Tomcat](http://tomcat.apache.org/), to run on.  The [`dockerfile`](https://github.com/albertattard/spring-with-and-without-boot/blob/master/spring-without-boot/Dockerfile), part of the application, makes use of Apache Tomcat to host our application, as shown next.

```dockerfile
FROM tomcat:jdk14-openjdk-oracle
ADD build/libs/spring-without-boot-1.0.war /usr/local/tomcat/webapps/ROOT.war
EXPOSE 8080
CMD ["catalina.sh", "run"]
```

When Apache Tomcat starts, it will automatically deploy our application and runs it.  This Apache Tomcat will only contain our application and is not shared with other applications.

### Spring Framework with Spring Boot

The solution that makes use of Spring Boot has the following folder structure.

```bash
$ tree spring-with-boot
spring-with-boot
├── build.gradle
└── src
    └── main
        └── java
            └── demo
                └── boot
                    ├── ContactUsApplication.java
                    ├── ContactUsController.java
                    └── Country.java

5 directories, 4 files
```

Note that we have no XML files, but instead we have one extra source file, [`ContactUsApplication`](https://github.com/albertattard/spring-with-and-without-boot/blob/master/spring-with-boot/src/main/java/demo/boot/ContactUsApplication.java).

```java
package demo.boot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ContactUsApplication {
  public static void main( final String[] args ) {
    SpringApplication.run( ContactUsApplication.class, args );
  }
}
```

The above class replaces both XML files.

Our application has a `main()` method, which means it can run like any other Java standalone application.

```bash
$ java -jar application.jar
```

Spring Boot took care of configuration needed and also packaged Apache Tomcat webserver as part of our application.  Our application is standalone, and it only requires the correct version of Java.

Also, note that this version of the application does not contain a `dockerfile`, as it takes advantage from the [`bootBuildImage` Gradle task](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/html/#build-image) to build the docker image, as shown next.

```bash
$ ./gradlew bootBuildImage --imageName=spring-with-boot:local
```

Spring Boot will create a docker image making best use of docker layers and cache.  This is discussed in some depth in the [demo following this page]({{ '/docs/primer/demo/#dockerize-the-application' | absolute_url }}).

### With or without Spring Boot?

Comparing the two applications mentioned in before, it is clear that Spring Boot simplifies our lives.  We do not have to worry about configuring our application.  Spring Boot does that for us.  The two examples are very trivial and make use of just the Spring Framework.

Consider an application that needs a database access, communicates to an LDAP server, and exchanges messages with a message queue.  This application will require a fair amount of configuration.  Spring Boot will prove very helpful and in most cases all that we need to provide are the details of each connection, such as credentials, in a properties file.  Spring Boot will take care of the rest.

Nowadays, it is uncommon to have Spring Framework without Spring Boot.  Most tutorials available nowadays make use of Spring Boot as this removed the need of configuration.

## How does Spring Boot works?

Any Java library, referred to in this section as _dependency_, can take advantage of Spring Boot to improve its adoption.  Take for example the [Axon Framework](https://axoniq.io/), a [CQRS framework](https://martinfowler.com/bliki/CQRS.html).  This dependency takes advantage of Spring Boot to simplify the adoption of the same dependency.

Consider the applications mentioned in the [previous section](#what-is-the-difference-between-spring-framework-and-spring-boot), when we compare an application without Spring Boot with the same version of the application that makes use of Spring Boot.

1. Without Spring Boot, we simply imported the dependency.

   ```groovy
   dependencies {
     implementation 'org.springframework:spring-webmvc:5.2.7.RELEASE'
   }
   ```

1. In the second example, when using Spring Boot, we imported the Spring-Boot version of the dependency.

   ```groovy
   dependencies {
     implementation 'org.springframework.boot:spring-boot-starter-web'
   }
   ```

Spring Boot does not know how to configure anything.  Instead, Spring Boot provides the means for each dependency to configure itself.  Consider the following image.

![Spring-Boot-Library-Components.png]({{ '/assets/images/Spring-Boot-Library-Components.png' | absolute_url }})

The dotted boxes represent two libraries provided by Spring Boot.  A dependency (shown in blue in the above image), that need to be auto configured by Spring Boot, needs to provide two additional projects (shown in green and orange in the above image).

1. **Autoconfigure** is responsible from configuring the dependency and creates the required beans
1. **Starter** connects our configuration to Spring Boot

This is how Spring Boot reduces the configuration.  It simply shifts the configuration of the dependency to the developers of the same dependency as shown next.

![Spring Boot Move Configuration]({{ '/assets/images/Spring-Boot-Move-Configuration.png' | absolute_url }})

By importing the starter project, we will take advantage of the configuration defined in the autoconfigure project and also use the dependency itself.
