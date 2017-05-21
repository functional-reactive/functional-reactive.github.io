## Java Core Classes
Here we are collectiong all the things that we need
more often during the examples. All this we are 
missing in Core Java ;-)

### Pair , Tripel, Quad and Quint
From time to time we want to have classes like a Tuple or even more.
Here are some of them up to Quint. Hopefully we could remove them soon
and replace it with Core JDK Classes.

```java
public class Pair<T1, T2> {
  private T1 t1;
  private T2 t2;

  public Pair(final T1 t1, final T2 t2) {
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
    return "Pair{" +
        "t1=" + t1 +
        ", t2=" + t2 +
        '}';
  }

  @Override
  public boolean equals(final Object o) {
    if (this == o) return true;
    if (!(o instanceof Pair)) return false;
    final Pair<?, ?> pair = (Pair<?, ?>) o;
    return Objects.equals(t1, pair.t1) &&
        Objects.equals(t2, pair.t2);
  }

  @Override
  public int hashCode() {
    return Objects.hash(t1, t2);
  }
}
```
All classes are build up like the Pair. 
