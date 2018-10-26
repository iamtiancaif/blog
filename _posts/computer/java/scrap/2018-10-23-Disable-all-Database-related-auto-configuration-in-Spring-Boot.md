---
layout: post
title: "Disable all Database related auto configuration in Spring Boot"
categories: computer
tags: computer copy scrap java
author: "Bill Venners"
source: "https://www.javaworld.com/article/2076971/java-concurrency/how-the-java-virtual-machine-performs-thread-synchronization.html"
---

* content
{:toc}


The way I would do similar thing is:

    @Configuration
    @EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class, DataSourceTransactionManagerAutoConfiguration.class, HibernateJpaAutoConfiguration.class})
    @Profile ("client_app_profile_name")
    public class ClientAppConfiguration {
        //it can be left blank
    }

Write similar one for the server app (without excludes).

Last step is to disable Auto Configuration from main spring boot class:

    @SpringBootApplication
    public class SomeApplication extends SpringBootServletInitializer {
    
        public static void main(String[] args) {
            SpringApplication.run(SomeApplication.class);
        }
    
        protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
            return application.sources(SomeApplication.class);
        }
    }

Change: `@SpringBootApplication` into:

    @Configuration 
    @ComponentScan

This should do the job. Now, the dependencies that I excluded in the example might be incomplete. They were enough for me, but im not sure if its all to completely disable database related libraries. Check the list below to be sure:

[http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#auto-configuration-classes](http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#auto-configuration-classes)

Hope that helps









