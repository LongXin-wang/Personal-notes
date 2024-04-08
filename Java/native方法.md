- [JNI](#jni)


> https://javabetter.cn/oo/native-method.html#_2%E3%80%81%E7%94%A8-c-%E8%AF%AD%E8%A8%80%E7%BC%96%E5%86%99%E7%A8%8B%E5%BA%8F%E6%9C%AC%E5%9C%B0%E6%96%B9%E6%B3%95

# JNI

一般情况下，我们完全可以使用 Java 语言编写程序，但某些情况下，Java 可能满足不了需求，或者不能更好的满足需求，比如：

①、标准的 Java 类库不支持。
②、我们已经用另一种语言，比如说 C/C++ 编写了一个类库，如何用 Java 代码调用呢？
③、某些运行次数特别多的方法，为了加快性能，需要用更接近硬件的语言（比如汇编）编写。

Java Native Interface (JNI)标准就成为 Java 平台的一部分，它允许 Java 代码和其他语言编写的代码进行交互。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405222046830.png)