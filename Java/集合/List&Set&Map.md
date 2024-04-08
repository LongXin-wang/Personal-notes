- [集合框架](#集合框架)
- [List](#list)
  - [ArrayList](#arraylist)
  - [LinkedList](#linkedlist)
  - [Vector \& Stack](#vector--stack)
- [Set](#set)
  - [HashSet](#hashset)
  - [LinkedHashSet](#linkedhashset)
  - [TreeSet](#treeset)
- [Queue](#queue)
  - [ArrayDeque](#arraydeque)
  - [LinkedList](#linkedlist-1)
  - [PriorityQueue](#priorityqueue)
- [Map](#map)
  - [HashMap](#hashmap)
  - [LinkedHashMap](#linkedhashmap)
  - [TreeMap](#treemap)


# 集合框架

Java 集合框架可以分为两条大的支线：

①、Collection，主要由 List、Set、Queue 组成：

List 代表有序、可重复的集合，典型代表就是封装了动态数组的 ArrayList 和封装了链表的 LinkedList；
Set 代表无序、不可重复的集合，典型代表就是 HashSet 和 TreeSet；
Queue 代表队列，典型代表就是双端队列 ArrayDeque，以及优先级队列 PriorityQueue。

②、Map，代表键值对的集合，典型代表就是 HashMap。


![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405231551191.png)

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121136978.png)

# List

## ArrayList

```Java
// 创建一个集合
ArrayList<String> list = new ArrayList<String>();
// 添加元素
list.add("王二");
list.add("沉默");
list.add("陈清扬");

// 遍历集合 for 循环
for (int i = 0; i < list.size(); i++) {
    String s = list.get(i);
    System.out.println(s);
}
// 遍历集合 for each
for (String s : list) {
    System.out.println(s);
}

// 删除元素
list.remove(1);
// 遍历集合
for (String s : list) {
    System.out.println(s);
}

// 修改元素
list.set(1, "王二狗");
// 遍历集合
for (String s : list) {
    System.out.println(s);
}
```

## LinkedList

```Java
// 创建一个集合
LinkedList<String> list = new LinkedList<String>();
// 添加元素
list.add("王二");
list.add("沉默");
list.add("陈清扬");

// 遍历集合 for 循环
for (int i = 0; i < list.size(); i++) {
    String s = list.get(i);
    System.out.println(s);
}
// 遍历集合 for each
for (String s : list) {
    System.out.println(s);
}

// 删除元素
list.remove(1);
// 遍历集合
for (String s : list) {
    System.out.println(s);
}

// 修改元素
list.set(1, "王二狗");
// 遍历集合
for (String s : list) {
    System.out.println(s);
}
```

## Vector & Stack

List 的实现类还有一个 Vector，是一个元老级的类，比 ArrayList 出现得更早。ArrayList 和 Vector 非常相似，只不过 Vector 是线程安全的，像 get、set、add 这些方法都加了 synchronized 关键字，就导致执行效率会比较低，所以现在已经很少用了。

我就不写太多代码了，只看一下 add 方法的源码就明白了。

```Java
public synchronized boolean add(E e) {
    elementData[elementCount++] = e;
    return true;
}
```

这种加了同步方法的类，注定会被淘汰掉，就像StringBuilder 取代 StringBuffer那样。

Stack 是 Vector 的一个子类，本质上也是由动态数组实现的，只不过还实现了先进后出的功能（在 get、set、add 方法的基础上追加了 pop「返回并移除栈顶的元素」、peek「只返回栈顶元素」等方法），所以叫栈。

下面是这两个方法的源码，增删改查我就不写了，和 ArrayList 和 LinkedList 几乎一样。

```java
public synchronized E pop() {
    E       obj;
    int     len = size();

    obj = peek();
    removeElementAt(len - 1);

    return obj;
}

public synchronized E peek() {
    int     len = size();

    if (len == 0)
        throw new EmptyStackException();
    return elementAt(len - 1);
}
```

# Set

## HashSet

HashSet 其实是由 HashMap 实现的，只不过值由一个固定的 Object 对象填充，而键用于操作。来简单看一下它的源码。

```Java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    private transient HashMap<E,Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();

    public HashSet() {
        map = new HashMap<>();
    }

    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }

    public boolean remove(Object o) {
        return map.remove(o)==PRESENT;
    }
}
```

```Java
// 创建一个新的HashSet
HashSet<String> set = new HashSet<>();

// 添加元素
set.add("沉默");
set.add("王二");
set.add("陈清扬");

// 输出HashSet的元素个数
System.out.println("HashSet size: " + set.size()); // output: 3

// 判断元素是否存在于HashSet中
boolean containsWanger = set.contains("王二");
System.out.println("Does set contain '王二'? " + containsWanger); // output: true

// 删除元素
boolean removeWanger = set.remove("王二");
System.out.println("Removed '王二'? " + removeWanger); // output: true

// 修改元素，需要先删除后添加
boolean removeChenmo = set.remove("沉默");
boolean addBuChenmo = set.add("不沉默");
System.out.println("Modified set? " + (removeChenmo && addBuChenmo)); // output: true

// 输出修改后的HashSet
System.out.println("HashSet after modification: " + set); // output: [陈清扬, 不沉默]
```

## LinkedHashSet

LinkedHashSet 是一种基于哈希表实现的 Set 接口，它继承自 HashSet，并且使用链表维护了元素的插入顺序。因此，它既具有 HashSet 的快速查找、插入和删除操作的优点，又可以维护元素的插入顺序。


```Java
LinkedHashSet<String> set = new LinkedHashSet<>();

// 添加元素
set.add("沉默");
set.add("王二");
set.add("陈清扬");

// 删除元素
set.remove("王二");

// 修改元素
set.remove("沉默");
set.add("沉默的力量");

// 查找元素
boolean hasChenQingYang = set.contains("陈清扬");
System.out.println("set包含陈清扬吗？" + hasChenQingYang);
```

## TreeSet

TreeSet 是一种基于红黑树实现的有序集合，它实现了 SortedSet 接口，可以自动对集合中的元素进行排序。按照键的自然顺序或指定的比较器顺序进行排序。

```Java
// 创建一个 TreeSet 对象
TreeSet<String> set = new TreeSet<>();

// 添加元素
set.add("沉默");
set.add("王二");
set.add("陈清扬");
System.out.println(set); // 输出 [沉默, 王二, 陈清扬]

// 删除元素
set.remove("王二");
System.out.println(set); // 输出 [沉默, 陈清扬]

// 修改元素：TreeSet 中的元素不支持直接修改，需要先删除再添加
set.remove("陈清扬");
set.add("陈青阳");
System.out.println(set); // 输出 [沉默, 陈青阳]

// 查找元素
System.out.println(set.contains("沉默")); // 输出 true
System.out.println(set.contains("王二")); // 输出 false
```

# Queue

## ArrayDeque

```java
// 创建一个ArrayDeque
ArrayDeque<String> deque = new ArrayDeque<>();

// 添加元素
deque.add("沉默");
deque.add("王二");
deque.add("陈清扬");

// 删除元素
deque.remove("王二");

// 修改元素
deque.remove("沉默");
deque.add("沉默的力量");

// 查找元素
boolean hasChenQingYang = deque.contains("陈清扬");
System.out.println("deque包含陈清扬吗？" + hasChenQingYang);
```

## LinkedList

LinkedList 是基于链表实现的，而 ArrayDeque 是基于数组实现的。

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{}
```

```java
// 创建一个 LinkedList 对象
LinkedList<String> queue = new LinkedList<>();

// 添加元素
queue.offer("沉默");
queue.offer("王二");
queue.offer("陈清扬");
System.out.println(queue); // 输出 [沉默, 王二, 陈清扬]

// 删除元素
queue.poll();
System.out.println(queue); // 输出 [王二, 陈清扬]

// 修改元素：LinkedList 中的元素不支持直接修改，需要先删除再添加
String first = queue.poll();
queue.offer("王大二");
System.out.println(queue); // 输出 [陈清扬, 王大二]

// 查找元素：LinkedList 中的元素可以使用 get() 方法进行查找
System.out.println(queue.get(0)); // 输出 陈清扬
System.out.println(queue.contains("沉默")); // 输出 false

// 查找元素：使用迭代器的方式查找陈清扬
// 使用迭代器依次遍历元素并查找
Iterator<String> iterator = queue.iterator();
while (iterator.hasNext()) {
    String element = iterator.next();
    if (element.equals("陈清扬")) {
        System.out.println("找到了：" + element);
        break;
    }
}
```

## PriorityQueue


```java
import java.util.Comparator;
import java.util.PriorityQueue;

class Student {
    private String name;
    private int chineseScore;
    private int mathScore;

    public Student(String name, int chineseScore, int mathScore) {
        this.name = name;
        this.chineseScore = chineseScore;
        this.mathScore = mathScore;
    }

    public String getName() {
        return name;
    }

    public int getChineseScore() {
        return chineseScore;
    }

    public int getMathScore() {
        return mathScore;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", 总成绩=" + (chineseScore + mathScore) +
                '}';
    }
}

class StudentComparator implements Comparator<Student> {
    @Override
    public int compare(Student s1, Student s2) {
        // 比较总成绩
        return Integer.compare(s2.getChineseScore() + s2.getMathScore(),
                s1.getChineseScore() + s1.getMathScore());
    }
}

public class PriorityQueueComparatorExample {

    public static void main(String[] args) {
        // 创建一个按照总成绩排序的优先级队列
        PriorityQueue<Student> queue = new PriorityQueue<>(new StudentComparator());

        // 添加元素
        queue.offer(new Student("王二", 80, 90));
        System.out.println(queue);
        queue.offer(new Student("陈清扬", 95, 95));
        System.out.println(queue);
        queue.offer(new Student("小驼铃", 90, 95));
        System.out.println(queue);
        queue.offer(new Student("沉默", 90, 80));
        while (!queue.isEmpty()) {
            System.out.print(queue.poll() + " ");
        }
    }
}
```

# Map

## HashMap

```Java
// 创建一个 HashMap 对象
HashMap<String, String> hashMap = new HashMap<>();

// 添加键值对
hashMap.put("沉默", "cenzhong");
hashMap.put("王二", "wanger");
hashMap.put("陈清扬", "chenqingyang");

// 获取指定键的值
String value1 = hashMap.get("沉默");
System.out.println("沉默对应的值为：" + value1);

// 修改键对应的值
hashMap.put("沉默", "chenmo");
String value2 = hashMap.get("沉默");
System.out.println("修改后沉默对应的值为：" + value2);

// 删除指定键的键值对
hashMap.remove("王二");

// 遍历 HashMap
for (String key : hashMap.keySet()) {
    String value = hashMap.get(key);
    System.out.println(key + " 对应的值为：" + value);
}
```

## LinkedHashMap

LinkedHashMap 是 HashMap 的子类，它继承自 HashMap，并且使用链表维护了元素的插入顺序。

```Java
// 创建一个 LinkedHashMap，插入的键值对为 沉默 王二 陈清扬
LinkedHashMap<String, String> linkedHashMap = new LinkedHashMap<>();
linkedHashMap.put("沉默", "cenzhong");
linkedHashMap.put("王二", "wanger");
linkedHashMap.put("陈清扬", "chenqingyang");

// 遍历 LinkedHashMap
for (String key : linkedHashMap.keySet()) {
    String value = linkedHashMap.get(key);
    System.out.println(key + " 对应的值为：" + value);
}
```

## TreeMap

 实现了 SortedMap 接口，可以自动将键按照自然顺序或指定的比较器顺序排序，并保证其元素的顺序。内部使用红黑树来实现键的排序和查找。

```java
// 创建一个 TreeMap 对象
Map<String, String> treeMap = new TreeMap<>();

// 向 TreeMap 中添加键值对
treeMap.put("沉默", "cenzhong");
treeMap.put("王二", "wanger");
treeMap.put("陈清扬", "chenqingyang");

// 查找键值对
String name = "沉默";
if (treeMap.containsKey(name)) {
    System.out.println("找到了 " + name + ": " + treeMap.get(name));
} else {
    System.out.println("没有找到 " + name);
}

// 修改键值对
name = "王二";
if (treeMap.containsKey(name)) {
    System.out.println("修改前的 " + name + ": " + treeMap.get(name));
    treeMap.put(name, "newWanger");
    System.out.println("修改后的 " + name + ": " + treeMap.get(name));
} else {
    System.out.println("没有找到 " + name);
}

// 删除键值对
name = "陈清扬";
if (treeMap.containsKey(name)) {
    System.out.println("删除前的 " + name + ": " + treeMap.get(name));
    treeMap.remove(name);
    System.out.println("删除后的 " + name + ": " + treeMap.get(name));
} else {
    System.out.println("没有找到 " + name);
}

// 遍历 TreeMap
for (Map.Entry<String, String> entry : treeMap.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

与 HashMap 不同的是，TreeMap 会按照键的顺序来进行排序。

```java
// 创建一个 TreeMap 对象
Map<String, String> treeMap = new TreeMap<>();

// 向 TreeMap 中添加键值对
treeMap.put("c", "cat");
treeMap.put("a", "apple");
treeMap.put("b", "banana");

// 遍历 TreeMap
for (Map.Entry<String, String> entry : treeMap.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

a: apple
b: banana
c: cat
```





