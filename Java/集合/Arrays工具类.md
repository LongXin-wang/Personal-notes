- [创建数组](#创建数组)
- [比较数组](#比较数组)
- [排序数组](#排序数组)
- [数组检索](#数组检索)
- [打印数组](#打印数组)
- [数组转换](#数组转换)

# 创建数组

使用 Arrays 类创建数组可以通过以下三个方法：

- copyOf，复制指定的数组，截取或用 null 填充
- copyOfRange，复制指定范围内的数组到一个新的数组
- fill，对数组进行填充

1) copyOf

```java
String[] intro = new String[] { "沉", "默", "王", "二" };
String[] revised = Arrays.copyOf(intro, 3);
String[] expanded = Arrays.copyOf(intro, 5);
System.out.println(Arrays.toString(revised));
System.out.println(Arrays.toString(expanded));

[沉, 默, 王]
[沉, 默, 王, 二, null]
```

2) copyOfRange

```java
String[] intro = new String[] { "沉", "默", "王", "二" };
String[] abridgement = Arrays.copyOfRange(intro, 0, 3);
System.out.println(Arrays.toString(abridgement));
```

3) fill

```java
String[] stutter = new String[4];
Arrays.fill(stutter, "沉默王二");
System.out.println(Arrays.toString(stutter));

[沉默王二, 沉默王二, 沉默王二, 沉默王二]
```

# 比较数组

使用 Arrays 类提供的 equals 方法比较两个数组是否相等。

```java
String[] intro = new String[] { "沉", "默", "王", "二" };
boolean result = Arrays.equals(new String[] { "沉", "默", "王", "二" }, intro);
System.out.println(result);
boolean result1 = Arrays.equals(new String[] { "沉", "默", "王", "三" }, intro);
System.out.println(result1);

true
false
```

```java
public static boolean equals(Object[] a, Object[] a2) {
    if (a==a2)
        return true;
    if (a==null || a2==null)
        return false;

    int length = a.length;
    if (a2.length != length)
        return false;

    for (int i=0; i<length; i++) {
        if (!Objects.equals(a[i], a2[i]))
            return false;
    }

    return true;
}
```

# 排序数组

```Java
String[] intro1 = new String[] { "chen", "mo", "wang", "er" };
String[] sorted = Arrays.copyOf(intro1, 4);
Arrays.sort(sorted);
System.out.println(Arrays.toString(sorted));
```

# 数组检索

```java
String[] intro1 = new String[] { "chen", "mo", "wang", "er" };
String[] sorted = Arrays.copyOf(intro1, 4);
Arrays.sort(sorted);
int exact = Arrays.binarySearch(sorted, "wang");
System.out.println(exact);
int caseInsensitive = Arrays.binarySearch(sorted, "Wang", String::compareToIgnoreCase);
System.out.println(caseInsensitive);

3
3
```

# 打印数组

Arrays 类提供的 toString 方法返回数组的字符串表示形式。

```java
public static String toString(Object[] a) {
    if (a == null)
        return "null";

    int iMax = a.length - 1;
    if (iMax == -1)
        return "[]";

    StringBuilder b = new StringBuilder();
    b.append('[');
    for (int i = 0; ; i++) {
        b.append(String.valueOf(a[i]));
        if (i == iMax)
            return b.append(']').toString();
        b.append(", ");
    }
}
```

# 数组转换

Arrays.asList() 返回的是 java.util.Arrays.ArrayList，并不是 java.util.ArrayList，它的长度是固定的，无法进行元素的删除或者添加。


```java
String[] intro = new String[] { "沉", "默", "王", "二" };
List<String> rets = Arrays.asList(intro);
System.out.println(rets.contains("二"));

List<String> rets1 = new ArrayList<>(Arrays.asList(intro));
rets1.add("三");
rets1.remove("二");
```
