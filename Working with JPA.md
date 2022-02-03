# Working with JPA

The Java Persistence API (JPA) is an Object Relational Mapping (ORM) framework that’s part of the Java EE platform. JPA simplifies the implementation of the data access layer by letting developers work with an object oriented API instead of writing SQL queries by hand. The most popular JPA implementations are Hibernate, EclipseLink, and OpenJPA.

The Spring framework provides a Spring ORM module to integrate easily with ORM frameworks. You can also use Spring's declarative transaction management capabilities with JPA. In addition to the Spring ORM module, the Spring data portfolio project provides a consistent, Spring-based programming model for data access to relational and NoSQL datastores.

Spring Data integrates with most of the popular data access technologies, including JPA, MongoDB, Redis, Cassandra, Solr, ElasticSearch, etc.

This chapter explores the Spring Data JPA, explains how to use it with Spring Boot, and looks into how you can work with multiple databases in the same Spring Boot application.

Introducing the Spring Data JPA
Chapter 1 discussed how to develop a web application using SpringMVC and JPA without using Spring Boot. Without using Spring Boot, you need to configure various beans like DataSource, TransactionManager, LocalContainerEntityManagerFactoryBean, etc. by yourself. You can use the Spring Boot JPA Starter spring-boot-starter-data-jpa to quickly get up and running with JPA. Before getting into how to use spring-boot-starter-data-jpa, let’s first look at Spring Data JPA.

As noted in Chapter 5, Spring Data is an umbrella project that provides data access support for most of the popular data access technologies—including JPA, MongoDB, Redis, Cassandra, Solr, and ElasticSearch—in a consistent programming model. Spring Data JPA is one of the modules for working with relational databases using JPA.

At times, you may need to implement the data management applications to store, edit, and delete data. For those applications, you just need to implement CRUD (Create, Read, Update, Delete) operations for entities. Instead of implementing the same CRUD operations again and again or rolling out your own generic CRUD DAO implementation, Spring Data provides various repository abstractions, such as CrudRepository, PagingAndSortingRepository, JpaRepository, etc. They provide out-of-the-box support for CRUD operations, as well as pagination and sorting. 

![image](https://user-images.githubusercontent.com/16122994/152403378-ce0a93ae-8783-4d46-b949-7baa9b8137c1.png)


JpaRepository provides several methods for CRUD operations, along with the following interesting methods:

long count();—Returns the total number of entities available.

boolean existsById(ID id);—Returns whether an entity with the given ID exists.

List<T> findAll(Sort sort);—Returns all entities sorted by the given options.

Page<T> findAll(Pageable pageable);—Returns a page of entities meeting the paging restriction provided in the Pageable object.

Spring Data JPA not only provides CRUD operations out-of-the-box, but it also supports dynamic query generation based on the method names.

For example:

By defining a User findByEmail(String email) method, Spring Data will automatically generate the query with a where clause, as in "where email = ?1".

By defining a User findByEmailAndPassword(String email, String password) method, Spring Data will automatically generate the query with a where clause, as in "where email = ?1 and password=?2".

Note
You can also use other operators such as OR, LIKE, Between, LessThan, LessThanEqual, and so on. Refer to http://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation for a complete list of supporting operations.

But sometimes you may not be able to express your criteria using method names or the method names look ugly. Spring Data provides flexibility to configure the query explicitly using the @Query annotation.

```java
@Query("select u from User u where u.email=?1 and u.password=?2 and u.enabled=true")
User findByEmailAndPassword(String email, String password);
```
You can also perform data update operations using @Modifying and @Query, as follows:
```java
@Modifying
@Query("update User u set u.enabled=:status")
int updateUserStatus(@Param("status") boolean status)
```
Note that this example uses the named parameter :status instead of the positional parameter ?1.
  
  
# Using Spring Data JPA with Spring Boot
Now that you’ve had a glimpse of what Spring Data JPA is and what kind of features it provides, this section shows you how to put it into action.

Create a Spring Boot Maven project and add the following dependencies.
```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```
Create a JPA entity called User and a JPA repository interface called UserRepository.

Create a user JPA entity

```java
@Entity
@Table(name="USERS")
public class User
{
    @Id @GeneratedValue(strategy=GenerationType.AUTO)
    private Integer id;
    @Column(nullable=false)
    private String name;
    @Column(nullable=false, unique=true)
    private String email;
    private boolean disabled;

    //setters and getters
}
```
Create the UserRepository interface by extending the JpaRepository interface

JPA Repository Interface UserRepository.java
```java
public interface UserRepository extends JpaRepository<User, Integer>
{

}
```
You can now populate some sample data using the SQL script src/main/resources/data.sql:

```sql
insert into users(id, name, email,disabled)
values(1,'John','john@gmail.com', false);

insert into users(id, name, email,disabled)
values(2,'Rod','rod@gmail.com', false);

insert into users(id, name, email,disabled)
values(3,'Becky','becky@gmail.com', true);
```
Since you configured the in-memory database (H2) driver, Spring Boot automatically registers a DataSource. Since you added the spring-boot-starter-data-jpa dependency, Spring Boot autoconfiguration takes care of creating the JPA related beans like LocalContainerEntityManagerFactoryBean, TransactionManager, etc., automatically with sensible defaults.

Create a Spring Boot entry point class called SpringbootJPADemoApplication.java

SpringbootJPADemoApplication.java
```java
@SpringBootApplication
public class SpringbootJPADemoApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(SpringbootJPADemoApplication.class, args);
    }
}
```
Create a JUnit test class for testing the UserRepository methods, as shown in Listing 8-4.

SpringbootJPADemoApplicationTests.java
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootJPADemoApplicationTests
{

    @Autowired
    private UserRepository userRepository;

    @Test
    public void findAllUsers()  {
        List<User> users = userRepository.findAll();
        assertNotNull(users);
        assertTrue(!users.isEmpty());

    }

    @Test
    public void findUserById()  {
Optional<User> user = userRepository.getById(1);
        assertNotNull(user.get());
    }

    @Test
    public void createUser() {
        User user = new User(null, "Paul", "paul@gmail.com");
        User savedUser = userRepository.save(user);
        User newUser = userRepository.findById(savedUser.getId()).get();
        assertEquals("SivaPrasad", newUser.getName());
        assertEquals("sivaprasad@gmail.com", newUser.getEmail());
    }
}
```
Add Dynamic Query Methods
Now you’ll add some finder methods to see how dynamic query generation based on method names works.

To get a user by name, use this:

User findByName(String name)
To search for users by name, use this:

List<User> findByNameLike(String name)
The preceding method generates a where clause like where u.name like ?1.

Suppose you want to do a wildcard search, such as where u.name like %?1%. You can use @Query as follows:

  ```java
@Query("select u from User u where u.name like %?1%")
List<User> searchByName(String name)
  ```
Using the Sort and Pagination Features
Suppose you want to get all users by their names in ascending order. You can use the findAll(Sort sort) method as follows:

  ```java
Sort sort = new Sort(Direction.ASC, "name");
List<User> users = userRepository.findAll(sort);
  ```
You can also apply sorting on multiple properties, as follows:
```java
Order order1 = new Order(Direction.ASC, "name");
Order order2 = new Order(Direction.DESC, "id");
Sort sort = Sort.by(order1, order2);
List<User> users = userRepository.findAll(sort);
  ```
The users will be ordered first by name in ascending order and then by ID in descending order.

In many web applications, you’ll want to show data in a page-by-page manner. Spring Data makes it very easy to load data in the pagination style. Suppose you want to load the first 25 users on one page. We can use Pageable and PageRequest to get results by page, as follows:

  ```java
int size = 25;
int page = 0; //zero-based page index.
Pageable pageable = PageRequest.of(page, size);
Page<User> usersPage = userRepository.findAll(pageable);
  ```
The usersPage will contain the first 25 user records only. You can get additional details, like the total number of pages, the current page number, whether there is a next page, whether there is a previous page, and more.

usersPage.getTotalElements();—Returns the total amount of elements.

usersPage.getTotalPages();—Returns the total number of pages.

usersPage.hasNext();

usersPage.hasPrevious();

List<User> usersList = usersPage.getContent();

You can also apply pagination along with sorting as follows:
```java
Sort sort = new Sort(Direction.ASC, "name");
Pageable pageable = PageRequest.of(page, size, sort);
Page<User> usersPage = userRepository.findAll(pageable);
  ```
Working with Multiple Databases
Spring Boot autoconfiguration works out-of-the-box if you have single database to work with and provides plenty of customization options through its properties.

But if your application demands more control over the application configuration, you can turn off specific autoconfigurations and configure the components by yourself.

For example, you might want to use multiple databases in the same application. If you need to connect to multiple databases, you need to configure various Spring beans like DataSources, TransactionManagers, EntityManagerFactoryBeans, DataSourceInitializers, etc., explicitly.

Suppose you have an application where the security data has been stored in one database/schema and order-related data has been stored in another database/schema.

If you add the spring-boot-starter-data-jpa starter and just define the DataSource beans only, then Spring Boot will try to automatically create some beans (for example, TransactionManager), assuming there will be only one data source. It will fail.

Now you’ll see how you can work with multiple databases in Spring Boot and use the Spring Data JPA based application.

Create a Spring Boot application with the data-jpa starter. Configure the following dependencies in pom.xml:
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
 ```
Turn off the DataSource/JPA autoconfiguration. As you are going to configure the database related beans explicitly, you will turn off the DataSource/JPA autoconfiguration by excluding the AutoConfiguration classes

  
SpringbootMultipleDSDemoApplication.java
  ```java
@SpringBootApplication(
        exclude = { DataSourceAutoConfiguration.class,
                    HibernateJpaAutoConfiguration.class,
                    DataSourceTransactionManagerAutoConfiguration.class})
@EnableTransactionManagement
public class SpringbootMultipleDSDemoApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(SpringbootMultipleDSDemoApplication.class, args);
    }
}
  ```
As you have turned off AutoConfigurations, you are enabling TransactionManagement explicitly by using the @EnableTransactionManagement annotation.

Configure the datasource properties. Configure the Security and Orders database connection parameters in the application.properties file.

  ```java
datasource.security.driver-class-name=com.mysql.jdbc.Driver
datasource.security.url=jdbc:mysql://localhost:3306/security
datasource.security.username=root
datasource.security.password=admin

datasource.security.initialize=true

datasource.orders.driver-class-name=com.mysql.jdbc.Driver
datasource.orders.url=jdbc:mysql://localhost:3306/orders
datasource.orders.username=root
datasource.orders.password=admin

datasource.orders.initialize=true

hibernate.hbm2ddl.auto=update
hibernate.show-sql=true
  ```
Here, you have used custom property keys to configure the two datasource properties.

Create a security related JPA entity and a JPA repository. Then create a User entity, as follows:

  ```java
package com.apress.demo.security.entities;

@Entity
@Table(name="USERS")
public class User
{
    @Id @GeneratedValue(strategy=GenerationType.AUTO)
    private Integer id;

    @Column(nullable=false)
    private String name;

    @Column(nullable=false, unique=true)
    private String email;

    private boolean disabled;

    //setters & getters
}
  ```
Create UserRepository as follows:

  ```java
package com.apress.demo.security.repositories;

public interface UserRepository extends JpaRepository<User, Integer>
{

}
  ```
Note that you have created User.java and UserRepository.java in the com.apress.demo.security sub-packages.

Create an orders-related JPA entity and JPA repository. Create an Order entity as follows:
```java
package com.apress.demo.orders.entities;

@Entity
@Table(name="ORDERS")
public class Order
{
    @Id @GeneratedValue(strategy=GenerationType.AUTO)
    private Integer id;

    @Column(nullable=false, name="cust_name")
    private String customerName;

    @Column(nullable=false, name="cust_email")
    private String customerEmail;

    //setters & getters
}
  ```
Create the OrderRepository as follows:

  ```java
package com.apress.demo.orders.repositories;

public interface OrderRepository  extends JpaRepository<Order, Integer>
{

}
  ```
Note that you have created Order.java and OrderRepository.java in the com.apress.demo.orders sub-packages.

Create SQL scripts to initialize sample data. Create the security-data.sql script in the src/main/resources folder to initialize the USERS table with sample data.
```sql
delete from users;

insert into users(id, name, email,disabled)
values(1,'John','john@gmail.com', false);

insert into users(id, name, email,disabled)
values(2,'Rob','rob@gmail.com', false);

insert into users(id, name, email,disabled)
values(3,'Remo','remo@gmail.com', true);
  ```
Create the orders-data.sql script in the src/main/resources folder to initialize the ORDERS table with sample data.

  ```sql
delete from orders;

insert into orders(id, cust_name, cust_email)
values(1,'Andrew','andrew@gmail.com');

insert into orders(id, cust_name, cust_email)
values(2,'Paul','paul@gmail.com');

insert into orders(id, cust_name, cust_email)
values(3,'Jimmy','jimmy@gmail.com');
  ```
Create the SecurityDBConfig.java configuration class. You will configure the Spring beans such as DataSource, TransactionManager, EntityManagerFactoryBean, and DataSourceInitializer by connecting to the Security database in SecurityDBConfig.java

SecurityDBConfig.java
  ```java
package com.apress.demo.config;

@Configuration
@EnableJpaRepositories(
        basePackages = "com.apress.demo.security.repositories",
        entityManagerFactoryRef = "securityEntityManagerFactory",
        transactionManagerRef = "securityTransactionManager"
)
public class SecurityDBConfig
{
    @Autowired
    private Environment env;

    @Bean
    @ConfigurationProperties(prefix="datasource.security")
    public DataSourceProperties securityDataSourceProperties()
    {
        return new DataSourceProperties();
    }

    @Bean
    public DataSource securityDataSource()
    {
        DataSourceProperties securityDataSourceProperties = securityDataSourceProperties();
        return DataSourceBuilder.create()
                .driverClassName(securityDataSourceProperties.getDriverClassName())
                .url(securityDataSourceProperties.getUrl())
                .username(securityDataSourceProperties.getUsername())
                .password(securityDataSourceProperties.getPassword())
                .build();
    }

    @Bean
    public PlatformTransactionManager securityTransactionManager()
    {
        EntityManagerFactory factory = securityEntityManagerFactory().getObject();
        return new JpaTransactionManager(factory);
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean securityEntityManagerFactory()
    {
        LocalContainerEntityManagerFactoryBean factory =
                new LocalContainerEntityManagerFactoryBean();
        factory.setDataSource(securityDataSource());
        factory.setPackagesToScan("com.apress.demo.security.entities");
        factory.setJpaVendorAdapter(new HibernateJpaVendorAdapter());

        Properties jpaProperties = new Properties();
        jpaProperties.put("hibernate.hbm2ddl.auto", env.getProperty("hibernate.hbm2ddl.auto"));
        jpaProperties.put("hibernate.show-sql", env.getProperty("hibernate.show-sql"));
        factory.setJpaProperties(jpaProperties);

        return factory;
    }

    @Bean
    public DataSourceInitializer securityDataSourceInitializer()
    {
        DataSourceInitializer dsInitializer = new DataSourceInitializer();
        dsInitializer.setDataSource(securityDataSource());
        ResourceDatabasePopulator dbPopulator = new ResourceDatabasePopulator();
        dbPopulator.addScript(new ClassPathResource("security-data.sql"));
        dsInitializer.setDatabasePopulator(dbPopulator);
        dsInitializer.setEnabled(env.getProperty("datasource.security.initialize",
                                Boolean.class, false) );
        return dsInitializer;
    }

}
  ```
Observe that you have populated the datasource.security.* properties into DataSourceProperties by using @ConfigurationProperties(prefix="datasource.security") and DataSourceBuilder fluent API to create the DataSource bean.

While creating the LocalContainerEntityManagerFactoryBean bean, you have configured the package called com.apress.demo.security.entities to scan for JPA entities. You have configured the DataSourceInitializer bean to initialize the sample data from security-data.sql.

Finally, you enabled Spring Data JPA support by using the @EnableJpaRepositories annotation. As you are going to have multiple EntityManagerFactory and TransactionManager beans, you configured the bean IDs for entityManagerFactoryRef and transactionManagerRef by pointing to the respective bean names. You also configured the basePackages property to indicate where to look for the Spring Data JPA repositories (the packages).

Create the OrdersDBConfig.java configuration class. Similar to SecurityDBConfig.java, you will create OrdersDBConfig.java but point it to the Orders database.

OrdersDBConfig.java
  ```java
package com.apress.demo.config;

@Configuration
@EnableJpaRepositories(
        basePackages = "com.apress.demo.orders.repositories",
        entityManagerFactoryRef = "ordersEntityManagerFactory",
        transactionManagerRef = "ordersTransactionManager"
)
public class OrdersDBConfig
{

    @Autowired
    private Environment env;

    @Bean
    @ConfigurationProperties(prefix="datasource.orders")
    public DataSourceProperties ordersDataSourceProperties()
    {
        return new DataSourceProperties();
    }

    @Bean
    public DataSource ordersDataSource()
    {
        DataSourceProperties primaryDataSourceProperties = ordersDataSourceProperties();
        return DataSourceBuilder.create()
                .driverClassName(primaryDataSourceProperties.getDriverClassName())
                .url(primaryDataSourceProperties.getUrl())
                .username(primaryDataSourceProperties.getUsername())
                .password(primaryDataSourceProperties.getPassword())
                .build();
    }

    @Bean
    public PlatformTransactionManager ordersTransactionManager()
    {
        EntityManagerFactory factory = ordersEntityManagerFactory().getObject();
        return new JpaTransactionManager(factory);
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean ordersEntityManagerFactory()
    {
        LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
        factory.setDataSource(ordersDataSource());
        factory.setPackagesToScan("com.apress.demo.orders.entities");
        factory.setJpaVendorAdapter(new HibernateJpaVendorAdapter());

        Properties jpaProperties = new Properties();
        jpaProperties.put("hibernate.hbm2ddl.auto",env.getProperty("hibernate.hbm2ddl.auto"));
        jpaProperties.put("hibernate.show-sql", env.getProperty("hibernate.show-sql"));
        factory.setJpaProperties(jpaProperties);

        return factory;

    }

    @Bean
    public DataSourceInitializer ordersDataSourceInitializer()
    {
        DataSourceInitializer dsInitializer = new DataSourceInitializer();
        dsInitializer.setDataSource(ordersDataSource());
        ResourceDatabasePopulator dbPopulator = new ResourceDatabasePopulator();
        dbPopulator.addScript(new ClassPathResource("orders-data.sql"));
        dsInitializer.setDatabasePopulator(dbPopulator);
        dsInitializer.setEnabled(env.getProperty("datasource.orders.initialize",
                                Boolean.class, false));
        return dsInitializer;
    }

}
  ```
Note that you have used datasource.orders.* properties to create the DataSource bean, configured the com.apress.demo.orders.entities package to scan for JPA entities, and configured the DataSourceInitializer bean to initialize the database with sample data from orders-data.sql.

Now you’ll create a JUnit test class that invokes the JPA repository methods

SpringbootMultipleDSDemoApplicationTests.java
  ```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootMultipleDSDemoApplicationTests
{
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private OrderRepository orderRepository;

    @Test
    public void findAllUsers()
    {
        List<User> users = userRepository.findAll();
        assertNotNull(users);
        assertTrue(!users.isEmpty());
    }

    @Test
    public void findAllOrders()
    {
        List<Order> orders = orderRepository.findAll();
        assertNotNull(orders);
        assertTrue(!orders.isEmpty());
    }
}
  ```
Use OpenEntityManagerInViewFilter for Multiple Data Sources
If you have multiple database configuration set up as described in previous section in web applications, and want to use OpenEntityManagerInViewFilter to enable lazy loading of JPA entity LAZY associated collections while rendering the view, you need to register the OpenEntityManagerInViewFilter beans

WebMvcConfig.java
  ```java
@Configuration
public class WebMvcConfig
{

    @Bean
    public OpenEntityManagerInViewFilter primaryOpenEntityManagerInViewFilter()
    {
        OpenEntityManagerInViewFilter osivFilter =
                        new OpenEntityManagerInViewFilter();
        osivFilter.setEntityManagerFactoryBeanName
                        ("primaryEntityManagerFactory");
        return osivFilter;
    }

    @Bean
    public OpenEntityManagerInViewFilter reportingOpenEntityManagerInViewFilter()
    {
        OpenEntityManagerInViewFilter osivFilter =
                        new OpenEntityManagerInViewFilter();
        osivFilter.setEntityManagerFactoryBeanName
                        ("reportingEntityManagerFactory");
        return osivFilter;
    }
}
  ```
This code configures two OpenEntityManagerInViewFilter beans by setting the twoEntityManagerFactory bean names—primaryEntityManagerFactory and reportingEntityManagerFactory.

Note
To learn more about Spring Data JPA, visit the official Spring Data JPA Documentation at: http://docs.spring.io/spring-data/jpa/docs/current/reference/html .
