---
layout: post
title:  "Register a Servlet in Java"
categories: computer
tags:  computer copy java
author: "baeldung"
source: "http://www.baeldung.com/register-servlet"
---

* content
{:toc}


1\. Introduction  

-------------------

This article will provide an overview of how to register a servlet within Java EE and Spring Boot. Specifically, we will look at two ways to register a Java Servlet in Java EE — one using a _web.xml_ file, and the other using annotations. Then we’ll register servlets in Spring Boot using XML configuration, Java configuration, and through configurable properties.

A great introductory article on servlets can be found [here](http://www.baeldung.com/intro-to-servlets).

Registering Servlets in Java EE
-------------------------------

Let’s go over two ways to register a servlet in Java EE. First, we can register a servlet via _web.xml_. Alternatively, we can use the Java EE _@WebServlet_ annotation.

### Via web.xml

The most common way to register a servlet within your Java EE application is to add it to your _web.xml_ file:

	<welcome-file-list>  
		<welcome-file>index.html</welcome-file>  
		<welcome-file>index.htm</welcome-file>  
		<welcome-file>index.jsp</welcome-file>  
	</welcome-file-list>  
	<servlet>  
		<servlet-name>Example</servlet-name>  
		<servlet-class>com.baeldung.Example</servlet-class>  
	</servlet>  
	<servlet-mapping>  
		<servlet-name>Example</servlet-name>  
		<url-pattern>/Example</url-pattern>  
	</servlet-mapping>`

As you can see, this involves two steps: (1) adding our servlet to the _servlet_ tag, making sure to also specify the source path to the class the servlet resides within, and (2) specifying the URL path the servlet will be exposed on in the _url-pattern_ tag.

The Java EE _web.xml_ file is usually found in _WebContent/WEB-INF_.

### Via Annotations

Now let’s register our servlet using the _@WebServlet_ annotation on our custom servlet class. This eliminates the need for servlet mappings in the _server.xml_ and registration of the servlet in _web.xml_:

	@WebServlet(
	  name = "AnnotationExample",
	  description = "Example Servlet Using Annotations",
	  urlPatterns = {"/AnnotationExample"}
	)
	public class Example extends HttpServlet {  
	  
	    @Override
	    protected void doGet(
	      HttpServletRequest request, 
	      HttpServletResponse response) throws ServletException, IOException {
	  
		response.setContentType("text/html");
		PrintWriter out = response.getWriter();
		out.println("<p>Hello World!</p>");
	    }
	}

  

The code above demonstrates how to add that annotation directly to a servlet. The servlet will still be available at the same URL path as before.

<!--more-->  

Registering Servlets in Spring Boot
-----------------------------------

Now that we’ve shown how to register servlets in Java EE, let’s take a look at several ways to register servlets in a Spring Boot application.

### Programmatic Registration

Spring Boot supports 100% programmatic configuration of a web application.

First, we’ll implement the_ WebApplicationInitializer_ interface, then implement the _WebMvcConfigurer_ interface,which allows you to override preset defaults instead of having to specify each particular configuration setting, saving you time and allowing you to work with several tried and true settings out-of-the-box.

Let’s look at a sample _WebApplicationInitializer_ implementation:

	public class WebAppInitializer implements WebApplicationInitializer {
	  
	    public void onStartup(ServletContext container) throws ServletException {
		AnnotationConfigWebApplicationContext ctx
		  = new AnnotationConfigWebApplicationContext();
		ctx.register(WebMvcConfigure.class);
		ctx.setServletContext(container);
	 
		ServletRegistration.Dynamic servlet = container.addServlet(
		  "dispatcherExample", new DispatcherServlet(ctx));
		servlet.setLoadOnStartup(1);
		servlet.addMapping("/");
	     }
	}

  

Next, let’s implement the _WebMvcConfigurer_ interface:

	@Configuration
	public class WebMvcConfigure implements WebMvcConfigurer {
	 
	    @Bean
	    public ViewResolver getViewResolver() {
		InternalResourceViewResolver resolver
		  = new InternalResourceViewResolver();
		resolver.setPrefix("/WEB-INF/");
		resolver.setSuffix(".jsp");
		return resolver;
	    }
	 
	    @Override
	    public void configureDefaultServletHandling(
	      DefaultServletHandlerConfigurer configurer) {
		configurer.enable();
	    }
	 
	    @Override
	    public void addResourceHandlers(ResourceHandlerRegistry registry) {
		registry.addResourceHandler("/resources/**")
		  .addResourceLocations("/resources/").setCachePeriod(3600)
		  .resourceChain(true).addResolver(new PathResourceResolver());
	    }
	}
  

Above we specify some of the default settings for JSP servlets explicitly in order to support _.jsp_ views and static resource serving.

### XML Configuration

Another way to configure and register servlets within Spring Boot is through _web.xml_:

	<servlet>
	    <servlet-name>dispatcher</servlet-name>
	    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	    <init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/spring/dispatcher.xml</param-value>
	    </init-param>
	    <load-on-startup>1</load-on-startup>
	</servlet>
	 
	<servlet-mapping>
	    <servlet-name>dispatcher</servlet-name>
	    <url-pattern>/</url-pattern>
	</servlet-mapping>


The _web.xml_ used to specify configuration in Spring is similar to that found in Java EE. Above, you can see how we specify a few more parameters via attributes under the _servlet_ tag.

Here we use another XML to complete the configuration:

	<beans ...>
	     
	    <context:component-scan base-package="com.baeldung"/>
	 
	    <bean
	      class="org.springframework.web.servlet.view.InternalResourceViewResolver">
			<property name="prefix" value="/WEB-INF/jsp/"/>
			<property name="suffix" value=".jsp"/>
	    </bean>
	</beans>


Remember that your Spring _web.xml_ will usually live in _src/main/webapp/WEB-INF_.

### Combining XML and Programmatic Registration

Let’s mix an XML configuration approach with Spring’s programmatic configuration:

	public void onStartup(ServletContext container) throws ServletException {
	   XmlWebApplicationContext xctx = new XmlWebApplicationContext();
	   xctx.setConfigLocation('classpath:/context.xml');
	   xctx.setServletContext(container);
	 
	   ServletRegistration.Dynamic servlet = container.addServlet(
	     "dispatcher", new DispatcherServlet(ctx));
	   servlet.setLoadOnStartup(1);
	   servlet.addMapping("/");
	}  
  

### Registration by Bean

We can also programmatically configure and register our servlets using a _ServletRegistrationBean_. Below we’ll do so in order to register an _HttpServlet_ (which implements the _javax.servlet.Servlet_ interface):

	@Bean
	public ServletRegistrationBean exampleServletBean() {
	    ServletRegistrationBean bean = new ServletRegistrationBean(
	      new CustomServlet(), "/exampleServlet/*");
	    bean.setLoadOnStartup(1);
	    return bean;
	}

The main advantage of this approach is that it enables you to add both multiple servlets as well as different kinds of servlets to your Spring application.

Instead of merely utilizing a _DispatcherServlet,_ which is a more specific kind of _HttpServlet_ and the most common kind used in the _WebApplicationInitializer_ programmatic approach to configuration we explored in section 3.1, we’ll use a simpler _HttpServlet_ subclass instance which exposes the four basic _HttpRequest _operations through four functions: _doGet()_, _doPost()_, _doPut()_, and _doDelete()_ just like in Java EE.

Remember that HttpServlet is an abstract class (so it can’t be instantiated). We can whip up a custom extension easily, though:

	public class CustomServlet extends HttpServlet{
	    ...
	}

Registering Servlets with Properties
------------------------------------

Another, though uncommon, way to configure and register your servlets is to use a custom properties file loaded into the app via a _PropertyLoader, PropertySource, _or_ PropertySources _instance object_._

This provides an intermediate kind of configuration and the ability to otherwise customize _application.properties_ which provide little direct configuration for non-embedded servlets.

### System Properties Approach

We can add some custom settings to our_ application.properties_ file or another properties file. Let’s add a few settings to configure our _DispatcherServlet_:

	servlet.name=dispatcherExample
	servlet.mapping=/dispatcherExampleURL

Let’s load our custom properties into our application:

	System.setProperty("custom.config.location", "classpath:custom.properties");

And now we can access those properties via:

	System.getProperty("custom.config.location");

### Custom Properties Approach

Let’s start with a _custom.properties_ file:

	servlet.name=dispatcherExample
	servlet.mapping=/dispatcherExampleURL

We can then use a run-of-the-mill Property Loader:

	public Properties getProperties(String file) throws IOException {
	  Properties prop = new Properties();
	  InputStream input = null;
	  input = getClass().getResourceAsStream(file);
	  prop.load(input);
	  if (input != null) {
	      input.close();
	  }
	  return prop;
	}

And now we can add these custom properties as constants to our _WebApplicationInitializer_ implementation:

	private static final PropertyLoader pl = new PropertyLoader(); 
	private static final Properties springProps
	  = pl.getProperties("custom_spring.properties"); 
	 
	public static final String SERVLET_NAME
	  = springProps.getProperty("servlet.name"); 
	public static final String SERVLET_MAPPING
	  = springProps.getProperty("servlet.mapping");

We can then use them to, for example, configure our dispatcher servlet:

	ServletRegistration.Dynamic servlet = container.addServlet(
	  SERVLET_NAME, new DispatcherServlet(ctx));
	servlet.setLoadOnStartup(1);
	servlet.addMapping(SERVLET_MAPPING);`

The advantage of this approach is the absence of _.xml_ maintenance but with easy-to-modify configuration settings that don’t require redeploying the codebase.

### The PropertySource Approach

A faster way to accomplish the above is to make use of Spring’s _PropertySource_ which allows a configuration file to be accessed and loaded.

_PropertyResolver _is an interface implemented by _ConfigurableEnvironment, _which makes application properties available at servlet startup and initialization:

	@Configuration
	@PropertySource("classpath:/com/yourapp/custom.properties") 
	public class ExampleCustomConfig { 
	    @Autowired
	    ConfigurableEnvironment env; 
	 
	    public String getProperty(String key) { 
		return env.getProperty(key); 
	    } 
	}


Above, we autowire a dependency into the class and specify the location of our custom properties file. We can then fetch our salient property by calling the function_ getProperty()_ passing in the String value.

### The PropertySource Programmatic Approach

We can combine the above approach (which involves fetching property values) with the approach below (which allows us to programmatically specify those values):  

	ConfigurableEnvironment env = new StandardEnvironment(); 
	MutablePropertySources props = env.getPropertySources(); 
	Map map = new HashMap(); map.put("key", "value"); 
	props.addFirst(new MapPropertySource("Map", map));

We’ve created a map linking a key to a value then add that map to _PropertySources_ enabling invocation as needed.

Registering Embedded Servlets
-----------------------------

Lastly, we’ll also take a look at basic configuration and registration of embedded servlets within Spring Boot.

An embedded servlet provides full web container (Tomcat, Jetty, etc.) functionality without having to install or maintain the web-container separately.

You can add the required dependencies and configuration for simple live server deployment wherever such functionality is supported painlessly, compactly, and quickly.

We’ll only look at how to do this Tomcat but the same approach can be undertaken for Jetty and alternatives.

Let’s specify the dependency for an embedded Tomcat 8 web container in _pom.xml_:

<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
     <artifactId>tomcat-embed-core</artifactId>
     <version>8.5.11</version>
</dependency>

Now let’s add the tags required to successfully add Tomcat to the _.war_ produced by Maven at build-time:

	<build>
	    <finalName>embeddedTomcatExample</finalName>
	    <plugins>
		<plugin>
		    <groupId>org.codehaus.mojo</groupId>
		    <artifactId>appassembler-maven-plugin</artifactId>
		    <version>2.0.0</version>
		    <configuration>
		        <assembleDirectory>target</assembleDirectory>
		        <programs>
		            <program>
		                <mainClass>launch.Main</mainClass>
		                <name>webapp</name>
		            </program>
		    </programs>
		    </configuration>
		    <executions>
		        <execution>
		            <phase>package</phase>
		            <goals>
		                <goal>assemble</goal>
		            </goals>
		        </execution>
		    </executions>
		</plugin>
	    </plugins>
	</build>

If you are using Spring Boot, you can instead add Spring’s _spring-boot-starter-tomcat _dependency to your_ pom.xml_:
  
	<dependency>
	    <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter-tomcat</artifactId>
	    <scope>provided</scope>
	</dependency>

### Registration Through Properties

Spring Boot supports configuring most possible Spring settings through _application.properties_. After adding the necessary embedded servlet dependencies to your _pom.xml_, you can customize and configure your embedded servlet using several such configuration options:

	server.jsp-servlet.class-name=org.apache.jasper.servlet.JspServlet 
	server.jsp-servlet.registered=true
	server.port=8080
	server.servlet-path=/

Above are some of the application settings that can be used to configure the _DispatcherServlet _and static resource sharing. Settings for embedded servlets, SSL support, and sessions are also available.

There are really too many configuration parameters to list here but you can see the full list in the [Spring Boot documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html).

### Configuration Through YAML

Similarly, we can configure our embedded servlet container using YAML. This requires the use of a specialized YAML property loader — the _YamlPropertySourceLoader_ — which exposes our YAML and makes the keys and values therein available for use within our app.

	YamlPropertySourceLoader sourceLoader = new YamlPropertySourceLoader();
	PropertySource<?> yamlProps = sourceLoader.load("yamlProps", resource, null);


### Programmatic Configuration Through TomcatEmbeddedServletContainerFactory

Programmatic configuration of an embedded servlet container is possible through a subclassed instance of _EmbeddedServletContainerFactory_. For example, you can use the _TomcatEmbeddedServletContainerFactory_ to configure your embedded Tomcat servlet.

The _TomcatEmbeddedServletContainerFactory_ wraps the_ org.apache.catalina.startup.Tomcat_ object providing additional configuration options:

	@Bean
	public ConfigurableServletWebServerFactory servletContainer() {
	    TomcatServletWebServerFactory tomcatContainerFactory
	      = new TomcatServletWebServerFactory();
	    return tomcatContainerFactory;
	}

Then we can configure the returned instance:

	tomcatContainerFactory.setPort(9000);
	tomcatContainerFactory.setContextPath("/springboottomcatexample");

Each of those particular settings can be made configurable using any of the methods previously described.

We can also directly access and manipulate the_ org.apache.catalina.startup.Tomcat_ object:

	Tomcat tomcat = new Tomcat();
	tomcat.setPort(port);
	tomcat.setContextPath("/springboottomcatexample");
	tomcat.start();

Conclusion
----------

In this article, we’ve reviewed several ways to register a Servlet in a Java EE and Spring Boot application.

The source code used in this tutorial is available in the [Github project](https://github.com/eugenp/tutorials/tree/master/spring-boot).





