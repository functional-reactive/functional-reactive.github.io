## Java Core Basics
If you start with the functional way of coding, you will start 
searching for the best place to define the functions.
How to create them and some other practical things.
Here I will describe the way I would do it.. And what kind of language 
features I am using for this.

### Interfaces
Interfaces are a good point to start with stateless code pieces.
But lets start with the Java8 features we got and we could use.

#### Inheritance and default implementations
Since Java8 we have default implementations. This is good to define 
Interfaces with more than one method, but providing for n-1 methods an
implementation. So we got a Functional Interface and we could use now Lambdas with this.
But I don´t want to go too deep into the details. There is a lot of good documentation 
available on the market. 

#### public static methods
Here we could start now with the functional aspects. Ok not with the functional aspects 
itself, but with the way to create instances of functions.

Let´s see why.
If we are defining Functions, we have to type them.

```java
Function<String, String> func = (str) -> str + "-something usefull";
```
Now we could use this, but how we could define Functions with Generics?
We start now with the task, how to convert a Enumeration into a Stream, 
or more precise, an Enumeration<T> to a Stream<T>?

For this we need a Function like the follwing.
```java
Function<Enumeration<T>, Stream<T>> convert = (T t) -> ....
```
But this you are not able to do. So for this we need a Method that will create an instance of an 
function. And for this, we should use static methods in interfaces.

```java
  static <T> Function<Enumeration<T>, Stream<T>> enumToStream() {
    return (e) ->
        StreamSupport
            .stream(Spliterators.spliteratorUnknownSize(new Iterator<T>() {
              public T next() {
                return e.nextElement();
              }

              public boolean hasNext() {
                return e.hasMoreElements();
              }
            }, Spliterator.ORDERED), false);
  }
```





#### public static attributes
Yes, you could define static attibutes in interfaces. But it is a good advise 
to use them only for things that are not changing. Maybe a const String, 
or a defined NULL-Object for a specific class you have to deal with.

#### Java9 JEP213 - private methods in interfaces


### How you could combine old and new
In jab´a we have different ways to connect to the old Java World.
Here I want to describe a way, that not so often is used or 
maybe known by the developers.

Let´s say we have a LegacyService and it is not allowed to change anything. 
(or maybe it is a third party lib and you can not change anything)

From an instance of this LegacyService you could grap a Methodreference.

```java
final BiFunction<String, Integer, String> myFunction = legacyService::doWork;
```
Oh, what is this? No cast? Yes, no explicit cast.. And LegacyService is 
not implementing the Interface BiFunction. But....

What we are using here, is the mechanism "invoke dynamic". What must fit, is the 
amount and type of params and return type. If this match, you could do it. 
And it is not limited to the Functional Interfaces from the JDK. So if you have a
legacy method with four params, create a QuadFunction. And have in mind, 
return type could be Void.

```java
  public class LegacyService {
    public String doWork(String str, Integer integer) {
      return "legacy-" + str + " - " + integer;
    }
  }

  public static void main(String[] args) {
    final LegacyService legacyService = new LegacyService();
    final BiFunction<String, Integer, String> myFunction = legacyService::doWork;
    final Function<String, String> function = input -> "nextFunction - " + input;
    final BiFunction<String, Integer, String> andThen = myFunction.andThen(function);
    String hello = andThen.apply("Hello", 42);
    System.out.println("hello = " + hello);
  }
```

Now we could combine legacy code with new functions.