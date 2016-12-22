## Functional - Java

### Memoizing Functions
A Function is memoizing if the function will ever give the same result for the same input.
And here we mean, exactly the same result. Let´s say the result will be an instance of a car.
The result for the same input would give back the instance of the car. Not only an equal one.

But let´s start with the beginning.

AS starting point I will use the following function. 

```java
Function<Integer, Integer> squareFunction = x -> {
    System.out.println("In function");
    return x * x;
  };
```
Here I am using the System.out only to show on command line how often this method was invoked. 
I know that this is nothing for production!

If I call this function twice with the same input, the System.out would be on screen two times. So let´s define 
a memoized function.

```java
Function<Integer, Integer> memoizationFunction = Memoizer.memoize(squareFunction);
```

Now I want to have the System.out only on time on screen, expacting that the result was calculated one time.
What we have to do, to get the result?
This work is based on the orig article I found 
on [DZONE - Java8 Automatic Memoization](https://dzone.com/articles/java-8-automatic-memoization)

The solution here is based on the method (class Map) introduced with Java8.
Since Java8 you could provide a Lambda to calculate a value corresponding to a key, if the value is not available.
This means, if you request a key/value pair the first time, the lambda will be executed to create the value.
At the same time it ist stored in the map. For our solution the ``ConcurrentHashMap``` is used.
 
```java
public class Memoizer<T, U> {
  private final Map<T, U> memoizationCache = new ConcurrentHashMap<>();

  private Function<T, U> doMemoize(final Function<T, U> function) {
    return input -> memoizationCache.computeIfAbsent(input, function);
  }

  public static <T, U> Function<T, U> memoize(final Function<T, U> function) {
    return new Memoizer<T, U>().doMemoize(function);
  }
}
```
And now we have the first version, but only for ```Function<T,U>```.
But what could we do with a ```BiFunction<T1,T2, R>```?

Let´s play around with the Function itself. We could transform am ```BiFunction<T1,T2,R>``` 

```java
BiFunction<Integer,Integer,Integer> biFunction = (x,y) -> x * y;
```
into a ```Function<T1,Function<T2,R>>```.
```java
Function<Integer, Function<Integer, Integer>> biFunction = x -> y -> x * y;
```
With this we are able to build a memoized function again. This would end up in the first version like the following.

```java
Function<Integer, Function<Integer,Integer>> memoizationFunction
      = Memoizer.memoize(x -> Memoizer.memoize(y -> x * y));
```
So far so good, but the usage itself is not nice. First is the transformation into the Function of a Function instead of using the 
original BiFunction, and the second is the usage of the memoized Function itself.

```java
  public static void main(String[] args) {
    System.out.println("memoizationFunction = " + memoizationFunction.apply(2).apply(3));
    System.out.println("memoizationFunction = " + memoizationFunction.apply(2).apply(3));
  }
```
Now we could use the orig BiFunction inside the memoized Function.

```java
BiFunction<Integer,Integer,Integer> mul = (x, y) -> x*y;

Function<Integer, Function<Integer,Integer>> memoizationFunction
      = Memoizer.memoize(
          x -> Memoizer.memoize(
              y -> mul.apply(x,y)));
```

With this we could make it a little bit more comfortable and provide a create method.
The biggest change here is the introduction of a ```Supplier<Integer>```.
So we changed the BiFunction into ```BiFunction<Integer, Integer, Supplier<Integer>>```.
Now we could use the method create with a lambda expression as argument to get a memoized Function.
But still the result is ```Function<Integer, Function<Integer, Integer>>```

```java
  public static Function<Integer, Function<Integer, Integer>> create(
      BiFunction<Integer, Integer, Supplier<Integer>> biFuncSupplier) {
    return Memoizer.memoize(x -> Memoizer.memoize(y -> biFuncSupplier.apply(x, y).get()));
  }

  public static void main(String[] args) {
    final Function<Integer, Function<Integer, Integer>> function = create((x, y) -> () -> {
      System.out.println("execute x/y = " + x + " / " + y);
      return x * y;
    });
    System.out.println("memoizationFunction = " + function.apply(2).apply(3));
    System.out.println("memoizationFunction = " + function.apply(2).apply(3));
  }
```

But we don´t want to transform the orig BiFunction. The transformation could be done 
in a generic way. 
 
```java
  public static Function<Integer, Function<Integer, Integer>> transform(
      final BiFunction<Integer, Integer, Integer> biFunc) {
    return create((x, y) -> () -> biFunc.apply(x, y));
  }

  private static Function<Integer, Function<Integer, Integer>> create(
      BiFunction<Integer, Integer, Supplier<Integer>> biFuncSupplier) {
    return Memoizer.memoize(
        x -> Memoizer.memoize(
            y -> biFuncSupplier.apply(x, y).get()));
  }

  public static void main(String[] args) {
    final Function<Integer, Function<Integer, Integer>> function = transform((x, y) -> {
      System.out.println("execute x/y = " + x + " / " + y);
      return x * y;
    });
    System.out.println("memoizationFunction = " + function.apply(2).apply(3));
    System.out.println("memoizationFunction = " + function.apply(2).apply(3));
  }
```
Or if you want to remove the types with generics....

```java
  public static <T1, T2, R> Function<T1, Function<T2, R>> transform(
      final BiFunction<T1, T2, R> biFunc) {
    return create((x, y) -> () -> biFunc.apply(x, y));
  }

  private static <T1, T2, R> Function<T1, Function<T2, R>> create(
      BiFunction<T1, T2, Supplier<R>> biFuncSupplier) {
    return Memoizer.memoize(x -> Memoizer.memoize(y -> biFuncSupplier.apply(x, y).get()));
  }

  public static void main(String[] args) {
    final Function<Integer, Function<Integer, Integer>> function = transform((x, y) -> {
      System.out.println("execute x/y = " + x + " / " + y);
      return x * y;
    });
    System.out.println("memoizationFunction = " + function.apply(2).apply(3));
    System.out.println("memoizationFunction = " + function.apply(2).apply(3));
  }
```

Now we are able to transform the ```BiFunction<T1, T2, R>```, but the result is a ```Function<T1, Function<T2, R>>```
This is not nice to use, so we want to transform it back to a ```BiFunction<T1, T2, R>```.

```java
  public static <T1, T2, R> BiFunction<T1, T2, R> prepare(
      final Function<T1, Function<T2, R>> transformed) {
    return (x, y) -> transformed.apply(x).apply(y);
  }
```

Finally we are able to transform in both directions.
```BiFunction<T1, T2, R>``` -> ```Function<T1, Function<T2, R>>``` -> ```BiFunction<T1, T2, R>```.

All together is now 

```java
public static <T1, T2, R> BiFunction<T1, T2, R> prepare(
      final Function<T1, Function<T2, R>> transformed) {
    return (x, y) -> transformed.apply(x).apply(y);
  }

  public static <T1, T2, R> Function<T1, Function<T2, R>> transform(
      final BiFunction<T1, T2, R> biFunc) {
    return create((x, y) -> () -> biFunc.apply(x, y));
  }

  private static <T1, T2, R> Function<T1, Function<T2, R>> create(
      BiFunction<T1, T2, Supplier<R>> biFuncSupplier) {
    return Memoizer.memoize(
        x -> Memoizer.memoize(
            y -> biFuncSupplier.apply(x, y).get()));
  }

  public static void main(String[] args) {
    final Function<Integer, Function<Integer, Integer>> function = transform((x, y) -> {
      System.out.println("execute x/y = " + x + " / " + y);
      return x * y;
    });

    System.out.println("memoizationFunction = " + prepare(function).apply(2,3));
    System.out.println("memoizationFunction = " + prepare(function).apply(2,3));
  }
```

After we found out how to transform in small steps, we are able to merge all together in a smaller 
method. The result looks like the following.

```java
  public static <T1, T2, R> BiFunction<T1, T2, R> memoize(final BiFunction<T1, T2, R> biFunc) {
    final BiFunction<T1, T2, Supplier<R>> biFuncSupplier = (x, y) -> () -> biFunc.apply(x, y);
    final Function<T1, Function<T2, R>> transformed 
           = Memoizer.memoize(
               x -> Memoizer.memoize(
                   y -> biFuncSupplier.apply(x, y).get()));
    return (x, y) -> transformed.apply(x).apply(y);
  }

  public static void main(String[] args) {
    final BiFunction<Integer, Integer, Integer> function = memoize((x, y) -> {
      System.out.println("execute x/y = " + x + " / " + y);
      return x * y;
    });

    System.out.println("memoizationFunction = " + function.apply(2,3));
    System.out.println("memoizationFunction = " + function.apply(2,3));
  }
```

The step now to a NFunction is not so far..  let´s see how we could do it with 
three params. The first thing we have to create is a ```TriFunction<T1,T2,T3,R>```, because it is not part of the JDK until now.

```java
  @FunctionalInterface
  public interface TriFunction<T1, T2,T3, R> {
    R apply(T1 t1, T2 t2, T3 t3);

    default <V> TriFunction<T1, T2,T3, V> andThen(Function<? super R, ? extends V> after) {
      Objects.requireNonNull(after);
      return (T1 t1, T2 t2, T3 t3) -> after.apply(apply(t1, t2, t3));
    }
  }
```

And finally the memoize looks like....

```java
  public static <T1, T2,T3, R> TriFunction<T1, T2,T3, R> memoize(final TriFunction<T1, T2,T3, R> threeFunc) {
    final TriFunction<T1, T2,T3, Supplier<R>> threeFuncSupplier 
             = (x, y, z) -> () -> threeFunc.apply(x, y,z);
    final Function<T1, Function<T2, Function<T3, R>>> transformed
        = Memoizer.memoize(
            x -> Memoizer.memoize(
                y -> Memoizer.memoize(
                    z -> threeFuncSupplier.apply(x, y,z).get())));
    return (x, y, z) -> transformed.apply(x).apply(y).apply(z);
  }

  public static void main(String[] args) {
    final TriFunction<Integer, Integer, Integer, Integer> function = memoize((x, y, z) -> {
      System.out.println("execute x/y/z = " + x + " / " + y + " / " + z);
      return x * y * z;
    });

    System.out.println("memoizationFunction = " + function.apply(2,3,-1));
    System.out.println("memoizationFunction = " + function.apply(2,3,-1));
  }
```

So, now we have everything for a NFunction together ;-)
