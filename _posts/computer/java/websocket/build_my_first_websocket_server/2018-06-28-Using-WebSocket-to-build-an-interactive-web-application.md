---
layout: post
title:  "Using WebSocket to build an interactive web application"
categories: computer
tags:  computer copy java spring websocket first_learn
author: "web"
source: "https://spring.io/guides/gs/messaging-stomp-websocket/"
---

* content
{:toc}


This guide walks you through the process of creating a "hello world" application that sends messages back and forth, between a browser and the server. WebSocket is a very thin, lightweight layer above TCP. It makes it very suitable to use "subprotocols" to embed messages. In this guide we’ll dive in and use [STOMP](https://en.wikipedia.org/wiki/Streaming_Text_Oriented_Messaging_Protocol) messaging with Spring to create an interactive web application.

What you’ll build
-----------------

You’ll build a server that will accept a message carrying a user’s name. In response, it will push a greeting into a queue that the client is subscribed to.

What you’ll need
----------------

*   About 15 minutes
    
*   A favorite text editor or IDE
    
*   [JDK 1.8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) or later
    
*   [Gradle 4+](http://www.gradle.org/downloads) or [Maven 3.2+](https://maven.apache.org/download.cgi)
    
*   You can also import the code straight into your IDE:
    
    *   [Spring Tool Suite (STS)](https://spring.io/guides/gs/sts)
        
    *   [IntelliJ IDEA](https://spring.io/guides/gs/intellij-idea/)
        
    

How to complete this guide
--------------------------

Like most Spring [Getting Started guides](https://spring.io/guides), you can start from scratch and complete each step, or you can bypass basic setup steps that are already familiar to you. Either way, you end up with working code.

To start from scratch, move on to [Build with Gradle](https://spring.io/guides/gs/messaging-stomp-websocket/#scratch).

To skip the basics, do the following:

*   [Download](https://github.com/spring-guides/gs-messaging-stomp-websocket/archive/master.zip) and unzip the source repository for this guide, or clone it using [Git](https://spring.io/understanding/Git): `git clone [https://github.com/spring-guides/gs-messaging-stomp-websocket.git](https://github.com/spring-guides/gs-messaging-stomp-websocket.git)`
    
*   cd into `gs-messaging-stomp-websocket/initial`
    
*   Jump ahead to [Create a resource representation class](https://spring.io/guides/gs/messaging-stomp-websocket/#initial).
    

When you’re finished, you can check your results against the code in `gs-messaging-stomp-websocket/complete`.

<!--more-->  

Build with Gradle
-----------------

Build with Maven
----------------

First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Maven](https://maven.apache.org/) is included here. If you’re not familiar with Maven, refer to [Building Java Projects with Maven](https://spring.io/guides/gs/maven).

### Create the directory structure

In a project directory of your choosing, create the following subdirectory structure; for example, with `mkdir -p src/main/java/hello` on *nix systems:

└── src
    └── main
        └── java
            └── hello

`pom.xml`

    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
    
        <groupId>org.springframework</groupId>
        <artifactId>gs-messaging-stomp-websocket</artifactId>
        <version>0.1.0</version>
    
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.0.3.RELEASE</version>
        </parent>
    
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-websocket</artifactId>
            </dependency>
    
            <dependency>
                <groupId>org.webjars</groupId>
                <artifactId>webjars-locator-core</artifactId>
            </dependency>
            <dependency>
                <groupId>org.webjars</groupId>
                <artifactId>sockjs-client</artifactId>
                <version>1.0.2</version>
            </dependency>
            <dependency>
                <groupId>org.webjars</groupId>
                <artifactId>stomp-websocket</artifactId>
                <version>2.3.3</version>
            </dependency>
            <dependency>
                <groupId>org.webjars</groupId>
                <artifactId>bootstrap</artifactId>
                <version>3.3.7</version>
            </dependency>
            <dependency>
                <groupId>org.webjars</groupId>
                <artifactId>jquery</artifactId>
                <version>3.1.0</version>
            </dependency>
    
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
        </dependencies>
    
        <properties>
            <java.version>1.8</java.version>
        </properties>
    
        <build>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    
    </project>

The [Spring Boot Maven plugin](https://docs.spring.io/spring-boot/docs/current/maven-plugin) provides many convenient features:

*   It collects all the jars on the classpath and builds a single, runnable "über-jar", which makes it more convenient to execute and transport your service.
    
*   It searches for the `public static void main()` method to flag as a runnable class.
    
*   It provides a built-in dependency resolver that sets the version number to match [Spring Boot dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-dependencies/pom.xml). You can override any version you wish, but it will default to Boot’s chosen set of versions.
    

Build with your IDE
-------------------

Create a resource representation class
--------------------------------------

Now that you’ve set up the project and build system, you can create your STOMP message service.

Begin the process by thinking about service interactions.

The service will accept messages containing a name in a STOMP message whose body is a [JSON](https://spring.io/understanding/JSON) object. If the name given is "Fred", then the message might look something like this:

    {
        "name": "Fred"
    }

To model the message carrying the name, you can create a plain old Java object with a `name` property and a corresponding `getName()` method:

`src/main/java/hello/HelloMessage.java`

    package hello;
    
    public class HelloMessage {
    
        private String name;
    
        public HelloMessage() {
        }
    
        public HelloMessage(String name) {
            this.name = name;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    }

Upon receiving the message and extracting the name, the service will process it by creating a greeting and publishing that greeting on a separate queue that the client is subscribed to. The greeting will also be a JSON object, which might look something like this:

    {
        "content": "Hello, Fred!"
    }

To model the greeting representation, you add another plain old Java object with a `content`property and corresponding `getContent()` method:

`src/main/java/hello/Greeting.java`

    package hello;
    
    public class Greeting {
    
        private String content;
    
        public Greeting() {
        }
    
        public Greeting(String content) {
            this.content = content;
        }
    
        public String getContent() {
            return content;
        }
    
    }

Spring will use the [Jackson JSON](http://wiki.fasterxml.com/JacksonHome) library to automatically marshal instances of type `Greeting` into JSON.

Next, you’ll create a controller to receive the hello message and send a greeting message.

Create a message-handling controller
------------------------------------

In Spring’s approach to working with STOMP messaging, STOMP messages can be routed to [`@Controller`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Controller.html) classes. For example the `GreetingController` is mapped to handle messages to destination "/hello".

`src/main/java/hello/GreetingController.java`

    package hello;
    
    import org.springframework.messaging.handler.annotation.MessageMapping;
    import org.springframework.messaging.handler.annotation.SendTo;
    import org.springframework.stereotype.Controller;
    import org.springframework.web.util.HtmlUtils;
    
    @Controller
    public class GreetingController {
    
    
        @MessageMapping("/hello")
        @SendTo("/topic/greetings")
        public Greeting greeting(HelloMessage message) throws Exception {
            Thread.sleep(1000); // simulated delay
            return new Greeting("Hello, " + HtmlUtils.htmlEscape(message.getName()) + "!");
        }
    
    }

This controller is concise and simple, but there’s plenty going on. Let’s break it down step by step.

The [`@MessageMapping`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/messaging/handler/annotation/MessageMapping.html) annotation ensures that if a message is sent to destination "/hello", then the `greeting()` method is called.

The payload of the message is bound to a `HelloMessage` object which is passed into `greeting()`.

Internally, the implementation of the method simulates a processing delay by causing the thread to sleep for 1 second. This is to demonstrate that after the client sends a message, the server can take as long as it needs to process the message asynchronously. The client may continue with whatever work it needs to do without waiting on the response.

After the 1 second delay, the `greeting()` method creates a `Greeting` object and returns it. The return value is broadcast to all subscribers to "/topic/greetings" as specified in the [`@SendTo`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/messaging/handler/annotation/SendTo.html) annotation. Note that the name from the input message is sanitized since in this case it will be echoed back and re-rendered in the browser DOM on the client side.

Configure Spring for STOMP messaging
------------------------------------

Now that the essential components of the service are created, you can configure Spring to enable WebSocket and STOMP messaging.

Create a Java class named `WebSocketConfig` that looks like this:

`src/main/java/hello/WebSocketConfig.java`

    package hello;
    
    import org.springframework.context.annotation.Configuration;
    import org.springframework.messaging.simp.config.MessageBrokerRegistry;
    import org.springframework.web.socket.config.annotation.EnableWebSocketMessageBroker;
    import org.springframework.web.socket.config.annotation.StompEndpointRegistry;
    import org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer;
    
    @Configuration
    @EnableWebSocketMessageBroker
    public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    
        @Override
        public void configureMessageBroker(MessageBrokerRegistry config) {
            config.enableSimpleBroker("/topic");
            config.setApplicationDestinationPrefixes("/app");
        }
    
        @Override
        public void registerStompEndpoints(StompEndpointRegistry registry) {
            registry.addEndpoint("/gs-guide-websocket").withSockJS();
        }
    
    }

`WebSocketConfig` is annotated with `@Configuration` to indicate that it is a Spring configuration class. It is also annotated [`@EnableWebSocketMessageBroker`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/messaging/simp/config/EnableWebSocketMessageBroker.html). As its name suggests, `@EnableWebSocketMessageBroker` enables WebSocket message handling, backed by a message broker.

The `configureMessageBroker()` method implements the default method in `WebSocketMessageBrokerConfigurer` to configure the message broker. It starts by calling `enableSimpleBroker()` to enable a simple memory-based message broker to carry the greeting messages back to the client on destinations prefixed with "/topic". It also designates the "/app" prefix for messages that are bound for `@MessageMapping`-annotated methods. This prefix will be used to define all the message mappings; for example, "/app/hello" is the endpoint that the `GreetingController.greeting()` method is mapped to handle.

The `registerStompEndpoints()` method registers the "/gs-guide-websocket" endpoint, enabling SockJS fallback options so that alternate transports may be used if WebSocket is not available. The SockJS client will attempt to connect to "/gs-guide-websocket" and use the best transport available (websocket, xhr-streaming, xhr-polling, etc).

Create a browser client
-----------------------

With the server side pieces in place, now let’s turn our attention to the JavaScript client that will send messages to and receive messages from the server side.

Create an index.html file that looks like this:

`src/main/resources/static/index.html`

    <!DOCTYPE html>
    <html>
    <head>
        <title>Hello WebSocket</title>
        <link href="/webjars/bootstrap/css/bootstrap.min.css" rel="stylesheet">
        <link href="/main.css" rel="stylesheet">
        <script src="/webjars/jquery/jquery.min.js"></script>
        <script src="/webjars/sockjs-client/sockjs.min.js"></script>
        <script src="/webjars/stomp-websocket/stomp.min.js"></script>
        <script src="/app.js"></script>
    </head>
    <body>
    <noscript><h2 style="color: #ff0000">Seems your browser doesn't support Javascript! Websocket relies on Javascript being
        enabled. Please enable
        Javascript and reload this page!</h2></noscript>
    <div id="main-content" class="container">
        <div class="row">
            <div class="col-md-6">
                <form class="form-inline">
                    <div class="form-group">
                        <label for="connect">WebSocket connection:</label>
                        <button id="connect" class="btn btn-default" type="submit">Connect</button>
                        <button id="disconnect" class="btn btn-default" type="submit" disabled="disabled">Disconnect
                        </button>
                    </div>
                </form>
            </div>
            <div class="col-md-6">
                <form class="form-inline">
                    <div class="form-group">
                        <label for="name">What is your name?</label>
                        <input type="text" id="name" class="form-control" placeholder="Your name here...">
                    </div>
                    <button id="send" class="btn btn-default" type="submit">Send</button>
                </form>
            </div>
        </div>
        <div class="row">
            <div class="col-md-12">
                <table id="conversation" class="table table-striped">
                    <thead>
                    <tr>
                        <th>Greetings</th>
                    </tr>
                    </thead>
                    <tbody id="greetings">
                    </tbody>
                </table>
            </div>
        </div>
    </div>
    </body>
    </html>

This HTML file imports the `SockJS` and `STOMP` javascript libraries that will be used to communicate with our server using STOMP over websocket. We’re also importing here an `app.js` which contains the logic of our client application.

Let’s create that file:

`src/main/resources/static/app.js`

    var stompClient = null;
    
    function setConnected(connected) {
        $("#connect").prop("disabled", connected);
        $("#disconnect").prop("disabled", !connected);
        if (connected) {
            $("#conversation").show();
        }
        else {
            $("#conversation").hide();
        }
        $("#greetings").html("");
    }
    
    function connect() {
        var socket = new SockJS('/gs-guide-websocket');
        stompClient = Stomp.over(socket);
        stompClient.connect({}, function (frame) {
            setConnected(true);
            console.log('Connected: ' + frame);
            stompClient.subscribe('/topic/greetings', function (greeting) {
                showGreeting(JSON.parse(greeting.body).content);
            });
        });
    }
    
    function disconnect() {
        if (stompClient !== null) {
            stompClient.disconnect();
        }
        setConnected(false);
        console.log("Disconnected");
    }
    
    function sendName() {
        stompClient.send("/app/hello", {}, JSON.stringify({'name': $("#name").val()}));
    }
    
    function showGreeting(message) {
        $("#greetings").append("<tr><td>" + message + "</td></tr>");
    }
    
    $(function () {
        $("form").on('submit', function (e) {
            e.preventDefault();
        });
        $( "#connect" ).click(function() { connect(); });
        $( "#disconnect" ).click(function() { disconnect(); });
        $( "#send" ).click(function() { sendName(); });
    });

The main piece of this JavaScript file to pay attention to is the `connect()` and `sendName()` functions.

The `connect()` function uses [SockJS](https://github.com/sockjs) and [stomp.js](http://jmesnil.net/stomp-websocket/doc/) to open a connection to "/gs-guide-websocket", which is where our SockJS server is waiting for connections. Upon a successful connection, the client subscribes to the "/topic/greetings" destination, where the server will publish greeting messages. When a greeting is received on that destination, it will append a paragraph element to the DOM to display the greeting message.

The `sendName()` function retrieves the name entered by the user and uses the STOMP client to send it to the "/app/hello" destination (where `GreetingController.greeting()`will receive it).

Make the application executable
-------------------------------

Although it is possible to package this service as a traditional [WAR](https://spring.io/understanding/WAR) file for deployment to an external application server, the simpler approach demonstrated below creates a standalone application. You package everything in a single, executable JAR file, driven by a good old Java `main()` method. Along the way, you use Spring’s support for embedding the [Tomcat](https://spring.io/understanding/Tomcat) servlet container as the HTTP runtime, instead of deploying to an external instance.

`src/main/java/hello/Application.java`

    package hello;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    
    @SpringBootApplication
    public class Application {
    
        public static void main(String[] args) {
            SpringApplication.run(Application.class, args);
        }
    }

`@SpringBootApplication` is a convenience annotation that adds all of the following:

*   `@Configuration` tags the class as a source of bean definitions for the application context.
    
*   `@EnableAutoConfiguration` tells Spring Boot to start adding beans based on classpath settings, other beans, and various property settings.
    
*   Normally you would add `@EnableWebMvc` for a Spring MVC app, but Spring Boot adds it automatically when it sees spring-webmvc on the classpath. This flags the application as a web application and activates key behaviors such as setting up a `DispatcherServlet`.
    
*   `@ComponentScan` tells Spring to look for other components, configurations, and services in the `hello` package, allowing it to find the controllers.
    

The `main()` method uses Spring Boot’s `SpringApplication.run()` method to launch an application. Did you notice that there wasn’t a single line of XML? No web.xml file either. This web application is 100% pure Java and you didn’t have to deal with configuring any plumbing or infrastructure.

### Build an executable JAR

You can run the application from the command line with Gradle or Maven. Or you can build a single executable JAR file that contains all the necessary dependencies, classes, and resources, and run that. This makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

If you are using Gradle, you can run the application using `./gradlew bootRun`. Or you can build the JAR file using `./gradlew build`. Then you can run the JAR file:

java -jar build/libs/gs-messaging-stomp-websocket-0.1.0.jar

If you are using Maven, you can run the application using `./mvnw spring-boot:run`. Or you can build the JAR file with `./mvnw clean package`. Then you can run the JAR file:

java -jar target/gs-messaging-stomp-websocket-0.1.0.jar

The procedure above will create a runnable JAR. You can also opt to [build a classic WAR file](https://spring.io/guides/gs/convert-jar-to-war/) instead.

Logging output is displayed. The service should be up and running within a few seconds.

Test the service
----------------

Now that the service is running, point your browser at [http://localhost:8080](http://localhost:8080/) and click the "Connect" button.

Upon opening a connection, you are asked for your name. Enter your name and click "Send". Your name is sent to the server as a JSON message over STOMP. After a 1-second simulated delay, the server sends a message back with a "Hello" greeting that is displayed on the page. At this point, you can send another name, or you can click the "Disconnect" button to close the connection.

Summary
-------

Congratulations! You’ve just developed a STOMP-based messaging service with Spring.

See Also
--------

The following guides may also be helpful:

*   [Serving Web Content with Spring MVC](https://spring.io/guides/gs/serving-web-content/)
    
*   [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/)
    

Want to write a new guide or contribute to an existing one? Check out our [contribution guidelines](https://github.com/spring-guides/getting-started-guides/wiki).

All guides are released with an ASLv2 license for the code, and an [Attribution, NoDerivatives creative commons license](https://creativecommons.org/licenses/by-nd/3.0/) for the writing.  






