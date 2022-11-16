# Dependency Injection

## Learning Goals

- Manually link or wire beans using methods in the config class.
- Auto-wire beans using the `@Autowired` annotation.

## Introduction

Now that we understand a little more about beans and the application context,
let's discuss how we can use these beans and reference them.

In our spring-web-demo project, we referenced the `LunchService` in the
`LunchController` class like this:

```java
    // Add a reference to the LunchService
    private final LunchService lunchService;
    
    // Add a constructor
    @Autowired
    public LunchController(LunchService lunchService) {
        this.lunchService = lunchService;
    }
```

In that lesson we referred to this as "Spring magic". But this "Spring magic"
really is something we call dependency injection in action!
**Dependency injection** is a concept used in the Spring framework to inject
objects into other objects. In traditional programming, which we have used thus
far, we would do something like this instead:

```java
this.lunchService = new LunchService();
```

But with dependency injection, we don't have to instantiate the `LunchService`.
Instead, this technique allows us to loosely couple our components and shift the
work of instantiation onto Spring to manage our beans within the application
context! In other words, it will look at other beans defined in the context to
inject the specific field or parameter values. Here are some advantages of
dependency injection:

- Loosely coupled: If we remember from an earlier lesson, coupling refers to the
  dependency two classes may have on each other. Code that is loosely coupled is
  easier to reuse, maintain, and test as it reduces the number of dependencies.
- Readability: Dependency injection reduces the amount of cluttered logic that
  is often found in constructors.

In this lesson, we will go into more detail about what is happening under the hood
here and how we can reference one bean in another through the use of dependency
injection.

## Defining the Relationship

Continuing with the same example from the last lesson, let's say we have a
`Human` instance in our Spring context along with the `Dog` instance. We will
connect them together such that the `Human` instance "has a" `Dog`.

![Spring context with a human and a dog bean with a link between them](https://curriculum-content.s3.amazonaws.com/java-spring-1/spring-context-human-dog-linked.png)

We'll create a `Human` class in the `domain` package before proceeding forward:

```java
package com.example.demo.domain;

public class Human {
    private String name;
    private Dog dog;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    @Override
    public String toString() {
        return "Human{" +
                "name='" + name + '\'' +
                ", dog=" + dog +
                '}';
    }
}
```

We will look at two broad ways of creating the relationship:

1. Wire the beans together by defining the relationship in the configuration
   class.
2. Use the `@Autowired` annotation to inject beans.

Both of these approaches will require us to first add the `Human` and `Dog` beans
to the application context before linking them together.

We'll use the `@Bean` annotation in the first option and then the `@Component`
annotation in the second option, as the `@Autowired` annotation is typically used
with component classes.

## Add the Beans to the Context

First, we need to use `@Bean` on the methods instead of `@Component`. Let's reset
the `Dog` class to remove the `@Component` annotation:

```java
package com.example.demo.domain;

public class Dog {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

   @Override
   public String toString() {
      return "Dog{" +
              "name='" + name + '\'' +
              '}';
   }
}
```

We need to define the methods for creating the `Human` and `Dog` beans before we
can create a relationship between them.

Open the `MyConfig` class and add the following code:

```java
package com.example.demo.config;

import com.example.demo.domain.Dog;
import com.example.demo.domain.Human;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyConfig {
    @Bean
    public Dog dog() {
        Dog d = new Dog();
        d.setName("Luna");
        return d;
    }

    @Bean
    public Human human() {
        Human h = new Human();
        h.setName("Lily");
        return h;
    }
}
```

Then open the `DemoApplication` class and add the following code:

```java
package com.example.demo;

import com.example.demo.config.MyConfig;
import com.example.demo.domain.Dog;
import com.example.demo.domain.Human;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
       SpringApplication.run(DemoApplication.class, args);
        
        // create context
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);

        // access beans
        Human lily = context.getBean(Human.class);
        Dog luna = context.getBean(Dog.class);

        System.out.println(lily);
        System.out.println(luna);
    }
}
```

```java
Human{name='Lily', dog=null}
Dog{name='Luna'}
```

Running the application adds the beans to the Spring context but does not
define any relationship between the beans as indicated by the `dog=null` value of
the `Human` instance. This is what the Spring context looks like right now:

![Spring context with a Human and a Dog bean without any links](https://curriculum-content.s3.amazonaws.com/java-spring-1/spring-context-human-dog-separate.png)

And this is what we want to make it look like:

![Diagram showing the change from a Spring context with unlinked beans to a Spring context with the beans linked](https://curriculum-content.s3.amazonaws.com/java-spring-1/spring-context-human-dog-separate-to-linked.png)

## Setter-Based Dependency Injection

As mentioned before, there are two ways to create this relationship. We will first
explain how to wire beans by calling a bean method within another bean
method.

We will be calling the `dog()` method in the `human()` method by setting the
`dog` property of the `Human` instance.

Update the `human()` method in the `MyConfig` class to the following:

```java
@Bean
public Human human() {
    Human h = new Human();
    h.setName("Lily");
    h.setDog(dog());
    return h;
}
```

Now if we run the `DemoApplication` of the `DemoApplication` class, we will get the
following output:

```java
Human{name='Lily', dog=Dog{name='Luna'}}
Dog{name='Luna'}
```

Spring is performing the following steps when we run the `main` method:

1. Calls the `dog()` method and add the returned value to the context.
2. Calls the `human()` method.
   1. When it encounters the `dog()` method call in the method body, it looks
      for a `Dog` bean in its context.
   2. It assigns the `Dog` bean reference to the `dog` property of the `Human`
      instance.

Spring does not create multiple instances of the `Dog` bean unless we explicitly
define multiple methods that return a `Dog` instance.

We can also define a `Dog` parameter for the `human()` method which will cause
Spring to automatically reference the `Dog` bean in its context and assign it to
the `Human` bean’s `dog` property.

Open the `MyConfig` file and update the `human()` method like so:

```java
@Bean
public Human human(Dog dog) {
    Human h = new Human();
    h.setName("Lily");
    h.setDog(dog);
    return h;
}
```

Now run the `DemoApplication` again. It should give same output as we saw above:

```java
Human{name='Lily', dog=Dog{name='Luna'}}
Dog{name='Luna'}
```

Spring performs the following steps here:

1. Calls the `dog()` method and stores the returned instance in its context.
2. Calls the `human()` method. It sees that the method requires a `Dog` instance
   so, it checks its context for one and _injects_ the `Dog` instance into the
   parameter of the `human()` method.

When the dependencies are not managed directly by application code it is
referred to as _dependency injection_. It’s a technique used by the Spring
framework to set the specific field or parameter values.

## Use `@Autowired` for Bean Injection

The `@Autowired` annotation is another way to define links between beans in the
Spring context. They can be used with both `@Bean` methods or with components
(classes with stereotype annotation). Generally, they are mainly used with
components.

We mark specific fields or methods in a class with the `@Autowired` annotation
to signal the dependency to Spring.

There are three ways to use the `@Autowired` annotation:

1. Annotate a class constructor with parameters.
2. Annotate a class field or property.
3. Annotate the setter method.

Make sure to update the files to use `@Component` annotation before proceeding.
We will be modifying the `Human` class to define the links since it has the
`dog` field or property.

```java
// MyConfig.java

package com.example.demo.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "com.example.demo.domain")
public class MyConfig {

}
```

```java
// Dog.java

package com.example.demo.domain;

import org.springframework.stereotype.Component;

@Component
public class Dog {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Dog{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

```java
// Human.java

package com.example.demo.domain;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class Human {
    private String name;
    private Dog dog;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Dog getDog() {
        return dog;
    }

    public void setDog(Dog dog) {
        this.dog = dog;
    }

    @Override
    public String toString() {
        return "Human{" +
                "name='" + name + '\'' +
                ", dog=" + dog +
                '}';
    }
}
```

```java
// DemoApplication.java

package com.example.demo;

import com.example.demo.config.MyConfig;
import com.example.demo.domain.Dog;
import com.example.demo.domain.Human;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
        // create context
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);

        // access beans
       Human lily = context.getBean(Human.class);
       Dog luna = context.getBean(Dog.class);

        System.out.println(lily);
        System.out.println(luna);
    }
}
```

### Annotate Class Constructor

We have to create a constructor in the `Human` class that takes a `Dog` instance
as an argument and assigns it to the `dog` field in the class. Then we annotate
the constructor with `@Autowired`.

The most common way to use the `@Autowired` annotation is to create a constructor
in the class that will reference another bean. In our case, we'll create an
autowired constructor in the `Human` class that takes a `Dog` instance as an
argument.

```java
@Component
public class Human {
    private String name;
    private Dog dog;

    @Autowired
    public Human(Dog dog) {
       
    }
    
    // getters and setters
}
```

Since we are using the `@Autowired` annotation, we don't have to instantiate the
`dog` field in the `Human` class - we can just assign it to the
`Dog` argument passed into the constructor:

```java
@Component
public class Human {
    private String name;
    private Dog dog;

    @Autowired
    public Human(Dog dog) {
       this.dog = dog;
    }
    
    // getters and setters
}
```

We can now also make the `dog` instance variable `final` as well, since the value
is injected during object creation.

```java
@Component
public class Human {
    private String name;
    private final Dog dog;

    @Autowired
    public Human(Dog dog) {
       this.dog = dog;
    }
    
    // getters and setters
    // no setter for the dog since it is now a final field
}
```

If we run the application we will get the following output:

```java
Human{name='Lily', dog=Dog{name='Luna'}}
Dog{name='Luna'}
```

When Spring looks at the constructor, it searches for a `Dog` bean in its scope
and invokes the constructor with the `Dog` bean.

### Annotate the Class Field

We can annotate the field directly without using a constructor.

```java
@Component
public class Human {
    private String name;

    @Autowired
    private Dog dog;

    // getters, setters, and toString
}
```

This will create a link between the `Human` and `Dog` beans in the context. This
approach is not preferred because we can’t set the `dog` to be `final`. By not
setting the `dog` field to final, this would allow anyone to change the value. It
is also more difficult to manage the initial value.

### Annotate the Setter

We can use the `@Autowired` annotation on the setter of the field. This way has
the same issues as the field annotation approach. It also is more difficult to
read; therefore, it is not used in production.

```java
@Component
public class Human {
    private String name;

    private Dog dog;

    @Autowired
    public void setDog(Dog dog) {
        this.dog = dog;
    }

  // other getters and setters
}
```

## Conclusion

We have learned how to wire beans in this lesson through a design called
dependency injection. Spring makes it easy to link beans by automatically
injecting dependencies. For most of our application, we will use the `@Autowired`
annotation with the constructor, which is what we saw in the Spring MVC lessons.

## References

- [Benefits of Dependency Injection](https://betterprogramming.pub/the-6-benefits-of-dependency-injection-7802b207ec69)
- [Autowired Annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Autowired.html)
