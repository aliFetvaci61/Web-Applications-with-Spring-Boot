# Introduction to Spring Boot

The Spring framework is a very popular and widely used Java framework for building web and enterprise applications. Spring at its core is a dependency injection container that provides flexibility to configure beans in multiple ways, such as XML, Annotations, and JavaConfig. Over the years, the Spring framework grew exponentially by addressing the needs of modern business applications like security, support for NoSQL datastores, handling big data, batch processing, integration with other systems, etc. Spring, along with its sub-projects, became a viable platform for building enterprise applications.

The Spring framework is very flexible and provides multiple ways of configuring the application components. With a rich set of features combined with multiple configuration options, configuring Spring applications become complex and error-prone. The Spring team created Spring Boot to address the complexity of configuration through its powerful AutoConfiguration mechanism.

This chapter takes a quick look at the Spring framework. You’ll develop a web application using SpringMVC and JPA the traditional way (without using Spring Boot). Then you will look at the pain points of the traditional way and see how to develop the same application using Spring Boot.

# Overview of the Spring Framework
If you are a Java developer, then there is a good chance that you have heard about the Spring framework and have used it in your projects. The Spring framework was created primarily as a dependency injection container, but it is much more than that. Spring is very popular for several reasons:

Spring’s dependency injection approach encourages writing testable code

Easy-to-use and powerful database transaction management capabilities

Spring simplifies integration with other Java frameworks, like the JPA/Hibernate ORM and Struts/JSF web frameworks

State-of-the-art Web MVC framework for building web applications

Along with the Spring framework, there are many other Spring sub-projects that help build applications that address modern business needs:

Spring Data: Simplifies data access from relational and NoSQL datastores.

Spring Batch: Provides a powerful batch-processing framework.

Spring Security: Robust security framework to secure applications.

Spring Social: Supports integration with social networking sites like Facebook, Twitter, LinkedIn, GitHub, etc.

Spring Integration: An implementation of enterprise integration patterns to facilitate integration with other enterprise applications using lightweight messaging and declarative adapters.

There are many other interesting projects addressing various other modern application development needs. For more information, take a look at http://spring.io/projects .

Spring Configuration Styles
Spring initially provided an XML-based approach for configuring beans. Later Spring introduced XML-based DSLs, Annotations, and JavaConfig-based approaches for configuring beans. Listings 1-1 through 1-3 show how each of those configuration styles looks.

Listing 1-1. Example of XML-Based Configuration
```java
<bean id="userService" class="com.apress.myapp.service.UserService">
    <property name="userDao" ref="userDao"/>
</bean>

<bean id="userDao" class="com.apress.myapp.dao.JdbcUserDao">
    <property name="dataSource" ref="dataSource"/>
</bean>

<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/test"/>
    <property name="username" value="root"/>
    <property name="password" value="secret"/>
</bean>

<!-- DSL based configuration  -->
<beans>
    <jee:jndi-lookup id="entityManagerFactory" jndi-name="persistence/defaultPU"/>
</beans>

```
Listing 1-2. Example of Annotation-Based Configuration
```java
@Service public class UserService
{
    private UserDao userDao;
    @Autowired    public UserService(UserDao dao){
        this.userDao = dao;
    }
    ...
    ...
}
@Repository public class JdbcUserDao
{
    private DataSource dataSource;
    @Autowired
    public JdbcUserDao(DataSource dataSource){
        this.dataSource = dataSource;
    }
    ...
    ...
}
```
Listing 1-3. Example of a JavaConfig-Based Configuration
```java
@Configuration
public class AppConfig
{
    @Bean
    public UserService userService(UserDao dao){
        return new UserService(dao);
    }
    @Bean
    public UserDao userDao(DataSource dataSource){
        return new JdbcUserDao(dataSource);
    }
    @Bean
    public DataSource dataSource(){
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setUsername("root");
        dataSource.setPassword("secret");
        return dataSource;
    }
}
```

As you can see, Spring provides multiple approaches for configuring application components and you can even mix the approaches as well. For example, you can use JavaConfig- and Annotation-based configuration styles in the same application. That is a lot of flexibility, which is good and bad. People who are new to the Spring framework may get confused about which approach to follow.

As of now, the Spring community is suggesting you follow the JavaConfig-based approach, as it gives you more flexibility. But there is no one-size-fits-all kind of solution. You have to choose the approach based on your own application needs.

Now that you’ve had a glimpse of how various styles of Spring Bean configurations look, you’ll take a quick look at the configuration of a typical SpringMVC and JPA/Hibernate-based web application configuration.


# Developing Web Application Using SpringMVC and JPA
Before getting to know Spring Boot and learning what kind of features it provides, we’ll take a look at how a typical Spring web application configuration looks and learn about the pain points. Then, we will see how Spring Boot addresses those problems.

The first thing to do is create a Maven project and configure all the dependencies required in the pom.xml file, as shown in Listing 1-4.

Listing 1-4. The pom.xml File
```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                        http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.apress</groupId>
    <artifactId>springmvc-jpa-demo</artifactId>
    <packaging>war</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>springmvc-jpa-demo</name>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <failOnMissingWebXml>false</failOnMissingWebXml>
    </properties>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.2</version>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.3.7.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-jpa</artifactId>
            <version>1.11.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>1.7.22</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.22</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.22</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>1.4.193</version>
        </dependency>
        <dependency>
            <groupId>commons-dbcp</groupId>
            <artifactId>commons-dbcp</artifactId>
            <version>1.4</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.38</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.2.5.Final</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.thymeleaf</groupId>
            <artifactId>thymeleaf-spring4</artifactId>
            <version>2.1.4.RELEASE</version>
        </dependency>
    </dependencies>
</project>
```

We have configured all the Spring MVC, Spring Data JPA, JPA/Hibernate, Thymeleaf, and Log4j dependencies in the Maven pom.xml file.

Configure the service/DAO layer beans using JavaConfig , as shown in Listing 1-5.

Listing 1-5. The com.apress.demo.config.AppConfig.java File
```java
package com.apress.demo.config;

import java.util.Properties;

import javax.persistence.EntityManagerFactory;
import javax.sql.DataSource;                                                                       

import org.apache.commons.dbcp.BasicDataSource;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
import org.springframework.core.env.Environment;
import org.springframework.core.io.ClassPathResource;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.instrument.classloading.InstrumentationLoadTimeWeaver;
import org.springframework.jdbc.datasource.init.DataSourceInitializer;
import org.springframework.jdbc.datasource.init.ResourceDatabasePopulator;
import org.springframework.orm.hibernate4.HibernateExceptionTranslator;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(basePackages="com.apress.demo.repositories")
@PropertySource(value = { "classpath:application.properties" })
public class AppConfig
{

    @Autowired
    private Environment env;

    @Bean
    public static PropertySourcesPlaceholderConfigurer placeHolderConfigurer()
    {
        return new PropertySourcesPlaceholderConfigurer();
    }

    @Bean
    public PlatformTransactionManager transactionManager()
    {
        EntityManagerFactory factory = entityManagerFactory().getObject();
        return new JpaTransactionManager(factory);
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory()
    {
        LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();

        HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
        vendorAdapter.setShowSql(Boolean.TRUE);

        factory.setDataSource(dataSource());
        factory.setJpaVendorAdapter(vendorAdapter);
        factory.setPackagesToScan(env.getProperty("packages-to-scan"));

        Properties jpaProperties = new Properties();
        jpaProperties.put("hibernate.hbm2ddl.auto", env.getProperty("hibernate.hbm2ddl.auto"));
        factory.setJpaProperties(jpaProperties);

        factory.afterPropertiesSet();
        factory.setLoadTimeWeaver(new InstrumentationLoadTimeWeaver());
        return factory;
    }                                                                      

    @Bean
    public HibernateExceptionTranslator hibernateExceptionTranslator()
    {
        return new HibernateExceptionTranslator();
    }

    @Bean
    public DataSource dataSource()
    {
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName(env.getProperty("jdbc.driverClassName"));
        dataSource.setUrl(env.getProperty("jdbc.url"));
        dataSource.setUsername(env.getProperty("jdbc.username"));
        dataSource.setPassword(env.getProperty("jdbc.password"));
        return dataSource;
    }

    @Bean
    public DataSourceInitializer dataSourceInitializer(DataSource dataSource)
    {
        DataSourceInitializer dataSourceInitializer = new DataSourceInitializer();
        dataSourceInitializer.setDataSource(dataSource);
        ResourceDatabasePopulator databasePopulator = new ResourceDatabasePopulator();
        databasePopulator.addScript(new ClassPathResource(env.getProperty("init-scripts")));
        dataSourceInitializer.setDatabasePopulator(databasePopulator);
        dataSourceInitializer.setEnabled(Boolean.parseBoolean(env.getProperty("init-db", "false")));
        return dataSourceInitializer;
    }
}
```

In the AppConfig.java configuration class , we did the following:

Marked it as a Spring Configuration class using the @Configuration annotation.

Enabled Annotation-based transaction management using @EnableTransactionManagement.

Configured @EnableJpaRepositories to indicate where to look for Spring Data JPA repositories.

Configured the PropertyPlaceHolder bean using the @PropertySource annotation and PropertySourcesPlaceholderConfigurer bean definition, which loads properties from the application.properties file.

Defined beans for DataSource, JPA EntityManagerFactory, and JpaTransactionManager.

Configured the DataSourceInitializer bean to initialize the database by executing the data.sql script on application start-up.

Now configure the property placeholder values in application.properties, as shown in Listing 1-6.

Listing 1-6. The src/main/resources/application.properties File
```java
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/test
jdbc.username=root
jdbc.password=admin
init-db=true
init-scripts=data.sql
hibernate.dialect=org.hibernate.dialect.MySQLDialect
hibernate.show_sql=true
hibernate.hbm2ddl.auto=update
packages-to-scan=com.apress.demo
```
Create a simple SQL script called data.sql to populate sample data into the USER table, as shown in Listing 1-7.

Listing 1-7. The src/main/resources/data.sql File
```java
delete from user;
insert into user(id, name) values(1,'John');
insert into user(id, name) values(2,'Smith');
insert into user(id, name) values(3,'Siva');
Create the log4j.properties file with a basic configuration, as shown in Listing 1-8.
```
Listing 1-8. The src/main/resources/log4j.properties File

```java
log4j.rootCategory=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p %t %c{2}:%L - %m%n
log4j.category.com.apress=DEBUG
log4j.category.org.springframework=INFO
```
Now configure the Spring MVC web layer beans such as ThymeleafViewResolver, static ResourceHandlers, and MessageSource for i18n, as shown in Listing 1-9.

Listing 1-9. The com.apress.demo.config.WebMvcConfig.java File

```java
package com.apress.demo.config;

import org.springframework.context.MessageSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.support.ReloadableResourceBundleMessageSource;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ResourceHandlerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.thymeleaf.spring4.SpringTemplateEngine;
import org.thymeleaf.spring4.view.ThymeleafViewResolver;
import org.thymeleaf.templateresolver.ServletContextTemplateResolver;
import org.thymeleaf.templateresolver.TemplateResolver;

@Configuration
@ComponentScan(basePackages = { "com.apress.demo.web"})
@EnableWebMvc
public class WebMvcConfig extends WebMvcConfigurerAdapter
{

    @Bean
    public TemplateResolver templateResolver() {
        TemplateResolver templateResolver = new ServletContextTemplateResolver();
        templateResolver.setPrefix("/WEB-INF/views/");
        templateResolver.setSuffix(".html");
        templateResolver.setTemplateMode("HTML5");
        templateResolver.setCacheable(false);
        return templateResolver;
    }

    @Bean
    public SpringTemplateEngine templateEngine() {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver());
        return templateEngine;
    }

    @Bean
    public ThymeleafViewResolver viewResolver() {
        ThymeleafViewResolver thymeleafViewResolver = new ThymeleafViewResolver();
        thymeleafViewResolver.setTemplateEngine(templateEngine());
        thymeleafViewResolver.setCharacterEncoding("UTF-8");
        return thymeleafViewResolver;
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry)
    {
        registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");
    }                                                                                                      

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer)
    {
        configurer.enable();
    }

    @Bean(name = "messageSource")
    public MessageSource messageSource()
    {
        ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
        messageSource.setBasename("classpath:messages");
        messageSource.setCacheSeconds(5);
        messageSource.setDefaultEncoding("UTF-8");
        return messageSource;
    }
}

```
In the WebMvcConfig.java configuration class , we did the following:

Marked it as a Spring Configuration class using @Configuration annotation.

Enabled Annotation-based Spring MVC configuration using @EnableWebMvc.

Configured ThymeleafViewResolver by registering the TemplateResolver, SpringTemplateEngine, and ThymeleafViewResolver beans.

Registered the ResourceHandlers bean to indicate requests for static resources. The URI /resources/** will be served from the /resources/ directory.

Configured MessageSource bean to load i18n messages from ResourceBundle messages_{country-code}.properties from the classpath.

Create the messages. properties file in the src/main/resources folder and add the following property:
```java
app.title=SpringMVC JPA Demo (Without SpringBoot)
```
Next, you are going to register the Spring MVC FrontController servlet DispatcherServlet.

Note Prior to Servlet 3.x specification, you have to register servlets/filters in web.xml. Since the Servlet 3.x specification, you can register servlets/filters programmatically using ServletContainerInitializer. Spring MVC provides a convenient class called AbstractAnnotationConfigDispatcherServletInitializer to register DispatcherServlet.

Listing 1-10. The com.apress.demo.config.SpringWebAppInitializer.java File
```java
package com.apress.demo.config;

import javax.servlet.Filter;

import org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter;
import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class SpringWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer
{
    @Override
    protected Class<?>[] getRootConfigClasses()
    {
        return new Class<?>[] { AppConfig.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses()
    {
        return new Class<?>[] { WebMvcConfig.class };
    }

    @Override
    protected String[] getServletMappings()
    {
        return new String[] { "/" };
    }

    @Override
    protected Filter[] getServletFilters()
    {
       return new Filter[]{ new OpenEntityManagerInViewFilter() };
    }
}
```
In the SpringWebAppInitializer.java configuration class , we did the following:

Configured AppConfig.class as RootConfirationClasses, which will become the parent ApplicationContext that contains bean definitions shared by all child (DispatcherServlet) contexts.

Configured WebMvcConfig.class as ServletConfigClasses, which is the child ApplicationContext that contains WebMvc bean definitions.

Configured / as ServletMapping, which means that all the requests will be handled by DispatcherServlet.

Registered OpenEntityManagerInViewFilter as a servlet filter so that we can lazy-load the JPA entity lazy collections while rendering the view.

Create a JPA entity user and its Spring Data JPA Repository interface UserRepository. Create a JPA entity called User.java, as shown in Listing 1-11, and a Spring Data JPA repository called UserRepository.java, as shown in Listing 1-12.

Listing 1-11. The com.apress.demo.domain.User.java File
```java
package com.apress.demo.domain;

import javax.persistence.*;

@Entity
public class User
{
    @Id @GeneratedValue(strategy=GenerationType.AUTO)
    private Integer id;
    private String name;

    public User()
    {
    }

    public User(Integer id, String name)
    {
        this.id = id;
        this.name = name;
    }

    public Integer getId()
    {
        return id;
    }

    public void setId(Integer id)
    {
        this.id = id;
    }

    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {                                                                                                                                                                                                              
        this.name = name;
    }
}
```
Listing 1-12. The com.apress.demo.repositories.UserRepository.java File
```java
package com.apress.demo.repositories;

import org.springframework.data.jpa.repository.JpaRepository;
import com.apress.demo.domain.User;

public interface UserRepository extends JpaRepository<User, Integer>
{

}
```
If you don’t understand what JpaRepository is, don’t worry. You will learn more about Spring Data JPA in future chapters.

Create a SpringMVC controller to handle URL /, which renders a list of users. See Listing 1-13.

Listing 1-13. The com.apress.demo.web.controllers.HomeController.java File
```java
package com.apress.demo.web.controllers;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

import com.apress.demo.repositories.UserRepository;

@Controller
public class HomeController
{
    @Autowired
    private UserRepository userRepo;

    @RequestMapping("/")
    public String home(Model model)
    {
        model.addAttribute("users", userRepo.findAll());
        return "index";
    }
}

```
Create a Thymeleaf view called index.html in the /WEB-INF/views/ folder to render a list of users, as shown in Listing 1-14.

Listing 1-14. The src/main/webapp/WEB-INF/views/index.html File
```js
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="utf-8"/>
        <title>Home</title>
    </head>
    <body>
        <h2 th:text="#{app.title}">App Title</h2>
        <table>
            <thead>
                <tr>
                    <th>Id</th>
                    <th>Name</th>
                </tr>
            </thead>
            <tbody>
                <tr th:each="user : ${users}">
                    <td th:text="${user.id}">Id</td>
                    <td th:text="${user.name}">Name</td>
                </tr>
            </tbody>
        </table>
    </body>
</html>
```
You are all set now to run the application. But before that, you need to download and configure a server like Tomcat, Jetty, or Wildflyetc in your IDE. You can download Tomcat 8 and configure your favorite IDE, run the application, and point your browser to http://localhost:8080/springmvcjpa-demo. If you do, you should see the list of user details in a table , as shown in Figure 1-1.
![image](https://user-images.githubusercontent.com/16122994/152487141-fdbc80c7-8d1e-47f1-bdf4-17fac0b6bdf8.png)

Yay…you did it. But wait, isn’t it too much work to just show a list of user details pulled from a database table?

Let’s be honest and fair. All this configuration is not just for this one use-case. This configuration becomes the basis for the rest of the application. Again, this is too much work to do if you want to quickly get up and running. Another problem with it is, assume that you want to develop another SpringMVC application with a similar technical stack. You could copy and paste the configuration and tweak it. Right?

Remember one thing: if you have to do the same thing again and again, you should find an automated way to do it.

Apart from writing the same configuration again and again, do you see any other problems here? Let’s take a look at the problems I am seeing here.

You need to hunt through all the compatible libraries for the specific Spring version and configure them.

Most of the time, you’ll configure the DataSource, EntitymanagerFactory, TransactionManager, etc. beans the same way. Wouldn’t it be great if Spring could do it for you automatically?

Similarly, you configure the SpringMVC beans like ViewResolver, MessageSource, etc. the same way most of the time. If Spring can automatically do it for you, that would be awesome.

What if Spring is capable of configuring beans automatically? What if you can customize the automatic configuration using simple customizable properties?

For example, instead of mapping the DispatcherServlet url-pattern to /, you want to map it to /app/. Instead of putting Thymeleaf views in the /WEB-INF/views folder, you want to place them in the /WEB-INF/templates/ folder.

So basically you want Spring to do things automatically, yet provide the flexibility to override the default configuration in a simpler way. You are about to enter the world of Spring Boot, where your dreams can come true!

A Quick Taste of Spring Boot
Welcome to Spring Boot! Spring Boot will configure application components automatically for you, but allows you to override the defaults if you want to.

Instead of explaining this in theory, I prefer to explain by example. In this section, you’ll see how to implement the same application, this time using Spring Boot.

Create a Maven-based Spring Boot project and configure the dependencies in the pom.xml file, as shown in Listing 1-15.

Listing 1-15. The pom.xml File
```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
        http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.apress</groupId>
    <artifactId>hello-springboot</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>hello-springboot</name>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.0.RELEASE</version>
    </parent>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
Wow, this pom.xml file suddenly become so small!

Note Don’t worry if this configuration doesn’t make sense at this point in time. You have plenty more to learn in coming chapters.

If you want to use any the MILESTONE or SNAPSHOT version of Spring Boot, you need to configure the following milestone/snapshot repositories in pom.xml.
```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.0.BUILD-SNAPSHOT</version>
</parent>

<repositories>
    <repository>
        <id>spring-snapshots</id>
        <name>Spring Snapshots</name>
        <url>https://repo.spring.io/snapshot</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
    <repository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/milestone</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>

<pluginRepositories>
    <pluginRepository>
        <id>spring-snapshots</id>
        <name>Spring Snapshots</name>
        <url>https://repo.spring.io/snapshot</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </pluginRepository>
    <pluginRepository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/milestone</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </pluginRepository>
</pluginRepositories>

```
Configure datasource/JPA properties in src/main/resources/application.properties, as shown in Listing 1-16.

Listing 1-16. The src/main/resources/application.properties File

```java
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=admin
spring.datasource.initialize=true
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```
Copy the same data.sql file into the src/main/resources folder.

Create a JPA Entity called User.java, Spring Data JPA Repository Interface called UserRepository.java, and controller called HomeController.java, as shown in the previous springmvc-jpa-demo application.

Create a Thymeleaf view to show the list of users. You can copy /WEB-INF/views/index.html, which you created in the springmvc-jpa-demo application, into the src/main/resources/templates folder of this new project.

Create a Spring Boot EntryPointclass Application.java file with the main method, as shown in Listing 1-17.

Listing 1-17. The com.apress.demo.Application.java File
```java
package com.apress.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application
{
    public static void main(String[] args)
    {
        SpringApplication.run(Application.class, args);
    }
}
```
Now run Application.java as a Java application and point your browser to http://localhost:8080/. You should see the list of users in a table format.

By you might be scratching your head, thinking “What is going on?”. The next section explains what just happened.

Easy Dependency Management
The first thing to note is the use of the dependencies named spring-boot-starter-*. Remember that I said, “Most of the time, you use the same configuration”. So when you add the springboot-starter-web dependency, it will by default pull all the commonly used libraries while developing Spring MVC applications, such as spring-webmvc, jackson-json, validation-api, and tomcat.

We added the spring-boot-starter-data-jpa dependency. This pulls all the spring-data-jpa dependencies and adds Hibernate libraries because most applications use Hibernate as a JPA implementation.

Autoconfiguration
Not only does the spring-boot-starter-web add all these libraries but it also configures the commonly registered beans like DispatcherServlet, ResourceHandlers, MessageSource, etc. with sensible defaults.

We also added spring-boot-starter-thymeleaf, which not only adds the Thymeleaf library dependencies but also configures the ThymeleafViewResolver beans automatically.

We haven’t defined any of the DataSource, EntityManagerFactory, or TransactionManager beans, but they are automatically created. How?

If you have any in-memory database drivers like H2 or HSQL in the classpath, then Spring Boot will automatically create an in-memory datasource and will register the EntityManagerFactory and TransactionManager beans automatically with sensible defaults.

But you are using MySQL, so you need to explicitly provide MySQL connection details. You have configured those MySQL connection details in the application.properties file and Spring Boot creates a DataSource using those properties.

Embedded Servlet Container Support
The most important and surprising thing is that we created a simple Java class annotated with some magical annotation (@SpringApplication), which has a main() method. By running that main() method, we are able to run the application and access it at http://localhost:8080/. Where does the servlet container come from?

We added spring-boot-starter-web, which pulls spring-boot-starter-tomcat automatically. When we run the main() method, it starts tomcat as an embedded container so that we don’t have to deploy our application on any externally installed tomcat server. What if we want to use a Jetty server instead of Tomcat? You simply exclude spring-boot-starter-tomcat from spring-boot-starter-web and include spring-boot-starter-jetty. That’s it.

But this looks magical! I can imagine what you are thinking . You are thinking like Spring Boot looks cool and it is doing a lot of things automatically for you. You still do not fully understand how it is all working behind the scenes. Right?
