---
title: "Factory design pattern with Spring boot"
date: 2020-02-16 09:47:17 +0530
description: Implement factory design pattern in a spring boot application
tags: [ Spring Boot, Gof, Design Patterns ]
---
> Design patterns are the `best solutions` for repeated problems that occurs when developing an application.

Factory design pattern is a common solution employed in cases, where the application has multiple implementations of a service and at runtime based on a config we could select a particular implementation. An example for the above would be, we can have multiple types of message queues as Amazon SQS, Apache Kafka or ActiveMQ. Based on the configuration value provided or on the basis of the user input we might need to use a particular messaging queue service. Let's see with an example of how to implement the example scenario in a spring boot application.

Let's first create an interface for the messaging service. 
{% gist a480c2002388e64d5d51720f457ccee7 MessagingService.java %}

The interface definition has 2 methods:
- getType
- sendMessage

The method `sendMessage` will hold the implementation of each messaging service to send a message to the queue. The other method `getType` will return a string that identifies the queue service like SQS, Kafka.

I created an enum, that specifies the messaging service name which will be used to return in the `getType` method in the interface.
{% gist a480c2002388e64d5d51720f457ccee7 MessagingServiceType.java %}

The Sqs Service and Kafka Service implementation goes like this.
{% gist a480c2002388e64d5d51720f457ccee7 SqsMessagingService.java %}
{% gist a480c2002388e64d5d51720f457ccee7 KafkaMessagingService.java %}

Now let's write the factory method.
{% gist a480c2002388e64d5d51720f457ccee7 MessagingServiceFactory.java %}

As you see in the factory method, we build a hashmap with the messaging service name as the key, and MessagingService implementation as the value. All the available implementation of `MessagingService` interface i.e *KafkaMessagingService* and *SqsMessagingService* are autowired by spring with the declaration:

```java
private List<MessagingService> messagingServices;
```

Now let's write a rest api and give the control to the clients to choose the messaging service(SQS or Kafka).
{% gist a480c2002388e64d5d51720f457ccee7 Controller.java %}

If we give a cURL request like this:
```
curl -X POST \
  http://localhost:8080/messaging/SQS \
  -H 'Accept: */*' \
  -H 'Content-Type: text/plain' \
  -H 'Host: localhost:8080' \
  -H 'cache-control: no-cache' \
  -d 'Some message for sqs'
```
The response comes with a `200` status.

Now when I want to add one more messaging service, say ActiveMq I will just add the following class.

{% gist a480c2002388e64d5d51720f457ccee7 ActiveMqService.java %}

Also I just need to add the enum in `MessagingServiceType.java`, so it goes as,

{% gist a480c2002388e64d5d51720f457ccee7 MessagingServiceTypeModified.java %}

There is no need to modify any other existing class.

The complete code of this example is available in github [here][github].

[github]: https://github.com/malathit/spring-boot-best-practises