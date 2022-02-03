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

@Query("select u from User u where u.email=?1 and u.password=?2 and u.enabled=true")
User findByEmailAndPassword(String email, String password);
You can also perform data update operations using @Modifying and @Query, as follows:

@Modifying
@Query("update User u set u.enabled=:status")
int updateUserStatus(@Param("status") boolean status)
Note that this example uses the named parameter :status instead of the positional parameter ?1.
