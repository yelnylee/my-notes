# List Interface API

* Query API

  ```java
  boolean contains(Object a);
  boolean containsAll(Collection<?> c);
  E get(int index);
  int indexof(Object o);
  int lastIndexOf(Object o);
  ```

* Insert API

  ```java
  boolean add(E a);
  boolean addAll(Collection<? extends E> c);
  boolean addAll(int index,Collection<? extends E> c);
  boolean replaceAll(UnaryOperator<E> operator);
  E set(int index,E element);
  List<E> subList(int fromIndex,int toIndex);
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

  ```java
  Iterator<E> iterator();
  Spliterator<E> spliterator(); //源码
  Stream<E> stream(); //源码
  Stream<E> parallelStream();//源码
  ListIterator<E> listIterator();
  ListIterator<E> listIterator(int index);
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

* 排序API

  ```java
  void sort(Comparator<? super E> c);
  ```

  