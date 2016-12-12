## Java Core Classes
Here we are collectiong all the things that we need
more often during the examples. All this we are 
missing in Core Java ;-)

### Tupel , Tripel, Quad and Quint
From time to time we want to have classes like a Tuple or even more.
Here are some of them up to Quint. Hopefully we could remove them soon
and replace it with Core JDK Classes.

```java
public class Tuple<T1, T2> {
  private T1 t1;
  private T2 t2;

  public Tuple(final T1 t1, final T2 t2) {
    this.t1 = t1;
    this.t2 = t2;
  }

  public T1 getT1() {
    return t1;
  }

  public T2 getT2() {
    return t2;
  }

  @Override
  public String toString() {
    return "Tuple{" +
        "t1=" + t1 +
        ", t2=" + t2 +
        '}';
  }

  @Override
  public boolean equals(final Object o) {
    if (this == o) return true;
    if (!(o instanceof Tuple)) return false;
    final Tuple<?, ?> tuple = (Tuple<?, ?>) o;
    return Objects.equals(t1, tuple.t1) &&
        Objects.equals(t2, tuple.t2);
  }

  @Override
  public int hashCode() {
    return Objects.hash(t1, t2);
  }
}
```
All classes are build up like the Tuple. 
The sourcecode you could find on GitHub [Functional-Reactive-Workshop - Core](https://github.com/functional-reactive/workshop_functional-reactive/tree/master/src/org/rapidpm/workshop/frp/core/model)
