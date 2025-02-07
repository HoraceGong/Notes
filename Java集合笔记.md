---
title: Java集合笔记
date: 2025-02-05 12:09:41
tags: [Java]
---

有关Java集合的基础、API与底层实现。

<!--more-->

# 双列集合

## 双列集合的特点

1. 一次需要存储一对数据，包括键和值。
2. 键不能重复，但是值可以。
3. 键和值是一一对应的关系，每一个键只能找到自己对应的值。
4. 键+值这个整体被称为键值对，或者叫做键值对对象，**在Java中叫做Entry对象**。



## Map集合常用API

Map是双列集合的顶层接口，它的功能是所有双列集合都可以继承使用的。

| 方法名称                            | 说明                     |
| ----------------------------------- | ------------------------ |
| V put(K key, V value)               | 添加元素                 |
| V remove(Object key)                | 根据键删除键值对元素     |
| void clear()                        | 移除所有的键值对元素     |
| boolean containsKey(Object key)     | 判断集合是否包含指定的键 |
| boolean containsValue(Object value) | 判断集合是否包含指定的值 |
| boolean isEmpty()                   | 判断集合是否为空         |
| int size()                          | 返回集合中键值对的个数   |



## Map集合的遍历方式

### 方法一：键找值

先获取所有的键，再根据所有的键获取全部的值。

```Java
public static void main(String[] args) {
  Map<String, String> map = new HashMap<String, String>();

  map.put("a", "A");
  map.put("b", "B");
  map.put("c", "C");

  //通过keySet()方法获取所有key的集合
  Set<String> set = map.keySet();

  for (String key : set) {
    System.out.println(key + " : " + map.get(key));
  }
}
```



### 方法二：获取全部键值对对象进行遍历

```java
public static void main(String[] args) {
  Map<String, String> map = new HashMap<>();

  map.put("a", "A");
  map.put("b", "B");
  map.put("c", "C");

  //通过entrySet()来获取Map中全部的entry对象
  for (Map.Entry<String, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ":" + entry.getValue());
  }
}
```



### 方法三：使用lambda表达式进行遍历

```java
public static void main(String[] args) {
  Map<String, String> map = new HashMap<>();

  map.put("a", "A");
  map.put("b", "B");
  map.put("c", "C");

  map.forEach(new BiConsumer<String, String>(){
    @Override
    public void accept(String key, String value) {
      System.out.println(key + " : " + value);
    }
  });

  System.out.println("---------------------");

  map.forEach((String key, String value) -> {
    System.out.println(key + " : " + value);
  }
             );

  System.out.println("---------------------");

  map.forEach((key, value) -> System.out.println(key + " : " + value));
}
```



## HashMap

### HashMap的特点

- HashMap底层是哈希表结构的
- 依赖hashCode方法和equals方法保证键的唯一
- **如果键存储的是自定义对象，需要重写hashCode方法和equals方法**；如果值存储的是自定义对象则不需要



### 在对象类中重写hashCode()和equals()

```java
public class Student {
  public String name;
  public int age;

  //此处省略getter setter和toString
  @Override
  public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Student student = (Student) o;
    return age == student.age && Objects.equals(name, student.name);
  }

  @Override
  public int hashCode() {
    return Objects.hash(name, age);
  }
}
```



## LinkedHashMap

- 由键决定：有序、不重复、无索引。
- **这里的有序指的是保证存储和取出的元素顺序一致**。
- 原理：底层**依旧是哈希表**，只是每个键值对元素又额外的多了一个双链表的机制记录存储的顺序。



## TreeMap

- TreeMap和TreeSet底层原理一样，都是红黑树结构的
- 由键决定特性：不重复、无索引、可排序
- 可排序：**对键进行排序**
- 注意：默认按照键的**从小到大**顺序进行排序，也可以自己制定键的排序规则



代码书写两种排序规则：

- 实现Comparable接口，制定比较规则
- 创建集合时传递Comparator比较器对象，制定比较规则

```java
public class Student implements Comparable<Student>{
  public String name;
  public int age;

  //返回值：
  //负数：表示当前要添加的元素是小的，存在左边
  //正数：表示当前要添加的元素是大的，存在右边
  //0：表示当前元素已经存在，舍弃
  @Override
  public int compareTo(Student o) {
    int i = this.getAge() - o.getAge();
    i = i == 0 ? this.getName().compareTo(o.getName()) : i;
    return i;
  }
}
```



# Collections

- java.util.Collections:集合工具类
- 作用：**Collections不是集合，而是集合的工具类**

| 方法名称                                        | 说明                               |
| ----------------------------------------------- | ---------------------------------- |
| boolean addAll(Collection<T> c, T ... elements) | 批量添加元素                       |
| void shuffle(List<?> list)                      | 打乱List集合元素的顺序             |
| void sort(List<T> list)                         | 排序                               |
| void sort(List<T> list, Comparator<T> c)        | 以指定的规则进行排序               |
| int binarySearch(List<T> list, T key)           | 以二分法查找元素                   |
| void copy(List<T> dest, List<T> src)            | 拷贝集合中的元素                   |
| int fill(List<T> list, T obj)                   | 使用指定的元素填充集合             |
| void max/min(Collection<T> coll)                | 根据默认的自然排序获取最大、最小值 |
| void swap(List<?> list, int i, int j)           | 交换集合中指定位置的元素           |
