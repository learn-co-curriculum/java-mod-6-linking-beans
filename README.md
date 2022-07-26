# Linking Beans

## Learning Goals

- Link beans using methods in the config class.
- Use the `@Autowired` annotation to create bean links.

## Introduction

We will learn how to create a link between two beans in this lesson. Let’s say
we have a `Human` instance in our Spring context along with the `Dog` instance.
We connect them together such that the `Human` instance “has a” `Dog`.

![Spring context with a human and a dog bean with a link between them](https://curriculum-content.s3.amazonaws.com/java-spring-1/spring-context-human-dog-linked.png)

Create `Human` class in the `main` package before proceeding forward:

```java
package main;

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

1. Define the relationship in the configuration class.
2. Use the `@Autowired` annotation to inject beans.

## Define Relationship in the Configuration Class

We need to define the methods for creating the `Human` and `Dog` beans before we
can create a relationship between them.

Open the `MyConfig` class and add the following code:

```java
package config;

import main.Dog;
import main.Human;
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

Then open the `Main` class and add the following code:

```java
package main;

import config.MyConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
    public static void main(String[] args) {
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

Running the `main` method of the `Main` class adds the beans to the Spring
context but does not define any relationship between the beans as indicated by
the `dog=null` value of the `Human` instance. This is what the Spring context
looks like right now:

![Spring context with a Human and a Dog bean without any links](https://curriculum-content.s3.amazonaws.com/java-spring-1/spring-context-human-dog-separate.png)

And this is what we want to make it look like:

![Diagram showing the change from a Spring context with unlinked beans to a Spring context with the beans linked](https://curriculum-content.s3.amazonaws.com/java-spring-1/spring-context-human-dog-separate-to-linked.png)

We can define the relationship in two ways:

1. Calling a bean method inside another bean method and linking the reference.
2. Defining parameters to auto inject the dependency

### **Call a Bean Method Within Another Bean Method**

We will be calling the `dog()` method in the `human()` method and setting the
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

Now if we run the `main` method of the `Main` class, we will get the following
output:

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

### Define Bean Method Parameter

We can also define a `Dog` parameter for the `human()` method which will cause
Spring to automatically reference the `Dog` bean in its context and assign it to
the `Human` bean’s `dog` property.

Open the `MyConfig` file and update it like so:

```java
package config;

import main.Dog;
import main.Human;
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
    public Human human(Dog dog) {
        Human h = new Human();
        h.setName("Lily");
        h.setDog(dog);
        return h;
    }
}
```

Now run the `main` method again and it should give the following output:

```java
Human{name='Lily', dog=Dog{name='Luna'}}
Dog{name='Luna'}
```

Spring performs the following steps here:

1. Calls the `dog()` method and stores the returned instance in its context.
2. Calls the `human()` method. It sees that the method requires a `Dog` instance
   so it checks its context for one and _injects_ the `Dog` instance into the
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

package config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages = "main")
public class MyConfig {

}
```

```java
// Dog.java

package main;

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

package main;

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
// Main.java

package main;

import config.MyConfig;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
    public static void main(String[] args) {
        // create context
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);

        // access beans
        Dog luna = context.getBean(Dog.class);
        luna.setName("Luna");
        Human lily = context.getBean(Human.class);
        lily.setName("Lily");

        System.out.println(lily);
        System.out.println(luna);
    }
}
```

### Annotate Class Constructor

We have to create a constructor in the `Human` class that takes a `Dog` instance
as an argument and assigns it to the `dog` field in the class. Then we annotate
the constructor with `@Autowired`.

```java
@Component
public class Human {
    private String name;
    private final Dog dog;

    @Autowired
    public Human(Dog dog) {
        this.dog = dog;
    }

		// getters, setters, and toString
		// no setter for "dog" since it's a final field
}
```

If we run the `main` method we will get the following output:

```java
Human{name='Lily', dog=Dog{name='Luna'}}
Dog{name='Luna'}
```

When Spring looks at the constructor it searches for a `Dog` bean in its scope
and invokes the constructor with the `Dog` bean. Notice that we have also set
the `dog` field to `final` since the value is injected during object creation.
This is the most common way we will be using the `@Autowired` annotation.

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
approach is not preferred because we can’t set the `dog` to be `final` which
would allow anyone to change the value. It is also more difficult to manage the
initial value.

### Annotate the Setter

We can use the `@Autowired` annotation on the setter of the field. This approach
has the same issues as the field annotation method while also being more
difficult to read which is why it is not used in production.

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

We have learned how to create links between beans in this lesson. Spring makes
it easy to link beans by automatically injecting dependencies. For most of our
applications we will use the `@Autowired` annotation with the constructor.
