- [String](#string)
  - [不可变](#不可变)
  - [string](#string-1)
  - [StringBuilder](#stringbuilder)
    - [StringBuild 的使用](#stringbuild-的使用)

# String

String 被声明为 final，因此它不可被继承。

内部使用 char 数组存储数据，该数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

```Java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
```

## 不可变

1、**可以缓存 hash 值**

因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

**2. String Pool 的需要**

如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

**3. 安全性**

String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。

**4. 线程安全**

String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

## string
String 被声明为 final，因此它不可被继承。内部使用 char 数组存储数据，该数组被声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组。并且 String 内部没有改变 value 数组的方法，因此可以保证 String 不可变。

**只有intern会去pool中取**

```Java
String s1 = new String("aaa");
String s2 = new String("aaa");
System.out.println(s1 == s2);           // false
# intern() 首先把 s1 引用的对象放到 String Pool(字符串常量池)中，然后返回这个对象引用。因此 s3 和 s1 引用的是同一个字符串常量池的对象。
String s3 = s1.intern();
System.out.println(s1.intern() == s3);  // true
# 自动放入String Pool
String s4 = "bbb";
String s5 = "bbb";
System.out.println(s4 == s5);  // true

String a = "aaa";
String b = a.replace("a", "b"); // 不会对a的值产生影响
System.out.println(a == b); // false，b是一个新创建的字符串对象
System.out.println(a); // 此时a还是"aaa"而不是"bbb"

String c = a.concat("cc");
System.out.println(c); // aaacc
System.out.println(a == c); // false，c是一个新创建的字符串对象
System.out.println(a); // 此时a还是"aaa"而不是"aaacc"

String d = a.substring(2);
System.out.println(d); // a
System.out.println(a == d); // false，d是一个新创建的字符串对象
System.out.println(a); // 此时a还是"aaa"而不是"a"
```

## StringBuilder

StringBuilder 类是非线程安全的 StringBuffer 类。即：

实际应用时，对字符串追加内容的操作几乎都是在一个线程中进行的（例如最开始的循环向字符串追加内容的代码），这样如果用 StringBuffer 的话，就会额外有给 StringBuffer 对象加锁的开销。

```java
public final class StringBuilder extends AbstractStringBuilder
    implements java.io.Serializable, CharSequence
{
    // ...

    public StringBuilder append(String str) {
        super.append(str);
        return this;
    }

    public String toString() {
        // Create a copy, don't share the array
        return new String(value, 0, count);
    }

    public StringBuilder reverse() {
        super.reverse();
        return this;
    }

    public AbstractStringBuilder reverse() {
        int n = count - 1; // 字符序列的最后一个字符的索引
        // 遍历字符串的前半部分
        for (int j = (n-1) >> 1; j >= 0; j--) {
            int k = n - j; // 计算相对于 j 对称的字符的索引
            char cj = value[j]; // 获取当前位置的字符
            char ck = value[k]; // 获取对称位置的字符
            value[j] = ck; // 交换字符
            value[k] = cj; // 交换字符
        }
        return this; // 返回反转后的字符串构建器对象
    }

}
```


```Java
// 在单线程环境下进行字符串拼接
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append(i);
}
System.out.println(sb.toString());
```

### StringBuild 的使用

```java
new String("二哥") + new String("三妹") 
==
new StringBuilder().append("二哥").append("三妹").toString();



## StringBuffer

StringBuffer 类对外暴露了可以修改其值的 append、insert、delete 等方法。一个 StringBuffer 对象在其缓冲区（一个字符数组 char[]）的容量足够的情况下，调用这些方法可以直接修改 StringBuffer 的值而不必创建新的对象。（在一个StringBuffer  的缓冲区的容量不足的时候，调用其 append 或者 insert 就会使 StringBuffer 创建一个新的更大的缓冲区，这时则会创建一个新的字符数组 char[] 对象。）

- 线程安全
- StringBuffer 在 insert、append、delete 这些 public 方法的定义处加了synchronized关键字

```Java
public final class StringBuffer extends AbstractStringBuilder implements Serializable, CharSequence {

    public StringBuffer() {
        super(16);
    }
    
    public synchronized StringBuffer append(String str) {
        super.append(str);
        return this;
    }

    public synchronized String toString() {
        return new String(value, 0, count);
    }

    // 其他方法
}
```

```Java
// 在多线程环境下进行字符串拼接
StringBuffer sb = new StringBuffer();
Runnable r = new Runnable() {
    public void run() {
        sb.append(Thread.currentThread().getName());
    }
};
Thread t1 = new Thread(r, "A");
Thread t2 = new Thread(r, "B");
Thread t3 = new Thread(r, "C");
t1.start();
t2.start();
t3.start();
t1.join();
t2.join();
t3.join();
System.out.println(sb.toString()); // 输出"ABC"
```