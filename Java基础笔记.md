# Java String

String被声明为final，因此它不可被继承。

内部使用char数组存储数据，且**该数组被声明为`final`**，这意味着value数组初始化之后就不能再引用其他数组。并且**String内部没有改变value数组的方法**，因此可以保证String不可变。

## 不可变的好处

- **可以缓存hash值**：因为String的hash值经常被使用（比如String用作HashMap的key）。不可变的特性可以使得hash值也不会变，因此只需要进行一次计算。
- **String Pool的需要**：如果一个String对象已经被创建过了，那么就会从String Pool中取得引用。只有String是不可变的，才可能使用String Pool。
- **安全性**：String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。
- **线程安全**：String不可变性天生具备线程安全特性，可以在多个线程中安全地使用。

## String, StringBufffer, StringBuilder

1. 可变性

- String不可变
- StringBuffer, StringBuilder可变

2. 线程安全

- String不可变，因此是线程安全的
- StringBuilder不是线程安全的
- StringBuffer是线程安全的，内部使用`synchronized`进行同步

## String.intern()

使用`String.intern()`可以保证相同内容的字符串变量引用同一的内存对象。

下面示例中，s1 和 s2 采用 new String() 的方式新建了两个不同对象，而 s3 是通过 s1.intern() 方法取得一个对象引用。intern() 首先把 s1 引用的对象放到 String Pool(字符串常量池)中，然后返回这个对象引用。因此 s3 和 s1 引用的是同一个字符串常量池的对象。

```java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
String s3 = s1.intern();
System.out.println(s1.intern() == s3);  // true
```

如果是采用`"bbb"`这种使用双引号的形式创建字符串实例，会自动地将新建的对象放入String Pool中。

```java
String s4 = "bbb";
String s5 = "bbb";
System.out.println(s4 == s5); //true
```

