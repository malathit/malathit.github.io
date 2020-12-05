---
title: Spring boot validation framework
tags:
- Spring Boot
- validations
- API
- rest
category: development
excerpt: "This is a tutorial on how to validate with the spring-boot validation framework and write a custom validator"
---

**Why is validating my inputs important?**

> Humans tend to make mistakes. It is always possible that people will give a wrong input, invalid data or not giving a required data in the input. Some people may even argue that all the requests to your API comes pre-validated. What if tomorrow, some other clients wants to consume your API and that other client doesn't have a clue why things goes wrong. It is always a good idea to validate the input, as things might break sooner or later in the client.

**When should the validation be happening?**

> Validations should be done, as soon as the request comes in, and before we start any futher processing with the data. This will help keeping your code clean and gives the guarantee that the input is error-free to continue with further processing, such as storing the data in DB or calling another service or something else.

**Is there any pre-built way provided by spring boot to handle the validations?**
> Yes, Spring supports validation with its validation framework, that supports JSR-303 specification. This will help you follow the best practises/guidelines while also not let you re-invent the wheel. Let's see about `Spring validation framework` in this post.


## Spring validation framework
Spring validation framework provides 2 ways to validate the inout
*  Provide the most commonly and frequently used constraints out of the box, such as `@NotNull` or `@Min` or `@Pattern`

*  Provides support to write your own validation logic by creating your custom constraint annotation.

Let's jump to an example and see things in action.

#### 1. Define your request object
I am going to create a customer object with username, first and last names as fields.

{% gist 6a2750a06257a5612f0774e3f0faa6da CreateCustomerRequest.java %}


Please note that I am using the spring provided`@NotBlank`, `@Size`,  `@Min`,  `@Max`. However, as you might have guessed, `@UniqueUsername` is not a Spring provided constraint. Now, let's see how to implement a custom validator by defining an annotation and then defining the validation logic in the validator class

#### 2. Create a custom annotation
{% gist 6a2750a06257a5612f0774e3f0faa6da UniqueUsername.java %}

This is a custom [annotation](https://www.javatpoint.com/java-annotation) defined in java and the points of interest here are
* There is `message` value defined and the default value holds the error message that will be returned in the repsonse when there is an error. (P.S I have kept it blank, as I will be defining the error messages in the validator class)

* The value `groups` is used to specify the execution order of the validation constraints. For example, I may want my `@NotBlank` validations to be called before invoking my custom constraint validation.

* The value `payload` is useful for clients that invokes the validation and an example can be found [here](https://docs.jboss.org/hibernate/validator/5.1/reference/en-US/html/validator-customconstraints.html#validator-customconstraints-constraintannotation). However in spring boot app, the validations are triggered by spring itself and hence not useful.

* The `@COnstraint` annotation is a must, and that specifies the validators that needs to be invoked when the annotation is specified anywhere.

#### 3. Create a validator class that implements the validation logic 
{% gist 6a2750a06257a5612f0774e3f0faa6da UniqueUsernameValidator.java %}

The validation logic is implemented in the `isValid` method to check if the username is already taken. Note that the class must implement `ConstraintValidator` interface. Also, we can be ensured that spring will autowire the beans for the fields in the class. In the example above, the `userRepository`  object is autowired by Spring when creating the constructor for the class.

The validation logic is straight forward.
- If username is not given, set the error message as "Username is required". 
- If the username is already taken, then set the error mesaage as "Username taken already"

By default, when the validator returns false, the framework will return the error message specified in the `message` value in the annotation class defined in step 2.

But in our case, we will send different error message depending on the input given for the username field. This is achieved with the following snippet

```java
constraintValidatorContext.disableDefaultConstraintViolation();
constraintValidatorContext.buildConstraintViolationWithTemplate(
    "Username should be unique. '"  + username + "' is already taken").addConstraintViolation();
```
This is also useful if we want to return multiple error messages in a single validator.
#### References
* [Creating custom constraints](https://docs.jboss.org/hibernate/validator/5.1/reference/en-US/html/validator-customconstraints.html
)
