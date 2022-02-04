# Web Applications with Spring Boot

Spring MVC is the most popular Java web framework based on the Model-View-Controller (MVC) design pattern. Since the Spring 3.0 version, Spring MVC has provided annotation based request mapping capabilities using @Controller and @RequestMapping. But configuring Spring MVC web application components such as DispatcherServlet, ViewResolvers, MultiPartResolver, and ExceptionHandlers is a repetitive and tedious process.

Spring Boot makes it very easy to get started with Spring MVC because the Spring Boot autoconfiguration mechanism configures most of the components such as DispatcherServlet, ViewResolvers, ContentNegotiatingViewResolver, LocaleResolver, MessageCodesResolver, etc., with default values and provides the options to customize them.

Traditionally JSPs are being used for view rendering, but there are many other view templating technologies emerged over the time such as Thymeleaf, Mustache, Groovy Templates, FreeMarker, etc. Spring Boot provides autoconfiguration for these view templating libraries as well.

Spring Boot provides embedded servlet container support so that you can build your applications as self-contained deployment units. Spring Boot supports Tomcat, Jetty, and Undertow servlet containers out-of-the-box and provides customization hooks to implement all server level customizations.

This chapter looks into how to develop Spring MVC based web applications using the Web starter with Tomcat, Jetty, and Undertow as embedded servlet containers. You will learn how to customize the default Spring MVC configuration and how to register servlets, filters, and listeners as Spring beans.

You will learn how to use Thymeleaf View templates, how to perform form validations, and how to upload files and use ResourceBundles for internationalization (i18n).

Finally, you will learn about various ways of handling exceptions using @ExceptionHandler and @ControllerAdvice annotations and how Spring Boot makes it even simpler to do so.

# Introducing SpringMVC
Spring MVC is a powerful web framework built on MVC and front controller design patterns. Spring MVC provides DispatcherServlet, which acts as a front controller by receiving all the requests and delegates the processing to request handlers (controllers). Once the processing is done, ViewResolver will render a view based on the view name. Figure 10-1 shows this flow process.

![image](https://user-images.githubusercontent.com/16122994/152397608-820511c2-d8ba-43ac-9e4d-a3843a072454.png)
Figure 10-1. SpringMVC request processing flow

Spring MVC provides annotation-based mapping support to map request URL patterns to handler classes using @Controller and @RequestMapping annotations 


# SpringMVC Annotation-Based Controller
```java
@Controller
public class HomeController
{
    @RequestMapping(value="/home", method=RequestMethod.GET)
    public String home(Model model) {
        model.addAttribute("message", "Hello Spring MVC!!");
        return "home";
    }
}
```

The @Controller annotation on the HomeController class marks it as a request handler Spring component and the home() method will handle the GET requests to the /home URL. The ViewResolver will resolve the logical view name "home" to a view template, say /WEB-INF/views/home.html, and then render the view.

Spring 4.3 introduced @GetMapping, @PostMapping, @PutMapping, etc., annotations as convenient composed annotations so that you don’t have to specify a method type in @RequestMapping(value="/url", method=RequestMethod.XXX).

```java
@Controller
public class HomeController
{

    @GetMapping("/home")
    public String home(Model model) {
        model.addAttribute("message", "Hello Spring MVC!!");
        return "home";
    }

    @PostMapping("/users")
    public String createUser(User user) {
        userRepository.save(user);
        return "users";
    }
}
```

For SpringMVC-based web applications, we need to configure various web layer components like DispatcherServlet, ViewResolvers, LocaleResolver, HandlerExceptionResolver, etc. Spring Boot provides the Web starter, which autoconfigures all these commonly used web layer components, thus making web application development much easier.

# Developing Web Application Using Spring Boot
Spring Boot provides the Web starter spring-boot-starter-web for developing web applications using Spring MVC. Spring Boot autoconfiguration registers the SpringMVC beans like DispatcherServlet, ViewResolvers, ExceptionHandlers, etc. You can develop a Spring Boot web application as a JAR type module using an embedded servlet container or as a WAR type module, which can be deployed on any external servlet container.
```java
<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
The spring-boot-starter-web starter by default configures DispatcherServlet to the URL pattern "/" and adds Tomcat as the embedded servlet container, which runs on port 8080.

Spring Boot by default serves the static resources (HTML, CSS, JS, images, etc.) from the following CLASSPATH locations:

--> static

--> public

--> resources

--> META-INF/resources


Create a Spring Boot Maven project with the Web starter and add the Bootstrap WebJars dependency.
```java
<dependencies>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.webjars.bower</groupId>
        <artifactId>bootstrap</artifactId>
        <version>3.3.7</version>
    </dependency>

</dependencies>

```
Create the styles.css stylesheet in the src/main/resources/static/css folder.

```js
body {
        background-color: #A7A5A4;
        padding-top: 50px;
}
```
Copy an image, such as spring-boot.png, into the src/main/resources/static/images folder.

Create the index.html file in the src/main/resources/public folder and add bootstrap.css to the index.html file. Use the Bootstrap navigation bar component.

```js
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8" />
<title>Home</title>
<link rel="stylesheet" href="webjars/bootstrap/3.3.7 /css/bootstrap.css" />
<link rel="stylesheet" href="css/styles.css" />
</head>
<body>

    <nav class="navbar navbar-inverse navbar-fixed-top">
      <div class="container">
        <div class="navbar-header">
          <button type="button" class="navbar-toggle collapsed"
                    data-toggle="collapse" data-target="#navbar"
                    aria-expanded="false" aria-controls="navbar">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
          <a class="navbar-brand" href="#">Project name</a>
        </div>
        <div id="navbar" class="collapse navbar-collapse">
          <ul class="nav navbar-nav">
            <li class="active"><a href="#">Home</a></li>
            <li><a href="#about">About</a></li>
            <li><a href="#contact">Contact</a></li>
          </ul>
        </div>
      </div>
    </nav>

    <div class="container">
      <h2>Hello World!!</h2>
      <img alt="SpringBoot" src="images/spring-boot.png" />
    </div>

</body>
</html>
```
Create an application entry point class.

```java
@SpringBootApplication
public class SpringbootWebDemoApplication
{
    public static void main(String[] args)
    {
        SpringApplication.run(SpringbootWebDemoApplication.class, args);
    }
}
```
Now run the SpringbootWebDemoApplication and navigate to http://localhost:8080/. You should be able to see the web page with the Bootstrap navigation bar

![image](https://user-images.githubusercontent.com/16122994/152398966-9c64d941-ec83-4a82-8e25-359e77c4e7dc.png)

By default, the Spring Boot Web starter uses Tomcat as the embedded servlet container and runs on port 8080. However, you can customize the server properties using server.* in application.properties.

```java
server.port=9090
server.servlet.context-path=/demo
server.servlet.path=/app

```

With these customizations, DispatcherServlet is configured to handle the URL pattern /app, the root contextPath will be /demo, and Tomcat now runs on port 9090. So, you would access the index.html file at http://localhost:9090/demo/app/.


# Using the Tomcat, Jetty, and Undertow Embedded Servlet Containers
As mentioned, the Spring Boot Web starter includes Tomcat as the embedded servlet container by default. Instead of Tomcat, though, you can use other servlet containers like Jetty or Undertow.

To use Jetty as the embedded container, you simply need to exclude spring-boot-starter-tomcat and add spring-boot-starter- jetty.

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>

```
Undertow ( http://undertow.io/ ) is a web server written in Java. It provides blocking and non-blocking APIs based on NIO. Spring Boot provides autoconfiguration support for the Undertow server as well. Similar to what you saw with Jetty, you can configure Spring Boot to use the Undertow embedded server instead of Tomcat as follows:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```
You can customize various properties of the Tomcat, Jetty, and Undertow servlet containers using the server.tomcat.* , server.jetty.*, and server.undertow.* properties, respectively.

```java
server.tomcat.accesslog.directory=logs # Directory in which log files are created.
server.tomcat.accesslog.enabled=false # Enable access log.
server.tomcat.accesslog.file-date-format=.yyyy-MM-dd # Date format to place in log file name.
server.tomcat.basedir= # Tomcat base directory. If not specified a temporary directory will be used.
server.tomcat.max-connections= # Maximum number of connections that the server will accept and process at any given time.
server.tomcat.max-http-header-size=0 # Maximum size in bytes of the HTTP message header.
server.tomcat.max-http-post-size=0 # Maximum size in bytes of the HTTP post content.
server.tomcat.max-threads=0 # Maximum amount of worker threads.
server.tomcat.min-spare-threads=0 # Minimum amount of worker threads.
server.tomcat.port-header=X-Forwarded-Port # Name of the HTTP header used to override the original port value.

server.jetty.acceptors= # Number of acceptor threads to use.
server.jetty.accesslog.append=false # Append to log.
server.jetty.accesslog.date-format=dd/MMM/yyyy:HH:mm:ss Z
server.jetty.accesslog.enabled=false # Enable access log.
server.jetty.accesslog.filename= # Log filename. If not specified, logs will be redirected to "System.err".
server.jetty.accesslog.log-cookies=false # Enable logging of the request cookies.
server.jetty.accesslog.log-latency=false # Enable logging of request processing time.
server.jetty.max-http-post-size=0 # Maximum size in bytes of the HTTP post or put content.

server.undertow.accesslog.dir= # Undertow access log directory.
server.undertow.accesslog.enabled=false # Enable access log.
server.undertow.accesslog.rotate=true # Enable access log rotation.
server.undertow.accesslog.suffix=log # Log file name suffix.
server.undertow.buffer-size= # Size of each buffer in bytes.
server.undertow.io-threads= # Number of I/O threads to create for the worker.
server.undertow.max-http-post-size=0 # Maximum size in bytes of the HTTP post content.

```
Use the org.springframework.boot.autoconfigure.web.ServerProperties class to see a complete list of server customization properties.

# Customizing Embedded Servlet Containers
Spring Boot provides lot of customization options for configuring servlet containers using the server.* properties. You can customize the port, connectionTimeout, contextPath, and SSL configuration parameters, as well as the session configuration parameters by configuring these properties in application.properties.

But if you need more control, you can register embedded servlet containers programmatically by registering a bean of type TomcatServletWebServerFactory, JettyServletWebServerFactory, or UndertowServletWebServerFactory based on the embedded server you want to use.

One common scenario where you would want to register embedded servlet containers programmatically is to redirect the default HTTP request to the HTTPS protocol.

Suppose your application is running on http://localhost:8080 and you want to use the HTTPS protocol. If anyone is accessing http://localhost:8080, you’ll want to redirect the request to https://localhost:8443.

First, you generate a self-signed SSL certificate using the following command:

keytool -genkey -alias mydomain -keyalg RSA -keysize 2048 -keystore KeyStore.jks -validity 3650
After providing answers to questions that keytool asks, it will generate a KeyStore.jks file and copy it to the src/main/resources folder.

Now configure the SSL properties in the application.properties file as follows:

```java
server.port=8443
server.ssl.key-store=classpath:KeyStore.jks
server.ssl.key-store-password=mysecret
server.ssl.keyStoreType=JKS
server.ssl.keyAlias=mydomain
```
If you are using the Tomcat embedded container , you can register TomcatServletWebServerFactory programmatically

```java
@Configuration
public class TomcatConfiguration
{
    @LocalServerPort
    int serverPort;

    @Bean
    public ServletWebServerFactory servletContainer() {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint securityConstraint = new SecurityConstraint();
                securityConstraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                securityConstraint.addCollection(collection);
                context.addConstraint(securityConstraint);
            }
        };

        tomcat.addAdditionalTomcatConnectors(initiateHttpConnector());
        return tomcat;
    }

    private Connector initiateHttpConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        connector.setPort(8080);
        connector.setSecure(false);
        connector.setRedirectPort(serverPort);
        return connector;
    }
}
```
With this customization, the request to http://localhost:8080/ will be automatically redirected to https://localhost:8443/.


# Customizing SpringMVC Configuration
Most of the time, Spring Boot’s default autoconfiguration, along with the customization properties, will be sufficient to tune your web application. But at times, you may need more control to configure the application components in a specific way to meet your application needs.

If you want to take advantage of Spring Boot’s autoconfiguration and add some additional MVC configuration (interceptors, formatters, view controllers, etc.), then you can create a configuration class without the @EnableWebMvc annotation, which implements WebMvcConfigurer and supplies additional configuration. 

Note
If you want complete control over the Spring MVC configuration, you can add your own configuration class annotated with @EnableWebMvc. Spring Boot’s WebMVC autoconfiguration will be completely turned off if you create a configuration class with the @Configuration and @EnableWebMvc annotations.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer
{
    @Override
    public void addViewControllers(ViewControllerRegistry registry){
        registry.addViewController("/login").setViewName("public/login");
        registry.addRedirectViewController("/", "/home");
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //Add additional interceptors here
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/assets/").addResourceLocations("/resources/assets/");
    }

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Override
    public void addFormatters(FormatterRegistry registry) {
        //Add additional formatters here
    }
}
```
SpringMVC provides WebMvcConfigurerAdapter, which is an implementation of the WebMvcConfigurer interface. But WebMvcConfigurerAdapter is deprecated as of Spring 5.0, because WebMvcConfigurer has default method implementations and uses Java 8 default method support.

# Spring Boot Web Application as a Deployable WAR
The Spring Boot web application can be developed using WAR type packaging also. The first thing you do if you want to build a deployable WAR file is change the packaging type.

If you are using Maven, then in pom.xml, change the packaging type to war.

```java
<packaging>war</packaging>
```
If you are using Gradle, you need to apply the WAR plugin.

apply plugin: 'war'
When you add the spring-boot-starter-web dependency. It will transitively add the spring-boot-starter-tomcat dependency as well. So you need to add spring-boot-starter-tomcat as the provided scope so that it won’t get packaged inside the WAR file.

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```
If you are using Gradle, add spring-boot-starter-tomcat with the providedRuntime scope as follows:

# Error Handling
You can handle exceptions in Spring MVC applications by registering the SimpleMappingExceptionResolver bean and configuring which view to render for what type of exception

```java
@Configuration
@EnableWebMvc
public class WebMvcConfig implements WebMvcConfigurer
{
    @Bean(name="simpleMappingExceptionResolver")
    public SimpleMappingExceptionResolver simpleMappingExceptionResolver()
    {
        SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();

        Properties mappings = new Properties();
        mappings.setProperty("DataAccessException", "dbError");
        mappings.setProperty("RuntimeException", "error");

        exceptionResolver.setExceptionMappings(mappings);
        exceptionResolver.setDefaultErrorView("error");
        return exceptionResolver;
    }
}

```
You can also use the @ExceptionHandler annotation to define handler methods for specific exception types

```java
@Controller
public class CustomerController
{
    @GetMapping("/customers/{id}")
    public String findCustomer(@PathVariable Long id, Model model)
    {
        Customer c = customerRepository.findById(id);
        if(c == null) throw new CustomerNotFoundException();
        model.add("customer", c);
        return "view_customer";
    }

    @ExceptionHandler(CustomerNotFoundException.class)
    public ModelAndView handleCustomerNotFoundException(CustomerNotFoundException ex)
    {
        ModelAndView model = new ModelAndView("error/404");
        model.addObject("exception", ex);
        return model;
    }
}

```
The handleCustomerNotFoundException() method in CustomerController will only handle the exception CustomerNotFoundException raised from CustomerController @RequestMapping methods.

You can handle exceptions globally by creating an exception handler class annotated with @ControllerAdvice. The @ExceptionHandler methods in the @ControllerAdvice class handle errors that occur in any controller request handling method.

```java
@ControllerAdvice
public class GlobalExceptionHandler
{
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(DataAccessException.class)
    public String handleDataAccessException(HttpServletRequest request, DataAccessException ex){
        logger.info("DataAccessException Occurred:: URL="+request.getRequestURL());
        return "db_error";
    }

    @ExceptionHandler(ServletRequestBindingException.class)
    public String servletRequestBindingException(ServletRequestBindingException e) {
        logger.error("ServletRequestBindingException occurred: "+e.getMessage());
        return "validation_error"
    }
}

```
Spring Boot registers a global error handler and maps /error by default, which renders an HTML response for browser clients and a JSON response for REST clients. You can provide the custom error page by implementing ErrorController.

```java
@Controller
public class GenericErrorController implements ErrorController
{

    private static final String ERROR_PATH = "/error";

    @RequestMapping(ERROR_PATH)
    public String error(){
        return "errorPage.html";
    }

    @Override
    public String getErrorPath() {
        return ERROR_PATH;
    }
}
```
You can also provide custom error pages based on the HTTP error status code. Spring Boot looks for error pages in the /error folder under the static resource locations (classpath:/static, classpath:/public, etc.).

For example, you can display the src/main/resources/static/error/404.html file when a 404 error occurs. Likewise, you can also display the src/main/resources/static/error/5xx.html file for all server errors with 5xx error status codes.
