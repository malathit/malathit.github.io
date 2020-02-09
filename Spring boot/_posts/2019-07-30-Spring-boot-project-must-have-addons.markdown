---
layout: tech_post
title:  "Spring boot project - Must have addons!"
date:   2019-07-30 03:49:41 +0530
description: Must do things with a spring boot project
tags: spring-boot lombok swagger checkstyle docker rest-api rest
---

Having worked with *spring boot* for a while, I find that there are certain things that can be added with ease and will be quite useful. Let's see them one by one.

### 1. start.spring.io
Many of you might be aware of this site. It lets you create a spring boot project based on your preference (say maven/gradle). It helps beginners to create spring boot projects without any issues on the first go.
	
### 2. Spring boot devtools
Spring boot [devtools][devtools-link] is a must have dependency in your spring boot project. It requires zero configuration. Only thing however you need to do is to enable `Automatic build` in your IDE. Your classpath changes are hot reloaded in the server with a restart during development.

### 3. lombok
[Lombok][lombok] is used to make your pojos/dtos/entities(model) object to be cluster free with getters, setters, constructors, builders, equals(), hashCode. A single annotation as `@Getter` and you are done with getters for all the variables defined in the class.

### 4. Swagger
[Swagger][swagger] is an openApi specification - the industry statndard for RESTful API design. [Springfox][springfox] provides dependencies that creates the swagger json and the swagger UI. With zero documentation effort, you get a UI from where you can try out your rest APIs and a beautiful documentation page. However you can further customize the swagger ui page with additional swagger annotations.

### 5. Checkstyle
We all might have at some point in our life, had to look at others code. Raise your hands if you are banging your head on the code format used by the other user. Alas! You are not alone. [Checkstyle][checkstyle] helps you setup a coding standard for your project. To enable checksytle integration in your build, say as part of maven build cyle you can use [maven-checkstyle-plugin][checkstyle-plugin]. The plugin can be configured to refer to a checkstyle file(rules for code format) and then provide warnings/errors as it parses your code.

### 6. Build properties and git properties
As mentioned [here][build-info] and [here][git-info], you can add the build info and git info as properties file with `spring-boot-maven-plugin` and `git-commit-id-plugin`. Use these generated property files with swagger-ui or actuator as mentioned [here][git-build-info-tutorial].

### 7. Spotify dockerfile plugin
In the current situation many of us use docker as the container engine and deploy the same. Maintaining the docker file manually has to happen for every version. Also building & pushing the docker image would also be manual if the former is the case.  However, with the `spotify-maven-plugin`, you can build and push docker image as part of the maven build cycle. Refer to the [readme][spotify-docker-plugin] of the plugin for more info.

### 8. Modelmapper
It is a common convention to use DTO (Data transfer Object) to get input from user and return response to the user. However the spring boot application will deal with the entity objects internally. Adding converters could be a boring process and there is a high chance that you might have duplicated conversion code and it's obviously prone to mistake. [Modelmapper][model-mapper] comes to the rescue in many cases for the above scenario. 

### 9. Spring data jpa
Spring data [jpa][jpa] helps you write almost zero sql queries and minimal data retreival and storage code. It also helps enable automatic [auditing][jpa-auditing] on your entities.

That's all for now folks!! I will update this post, if I come to know of additional add-ons.

[jpa-auditing]: https://www.baeldung.com/database-auditing-jpa
[jpa]: https://spring.io/projects/spring-data-jpa
[model-mapper]: https://www.baeldung.com/entity-to-and-from-dto-for-a-java-spring-application
[spotify-docker-plugin]: https://github.com/spotify/dockerfile-maven/blob/master/README.md
[git-build-info-tutorial]: https://www.baeldung.com/spring-git-information
[git-info]: https://docs.spring.io/spring-boot/docs/current/reference/html/howto-build.html#howto-git-info
[build-info]: https://docs.spring.io/spring-boot/docs/current/reference/html/howto-build.html#howto-build-info
[checkstyle-plugin]: https://www.baeldung.com/checkstyle-java
[checkstyke]: https://checkstyle.sourceforge.io/
[devtools-link]: https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html
[lombok]: https://projectlombok.org/
[swagger]: https://swagger.io/
[springfox]: https://www.baeldung.com/swagger-2-documentation-for-spring-rest-api