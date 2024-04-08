- [排序操作](#排序操作)
- [查找操作](#查找操作)
- [同步控制](#同步控制)
- [不可变集合](#不可变集合)

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121400148.png)

# 排序操作

```java
reverse(List list)：反转顺序
shuffle(List list)：洗牌，将顺序打乱
sort(List list)：自然升序
sort(List list, Comparator c)：按照自定义的比较器排序
swap(List list, int i, int j)：将 i 和 j 位置的元素交换位置
```

```java
List<String> list = new ArrayList<>();
list.add("沉默王二");
list.add("沉默王三");
list.add("沉默王四");
list.add("沉默王五");
list.add("沉默王六");

System.out.println("原始顺序：" + list);

// 反转
Collections.reverse(list);
System.out.println("反转后：" + list);

// 洗牌
Collections.shuffle(list);
System.out.println("洗牌后：" + list);

// 自然升序
Collections.sort(list);
System.out.println("自然升序后：" + list);

// 交换
Collections.swap(list, 2,4);
System.out.println("交换后：" + list);

原始顺序：[沉默王二, 沉默王三, 沉默王四, 沉默王五, 沉默王六]
反转后：[沉默王六, 沉默王五, 沉默王四, 沉默王三, 沉默王二]
洗牌后：[沉默王五, 沉默王二, 沉默王六, 沉默王三, 沉默王四]
自然升序后：[沉默王三, 沉默王二, 沉默王五, 沉默王六, 沉默王四]
交换后：[沉默王三, 沉默王二, 沉默王四, 沉默王六, 沉默王五]
```

# 查找操作

```java
binarySearch(List list, Object key)：二分查找法，前提是 List 已经排序过了
max(Collection coll)：返回最大元素
max(Collection coll, Comparator comp)：根据自定义比较器，返回最大元素
min(Collection coll)：返回最小元素
min(Collection coll, Comparator comp)：根据自定义比较器，返回最小元素
fill(List list, Object obj)：使用指定对象填充
frequency(Collection c, Object o)：返回指定对象出现的次数
```

```java
System.out.println("最大元素：" + Collections.max(list));
System.out.println("最小元素：" + Collections.min(list));
System.out.println("出现的次数：" + Collections.frequency(list, "沉默王二"));

// 没有排序直接调用二分查找，结果是不确定的
System.out.println("排序前的二分查找结果：" + Collections.binarySearch(list, "沉默王二"));
Collections.sort(list);
// 排序后，查找结果和预期一致
System.out.println("排序后的二分查找结果：" + Collections.binarySearch(list, "沉默王二"));

Collections.fill(list, "沉默王八");
System.out.println("填充后的结果：" + list);

原始顺序：[沉默王二, 沉默王三, 沉默王四, 沉默王五, 沉默王六]
最大元素：沉默王四
最小元素：沉默王三
出现的次数：1
排序前的二分查找结果：0
排序后的二分查找结果：1
填充后的结果：[沉默王八, 沉默王八, 沉默王八, 沉默王八, 沉默王八]
```

# 同步控制

HashMap 是线程不安全的，这个我们前面讲到了。那其实 ArrayList 也是线程不安全的，没法在多线程环境下使用，那 Collections 工具类中提供了多个 synchronizedXxx 方法，这些方法会返回一个同步的对象，从而解决多线程中访问集合时的安全问题。

```Java
SynchronizedList synchronizedList = Collections.synchronizedList(list);

static class SynchronizedList<E>
    extends SynchronizedCollection<E>
    implements List<E> {
    private static final long serialVersionUID = -7754090372962971524L;

    final List<E> list;

    SynchronizedList(List<E> list) {
        super(list); // 调用父类 SynchronizedCollection 的构造方法，传入 list
        this.list = list; // 初始化成员变量 list
    }

    // 获取指定索引处的元素
    public E get(int index) {
        synchronized (mutex) {return list.get(index);} // 加锁，调用 list 的 get 方法获取元素
    }
    
    // 在指定索引处插入指定元素
    public void add(int index, E element) {
        synchronized (mutex) {list.add(index, element);} // 加锁，调用 list 的 add 方法插入元素
    }
    
    // 移除指定索引处的元素
    public E remove(int index) {
        synchronized (mutex) {return list.remove(index);} // 加锁，调用 list 的 remove 方法移除元素
    }
}
```

# 不可变集合

emptyXxx()：制造一个空的不可变集合
singletonXxx()：制造一个只有一个元素的不可变集合
unmodifiableXxx()：为指定集合制作一个不可变集合

```Java
public static final <T> List<T> emptyList() {
    return (List<T>) EMPTY_LIST;
}

public static final List EMPTY_LIST = new EmptyList<>();
```