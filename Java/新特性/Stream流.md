- [流的概念](#流的概念)
- [流的创建](#流的创建)
- [操作流](#操作流)
  - [过滤](#过滤)
    - [filter、sorted、distinct、limit](#filtersorteddistinctlimit)
  - [映射](#映射)
    - [Map 和 flatMap](#map-和-flatmap)
  - [匹配](#匹配)
  - [组合](#组合)
- [中止流](#中止流)
  - [简单中止流](#简单中止流)
  - [结果中止收集 - 转换流](#结果中止收集---转换流)
    - [生成集合](#生成集合)
    - [生成拼接字符串](#生成拼接字符串)
    - [数据批量数学运算](#数据批量数学运算)
- [并行 Stream](#并行-stream)

# 流的概念

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121017254.png)

- stream 是 Collection 接口的方法，所以只有List Queue Set 都支持流。
- Map 接口不直接支持流，但是可以通过 Stream 接口的entrySet()

```Java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);

map.entrySet().stream()
    .map(entry -> entry.getKey() + ": " + entry.getValue())
    .forEach(System.out::println);
```

# 流的创建

单从“Stream”这个单词上来看，它似乎和 java.io 包下的 InputStream 和 OutputStream 有些关系。实际上呢，没毛关系。Java 8 新增的 Stream 是为了解放程序员操作集合（Collection）时的生产力

Stream 就好像一个高级的迭代器，但只能遍历一次，就好像一江春水向东流；在流的过程中，对流中的元素执行一些操作，比如“过滤掉长度大于 10 的字符串”、“获取每个字符串的首字母”等。

流的操作可以分为两种类型：

1）中间操作，可以有多个，每次返回一个新的流，可进行链式操作。

2）终端操作，只能有一个，每次执行完，这个流也就用光光了，无法执行下一个操作，因此只能放在最后。

中间操作不会立即执行，只有等到终端操作的时候，流才开始真正地遍历，用于映射、过滤等。通俗点说，就是一次遍历执行多个操作，性能就大大提高了。

| API              | 功能说明                                         |
| ---------------- | ------------------------------------------------ |
| stream()         | 创建出一个新的stream串行流对象                   |
| parallelStream() | 创建出一个可并行执行的stream流对象               |
| Stream.of()      | 通过给定的一系列元素创建一个新的Stream串行流对象 |


```Java
List<String> list = new ArrayList<>();
list.add("武汉加油");
list.add("中国加油");
list.add("世界加油");
list.add("世界加油");

long count = list.stream().distinct().count();
System.out.println(count);

Stream<T> distinct();
long count();
```

```Java
public class CreateStreamDemo {
    public static void main(String[] args) {
        String[] arr = new String[]{"武汉加油", "中国加油", "世界加油"};
        Stream<String> stream = Arrays.stream(arr);

        stream = Stream.of("武汉加油", "中国加油", "世界加油");

        List<String> list = new ArrayList<>();
        list.add("武汉加油");
        list.add("中国加油");
        list.add("世界加油");
        stream = list.stream();
    }
}
```

# 操作流

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121022015.png)

## 过滤

```Java
public class FilterStreamDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("周杰伦");
        list.add("王力宏");
        list.add("陶喆");
        list.add("林俊杰");
        Stream<String> stream = list.stream().filter(element -> element.contains("王"));
        stream.forEach(System.out::println);
    }
}
```

### filter、sorted、distinct、limit

```Java
public void testGetTargetUsers() {
    List<String> ids = Arrays.asList("205","10","308","49","627","193","111", "193");
    // 使用流操作
    List<Dept> results = ids.stream()
            .filter(s -> s.length() > 2)
            .distinct()
            .map(Integer::valueOf)
            .sorted(Comparator.comparingInt(o -> o))
            .limit(3)
            .map(id -> new Dept(id))
            .collect(Collectors.toList());
    System.out.println(results);
}

[Dept{id=111},  Dept{id=193},  Dept{id=205}]
```

## 映射

如果想通过某种操作把一个流中的元素转化成新的流中的元素，可以使用 map() 方法。

```Java
public class MapStreamDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("周杰伦");
        list.add("王力宏");
        list.add("陶喆");
        list.add("林俊杰");
        Stream<Integer> stream = list.stream().map(String::length);
        stream.forEach(System.out::println);
    }
}
```

### Map 和 flatMap

flatMap操作的时候其实是先每个元素处理并返回一个新的Stream，然后将多个Stream展开合并为了一个完整的新的Stream

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121028798.png)

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121029345.png)


```Java
/**
 * 演示map的用途：一对一转换
 */
public void stringToIntMap() {
    List<String> ids = Arrays.asList("205", "105", "308", "469", "627", "193", "111");
    // 使用流操作
    List<User> results = ids.stream()
            .map(id -> {
                User user = new User();
                user.setId(id);
                return user;
            })
            .collect(Collectors.toList());
    System.out.println(results);
}

[User{id='205'}, 
 User{id='105'},
 User{id='308'}, 
 User{id='469'}, 
 User{id='627'}, 
 User{id='193'}, 
 User{id='111'}]
```

```java
public void stringToIntFlatmap() {
    List<String> sentences = Arrays.asList("hello world","Jia Gou Wu Dao");
    // 使用流操作
    List<String> results = sentences.stream()
            .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
            .collect(Collectors.toList());
    System.out.println(results);
}

[hello, world, Jia, Gou, Wu, Dao]
```

## 匹配

```java
public class MatchStreamDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("周杰伦");
        list.add("王力宏");
        list.add("陶喆");
        list.add("林俊杰");

        boolean  anyMatchFlag = list.stream().anyMatch(element -> element.contains("王"));
        boolean  allMatchFlag = list.stream().allMatch(element -> element.length() > 1);
        boolean  noneMatchFlag = list.stream().noneMatch(element -> element.endsWith("沉"));
        System.out.println(anyMatchFlag); //true
        System.out.println(allMatchFlag); //true
        System.out.println(noneMatchFlag); //true
    }
}
```

## 组合

```Java
public class ReduceStreamDemo {
    public static void main(String[] args) {
        Integer[] ints = {0, 1, 2, 3};
        List<Integer> list = Arrays.asList(ints);

        Optional<Integer> optional = list.stream().reduce((a, b) -> a + b);
        Optional<Integer> optional1 = list.stream().reduce(Integer::sum);
        System.out.println(optional.orElse(0));    //6
        System.out.println(optional1.orElse(0));    //6

        int reduce = list.stream().reduce(6, (a, b) -> a + b);
        System.out.println(reduce);    //12
        int reduce1 = list.stream().reduce(6, Integer::sum);
        System.out.println(reduce1);    //12
    }
}
```

# 中止流

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121022061.png)

## 简单中止流

止方法里面像count、max、min、findAny、findFirst、anyMatch、allMatch、nonneMatch等方法，均属于这里说的简单结果终止方法。所谓简单，指的是其结果形式是数字、布尔值或者Optional对象值等。

```Java
public void testSimpleStopOptions() {
    List<String> ids = Arrays.asList("205", "10", "308", "49", "627", "193", "111", "193");
    // 统计stream操作后剩余的元素个数
    System.out.println(ids.stream().filter(s -> s.length() > 2).count());
    // 判断是否有元素值等于205
    System.out.println(ids.stream().filter(s -> s.length() > 2).anyMatch("205"::equals));
    // findFirst操作
    ids.stream().filter(s -> s.length() > 2)
            .findFirst()
            .ifPresent(s -> System.out.println("findFirst:" + s));
}

6
true
findFirst:205
```

##  结果中止收集 - 转换流

既然可以把集合或者数组转成流，那么也应该有对应的方法，将流转换回去——collect() 方法就满足了这种需求。

### 生成集合

```Java
public class CollectStreamDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("周杰伦");
        list.add("王力宏");
        list.add("陶喆");
        list.add("林俊杰");

        String[] strArray = list.stream().toArray(String[]::new);
        System.out.println(Arrays.toString(strArray));

        List<Integer> list1 = list.stream().map(String::length).collect(Collectors.toList());
        List<String> list2 = list.stream().collect(Collectors.toCollection(ArrayList::new));
        System.out.println(list1);
        System.out.println(list2);

        String str = list.stream().collect(Collectors.joining(", ")).toString();
        System.out.println(str);
    }
}

[周杰伦, 王力宏, 陶喆, 林俊杰]
[3, 3, 2, 3]
[周杰伦, 王力宏, 陶喆, 林俊杰]
周杰伦, 王力宏, 陶喆, 林俊杰
```


```Java
public void testCollectStopOptions() {
    List<Dept> ids = Arrays.asList(new Dept(17), new Dept(22), new Dept(23));
    // collect成list
    List<Dept> collectList = ids.stream().filter(dept -> dept.getId() > 20)
            .collect(Collectors.toList());
    System.out.println("collectList:" + collectList);
    // collect成Set
    Set<Dept> collectSet = ids.stream().filter(dept -> dept.getId() > 20)
            .collect(Collectors.toSet());
    System.out.println("collectSet:" + collectSet);
    // collect成HashMap，key为id，value为Dept对象
    Map<Integer, Dept> collectMap = ids.stream().filter(dept -> dept.getId() > 20)
            .collect(Collectors.toMap(Dept::getId, dept -> dept));
    System.out.println("collectMap:" + collectMap);
}

collectList:[Dept{id=22}, Dept{id=23}]
collectSet:[Dept{id=23}, Dept{id=22}]
collectMap:{22=Dept{id=22}, 23=Dept{id=23}}
```

### 生成拼接字符串

```java

public void testForJoinStrings() {
    List<String> ids = Arrays.asList("205", "10", "308", "49", "627", "193", "111", "193");
    StringBuilder builder = new StringBuilder();
    for (String id : ids) {
        builder.append(id).append(',');
    }
    // 去掉末尾多拼接的逗号
    builder.deleteCharAt(builder.length() - 1);
    System.out.println("拼接后：" + builder.toString());
}



public void testCollectJoinStrings() {
    List<String> ids = Arrays.asList("205", "10", "308", "49", "627", "193", "111", "193");
    String joinResult = ids.stream().collect(Collectors.joining(","));
    System.out.println("拼接后：" + joinResult);
}

拼接后：205,10,308,49,627,193,111,193
```

### 数据批量数学运算

```java

public void testNumberCalculate() {
    List<Integer> ids = Arrays.asList(10, 20, 30, 40, 50);
    // 计算平均值
    Double average = ids.stream().collect(Collectors.averagingInt(value -> value));
    System.out.println("平均值：" + average);
    // 数据统计信息
    IntSummaryStatistics summary = ids.stream().collect(Collectors.summarizingInt(value -> value));
    System.out.println("数据统计信息： " + summary);
}


平均值：30.0
总和： IntSummaryStatistics{count=5, sum=150, min=10, average=30.000000, max=50}
```

# 并行 Stream

使用并行流，可以有效利用计算机的多CPU硬件，提升逻辑的执行速度。并行流通过将一整个stream划分为多个片段，然后对各个分片流并行执行处理逻辑，最后将各个分片流的执行结果汇总为一个整体流。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121113564.png)
