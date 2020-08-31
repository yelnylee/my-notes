# Collection Interface API

   Java8及以后版本中支持了Stream<E> stream() 和Stream<E> parallelStream()接口，用于从集合中获取各种streams.

* Query API

  ```java
  boolean contains(Object a);
  boolean containsAll(Collection<?> c);
  ```

* Insert API

  ```java
  boolean add(E a);
  boolean addAll(Collection<? extends E> c);
  ```

* Remove API

  ```java
  boolean remove(Object o);
  boolean removeAll(Collection<?> c);
  boolean removeIf(Predicate<? super E> fiter);
  boolean retainAll(Collection<?> c);
  void clear();
  ```

* 迭代器 API

  支持三种方式遍历集合：1）使用流式操作 2）for-each 结构 3）Iterator

  ```java
  Iterator<E> iterator();
  Spliterator<E> spliterator(); //源码
  Stream<E> stream(); //源码
  Stream<E> parallelStream();//源码
  ```

* Property API

  ```java
  int size();
  boolean isEmpty();
  boolean equals();
  int hashCode();
  ```

* 转换数组

  ```java
  Object[] toArray();
  T[] toArray(T [] a);
  ```

  