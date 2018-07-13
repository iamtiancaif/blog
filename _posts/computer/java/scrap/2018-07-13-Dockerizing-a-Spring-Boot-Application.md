---
layout: post
title:  "Dockerizing a Spring Boot Application"
categories: computer
tags:  computer copy java spring_cloud docker doing
author: "baeldung"
source: "http://www.baeldung.com/dockerizing-spring-boot-application"
---

* content
{:toc}


1\. Overview
------------

In this article we’ll focus on how to dockerize a _Spring Boot Application_ to run it in an isolated environment, a.k.a. _container_.

Furthermore we’ll show how to create a composition of containers, which depend on each other and are linked against each other in a virtual private network. We’ll also see how they can be managed together with single commands.

Let’s start by creating a Java-enabled, lightweight base image, running [_Alpine Linux_](https://hub.docker.com/_/alpine/).

2\. Common Base Image
---------------------

We’re going to be using _Docker’s_ own build-file format: a _Dockerfile_.

A _Dockerfile_ is in principle, a linewise batch file, containing commands to build an image. It’s not absolutely necessary to put these commands into a file, because we’re able to pass them to the command-line, as well – a file is simply more convenient.

So, let’s write our first _Dockerfile_:

	FROM alpine:edge
	MAINTAINER baeldung.com
	RUN apk add --no-cache openjdk8
	COPY files/UnlimitedJCEPolicyJDK8/* \
	  /usr/lib/jvm/java-1.8-openjdk/jre/lib/security/

*   **FROM**: The keyword _FROM_, tells _Docker_ to use a given image with its tag as build-base. If this image is not in the local library, an online-search on _[DockerHub](https://hub.docker.com/explore/)_, or on any other configured remote-registry, is performed
*   **MAINTAINER**: A _MAINTAINER_ is usually an email address, identifying the author of an image
*   **RUN**: With the _RUN_ command, we’re executing a shell command-line within the target system. Here we utilizing _Alpine Linux’s_ package manager _apk_ to install the _Java 8 OpenJDK_
*   **COPY**: The last command tells _Docker_ to _COPY_ a few files from the local file-system, specifically a subfolder to the build directory, into the image in a given path

REQUIREMENTS: In order to run the tutorial successfully, you have to download the _Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files_ from [_Oracle_](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html). Simply extract the downloaded archive into a local folder named _‘files’._

To finally build the image and store it in the local library, we have to run:

	docker build --tag=alpine-java:base --rm=true .

NOTICE: The _–tag _option will give the image its name and _–rm=true_ will remove intermediate images after it has been built successfully. The last character in this shell command is a dot, acting as a build-directory argument.

3\. Dockerize a Standalone Spring Boot Application
--------------------------------------------------

As an example for an application which we can dockerize, we will take the _[spring-cloud-config/server](https://github.com/eugenp/tutorials/tree/master/spring-cloud/spring-cloud-config)_ from the [spring cloud configuration tutorial](http://www.baeldung.com/spring-cloud-configuration). As a preparation-step, we have to assemble a runnable jar file and copy it to our _Docker_ build-directory:

	tutorials $> cd spring-cloud-config/server
	server    $> mvn package spring-boot:repackage
	server    $> cp target/server-0.0.1-SNAPSHOT.jar \
		       ../../spring-boot-docker/files/config-server.jar
	server    $> cd ../../spring-boot-docker

Now we will create a _Dockerfile_ named _Dockerfile.server_ with the following content:

	FROM alpine-java:base
	MAINTAINER baeldung.com
	COPY files/spring-cloud-config-server.jar /opt/spring-cloud/lib/
	COPY files/spring-cloud-config-server-entrypoint.sh /opt/spring-cloud/bin/
	ENV SPRING_APPLICATION_JSON= \ 
	  '{"spring": {"cloud": {"config": {"server": \
	  {"git": {"uri": "/var/lib/spring-cloud/config-repo", \
	  "clone-on-start": true}}}}}}'
	ENTRYPOINT ["/usr/bin/java"]
	CMD ["-jar", "/opt/spring-cloud/lib/spring-cloud-config-server.jar"]
	VOLUME /var/lib/spring-cloud/config-repo
	EXPOSE 8888

*   FROM: As base for our image we will take the _Java_-enabled _Alpine Linux_, created in the previous section
*   COPY: We let _Docker_ copy our jar file into the image
*   ENV: This command lets us define some environment variables, which will be respected by the application running in the container. Here we define a customized [_Spring Boot Application_ configuration](http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html), to hand-over to the jar-executable later
*   ENTRYPOINT/CMD: This will be the executable to start when the container is booting. We must define them as _JSON-Array_, because we will use an _ENTRYPOINT_ in combination with a _CMD_ for some application arguments
*   VOLUME: Because our container will be running in an isolated environment, with no direct network access, we have to define a mountpoint-placeholder for our configuration repository
*   EXPOSE: Here we are telling _Docker_, on which port our application is listing. This port will be published to the host, when the container is booting

To create an image from our _Dockerfile_, we have to run _‘docker build’_, like before:

	$> docker build --file=Dockerfile.server \
	     --tag=config-server:latest --rm=true .

But before we’re going to run a container from our image, we have to create a volume for mounting:

	$> docker volume create --name=spring-cloud-config-repo

NOTICE: While a container is immutable, when not committed to an image after application exits, data stored in a volume will be persistent over several containers.

Finally we are able to run the container from our image:

	$> docker run --name=config-server --publish=8888:8888 \
	     --volume=spring-cloud-config-repo:/var/lib/spring-cloud/config-repo \
	     config-server:latest

*   First, we have to _–name_ our container. If not, one will be automatically chosen
*   Then, we must _–publish_ our exposed port (see _Dockerfile_) to a port on our host. The value is given in the form _‘host-port:container-port’_. If only a container-port is given, a randomly chosen host-port will be used. If we leave this option out, the container will be completely isolated
*   The _–volume_ option gives access to either a directory on the host (when used with an absolute path) or a previously created _Docker_ volume (when used with a _volume-name_). The path after the colon specifies the _mountpoint_ within the container
*   As argument we have to tell _Docker_, which image to be used. Here we have to give the _image-name_ from the previously ‘_docker build_‘ step
*   Some more useful options:
    *   -it – enable interactive mode and allocate a _pseudo-tty_
    *   -d – detach from the container after booting

If we ran the container in detached mode, we can inspect its details, stop it and remove it with the following commands:

	$> docker inspect config-server
	$> docker stop config-server
	$> docker rm config-server

4\. Dockerize Dependent Applications in a Composite
---------------------------------------------------

_Docker_ commands and _Dockerfiles_ are particularly suitable for creating individual containers. But if you want to _operate on a network of isolated applications_, the container management quickly becomes cluttered.

To solve that, _Docker _provides a tool named _Docker Compose_. This comes with an own build-file in _YAML_ format and is better suited in managing multiple containers. For example: it is able to start or stop a composite of services in one command, or merges logging output of multiple services together into one _pseudo-tty_.

Let’s build an example of two applications running in different docker containers. They will communicate with each other and be presented as “single unit” to the host system. We will build and copy the _spring-cloud-config/client_ example described in the [spring cloud configuration tutorial](http://www.baeldung.com/spring-cloud-configuration) to our _files_ folder, like we have done before with the _config-server_.

This will be our _docker-compose.yml_:

	version: '2'
	services:
	    config-server:
		container_name: config-server
		build:
		    context: .
		    dockerfile: Dockerfile.server
		image: config-server:latest
		expose:
		    - 8888
		networks:
		    - spring-cloud-network
		volumes:
		    - spring-cloud-config-repo:/var/lib/spring-cloud/config-repo
		logging:
		    driver: json-file
	    config-client:
		container_name: config-client
		build:
		    context: .
		    dockerfile: Dockerfile.client
		image: config-client:latest
		entrypoint: /opt/spring-cloud/bin/config-client-entrypoint.sh
		environment:
		    SPRING_APPLICATION_JSON: \
		      '{"spring": {"cloud":  \
		      {"config": {"uri": "http://config-server:8888"}}}}'
		expose:
		    - 8080
		ports:
		    - 8080:8080
		networks:
		    - spring-cloud-network
		links:
		    - config-server:config-server
		depends_on:
		    - config-server
		logging:
		    driver: json-file
	networks:
	    spring-cloud-network:
		driver: bridge
	volumes:
	    spring-cloud-config-repo:
		external: true

*   version: Specifies which format version should be used. This is a mandatory field. Here we use the newer version, whereas the _legacy format_ is ‘1’
*   services: Each object in this key defines a _service_, a.k.a container. This section is mandatory
    *   build: If given, _docker-compose_ is able to build an image from a _Dockerfile_
        *   context: If given, it specifies the build-directory, where the _Dockerfile_ is looked-up
        *   dockerfile: If given, it sets an alternate name for a _Dockerfile_
    *   image: Tells _Docker_ which name it should give to the image when build-features are used. Otherwise it is searching for this image in the library or _remote-registry_
    *   networks: This is the identifier of the named networks to use. A given _name-value_ must be listed in the _networks_ section
    *   volumes: This identifies the named volumes to use and the mountpoints to mount the volumes to, separated by a colon. Likewise in _networks_ section, a _volume-name_ must be defined in a separate _volumes_ section
    *   links: This will create an internal network link between _this_ service and the listed service. _This_ service will be able to connect to the listed service, whereby the part before the colon specifies a _service-name_ from the _services_ section and the part after the colon specifies the hostname at which the service is listening on an exposed port
    *   depends_on: This tells _Docker_ to start a service only, if the listed services have started successfully. NOTICE: This works only at container level! For a workaround to start the dependent _application_ first, see _config-client-entrypoint.sh_
    *   logging: Here we are using the _‘json-file’_ driver, which is the default one. Alternatively _‘syslog’_ with a given address option or _‘none’ _can be used
*   networks: In this section we’re specifying the _networks _available to our services_._ In this example we let _docker-compose_ create a named _network_ of type _‘bridge’_ for us. If the option _external_ is set to _true_, it will use an existing one with the given name
*   volumes: This is very similar to the _networks_ section

Before we continue, we will check our build-file for syntax-errors:

	`$> docker-compose config`

This will be our _Dockerfile.client_ to build the _config-client_ image from. It differs from the _Dockerfile.server_ in that we additionally install _OpenBSD netcat_ (which is needed in the next step) and make the _entrypoint_ executable:

	FROM alpine-java:base
	MAINTAINER baeldung.com
	RUN apk --no-cache add netcat-openbsd
	COPY files/config-client.jar /opt/spring-cloud/lib/
	COPY files/config-client-entrypoint.sh /opt/spring-cloud/bin/
	RUN chmod 755 /opt/spring-cloud/bin/config-client-entrypoint.sh

And this will be the customized _entrypoint_ for our _config-client service_. Here we use _netcat_ in a loop to check whether our _config-server_ is ready. You have to notice, that we can reach our _config-server_ by its _link-name,_ instead of an IP address:

	#!/bin/sh
	while ! nc -z config-server 8888 ; do
	    echo "Waiting for upcoming Config Server"
	    sleep 2
	done
	java -jar /opt/spring-cloud/lib/config-client.jar

Finally we can build our images, create the defined containers and start it in one command:

	$> docker-compose up --build

To stop the containers, remove it from _Docker_ and remove the connected _networks_ and _volumes_ from it, we can use the opposite command:

	$> docker-compose down

A nice feature of _docker-compose_ is the ability to scale services. For example, we can tell _Docker_ to run one container for the _config-server_ and three containers for the _config-client_.

But for this to work properly, we have to remove the _container_name_ from our _docker-compose.yml_, for letting _Docker_ choose one, and we have to change the _exposed port configuration_, to avoiding clashes.

After that, we are able to scale our services like so:

	$> docker-compose build
	$> docker-compose up -d
	$> docker-compose scale config-server=1 config-client=3

5\. Conclusion
--------------

As we’ve seen, we are now able to build custom _Docker_ images, running a _Spring Boot Application_ as a _Docker_ container and creating dependent containers with _docker-compose_.

For further reading about the build-files, we refer to the official _[Dockerfile reference](https://docs.docker.com/engine/reference/builder/)_ and the _[docker-compose.yml reference](https://docs.docker.com/compose/compose-file/)_.

As usual, the source codes for this tutorial can be found _[on Github](https://github.com/eugenp/tutorials/tree/master/spring-cloud/spring-cloud-config/server)_.




