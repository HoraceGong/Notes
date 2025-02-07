---
title: Java反射与动态代理笔记
date: 2025-02-05 22:56:34
tags: [Java]
---

Java反射与动态代理笔记

<!--more-->

# 反射

反射允许对成员变量，成员方法和构造方法的信息进行编程访问。



## 获取class对象的三种方式

- Class.forName("全类名"); //最为常用的
- 类名.class  //一般更多的是当做参数进行传递
- 对象.getClass();  //当有了这个类的对象时，才可以使用

```java
public class ReflectionDemo1 {
  public static void main(String[] args) throws ClassNotFoundException {
    Class clazz = Class.forName("Reflection.Student");
    System.out.println(clazz);

    Class clazz2 = Student.class;
    System.out.println(clazz2);

    Student s = new Student();
    Class clazz3 = s.getClass();
    System.out.println(clazz3);

    System.out.println(clazz == clazz2 && clazz2 == clazz3);
  }
}
```



## 反射获取构造方法

### Class类中用于获取构造方法的方法

- Constructor<?>[] getConstructors()：返回所有public构造方法对象的数组
- Constructor<?>[] getDeclaredConstructors()：返回所有构造方法对象的数组（包括public和private）
- Constructor<T> getConstructor(Class<?> ...parameterTypes)：返回单个public构造方法对象
- Constructor<T> getDeclaredConstructor(Class<?> ...parameterTypes)：返回单个构造方法对象（包括public和private）



### Constructor类中用于创建对象的方法

- T newInstance(Object... initargs)：根据指定的构造方法创建对象
- setAccessible(boolean flag)：设置为true，表示取消访问检查



```java
public class ReflectionDemo2 {
  public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException {
    Class clazz = Class.forName("Reflection.Student");

    //获取构造方法
    Constructor[] cons = clazz.getConstructors();
    for (Constructor con : cons) {
      System.out.println(con);
    }

    Constructor[] declaredCons = clazz.getDeclaredConstructors();
    for (Constructor declaredCon : declaredCons) {
      System.out.println(declaredCon);
    }

    Constructor con1 = clazz.getDeclaredConstructor();
    System.out.println(con1);

    Constructor con2 = clazz.getDeclaredConstructor(String.class, int.class);
    System.out.println(con2);

  }
}
```



## 反射获取成员变量

### Class类中用于获取成员变量的方法

- Field[] getFields()：返回所有public变量对象的数组
- Field[] getDeclaredFields()：返回所有变量对象的数组（包括public和private）
- Field getField(String name)：返回单个public变量对象
- Field getDeclaredField(String name)：返回单个变量对象（包括public和private）



### Field类中用于创建对象的方法

- void set(Object obj, Object value)：赋值
- Object get(Object obj)：获取值



## 反射获取成员方法

### Class类中用于获取成员方法的方法

- Method[] getMethods()：返回所有public成员方法对象的数组，包括继承的
- Method[] getDeclaredMethods()：返回所有构造方法对象的数组（包括public和private），不包括继承的
- Method getMethod(String name, Class<?> ...parameterTypes)：返回单个public成员方法对象
- Method getDeclaredMethod(String name, Class<?> ...parameterTypes)：返回单个成员方法对象（包括public和private）



### Method类中用于创建对象的方法

- Object invoke(Object obj, Object... args)：运行方法
- 参数一：用obj对象调用该方法
- 参数二：调用方法的传递的参数（如果没有就不写）
- 返回值：方法的返回值（如果没有就不写）



# 动态代理

特点：无侵入式地给代码添加额外的功能。

## 如何为Java对象创建一个代理对象

- **Java.lang.reflect.Proxy**类：提供了为对象产生代理对象的方法。

```java
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h);
//参数一：用于指定用哪个类加载器，去加载生成的代理类
//参数二：指定接口，这些接口用于指定生成的代理长什么样，也就是有哪些方法
//参数三：用来指定生成的代理对象要干什么事情
```

```java
/**
 * 类的作用：创建一个代理
 */
public class ProxyUtil {

  /**
     * 方法的作用：给一个Star的对象创建一个代理
     * 形参：被代理的BigStar对象
     * 返回值：给BigStar创建的代理
     */
  public static Star createProxy(BigStar bigStar){
    Star star = (Star) Proxy.newProxyInstance(
      ProxyUtil.class.getClassLoader(),//参数一：用于指定用哪个类加载器，去加载生成的代理类
      new Class[]{Star.class},//参数二：指定接口，这些接口用于指定生成的代理长什么样，也就是有哪些方法
      new InvocationHandler() {//参数三：用来指定生成的代理对象要干什么事情
        /**
                     * @param proxy 代理的对象
                     * @param method 要运行的方法sing
                     * @param args 调用sing方法时，传递的实参
                     */
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
          if ("sing".equals(method.getName())) {
            System.out.println("准备话筒，收钱");
          } else if ("dance".equals(method.getName())) {
            System.out.println("准备场地，收钱");
          }

          //去找大明星开始唱歌或者跳舞
          //代码的表现形式：调用大明星里面唱歌或跳舞的方法
          return method.invoke(bigStar, args);
        }
      });

    return star;
  }
}
```

```java
public class Test {
  public static void main(String[] args) {
    //找大明星唱一首歌
    //1. 获取代理的对象
    Star proxy = ProxyUtil.createProxy(new BigStar("鸡哥"));
    //2. 调用唱歌的方法
    String result = proxy.sing("只因你太美");
    System.out.println(result);
    //3. 调用跳舞的方法
    proxy.dance();
  }
}
```



