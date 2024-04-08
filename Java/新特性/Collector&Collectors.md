- [collect\\Collector\\Collectors区别与关联](#collectcollectorcollectors区别与关联)
- [Collector使用与剖析](#collector使用与剖析)
  - [恒等处理](#恒等处理)
  - [归约总结](#归约总结)
  - [分组](#分组)

> https://heapdump.cn/article/4476339


# collect\Collector\Collectors区别与关联 

1️⃣ collect是Stream流的一个终止方法，会使用传入的收集器（入参）对结果执行相关的操作，这个收集器必须是Collector接口的某个具体实现类
2️⃣ Collector是一个接口，collect方法的收集器是Collector接口的具体实现类
3️⃣ Collectors是一个工具类，提供了很多的静态工厂方法，提供了很多Collector接口的具体实现类，是为了方便程序员使用而预置的一些较为通用的收集器（如果不使用Collectors类，而是自己去实现Collector接口，也可以）。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121342862.png)

# Collector使用与剖析

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121343624.png)

## 恒等处理

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121346867.png)

```Java
list.stream().collect(Collectors.toList());
list.stream().collect(Collectors.toSet());
list.stream().collect(Collectors.toCollection());
```

## 归约总结

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121347274.png)

```Java
public void calculateSum() {
    Integer salarySum = getAllEmployees().stream()
            .filter(employee -> "上海公司".equals(employee.getSubCompany()))
            .collect(Collectors.summingInt(Employee::getSalary));
    System.out.println(salarySum);
}
```

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121349199.png)

```Java
public void findHighestSalaryEmployee() {
    Optional<Employee> highestSalaryEmployee = getAllEmployees().stream()
            .filter(employee -> "上海公司".equals(employee.getSubCompany()))
            .collect(Collectors.maxBy(Comparator.comparingInt(Employee::getSalary)));
    System.out.println(highestSalaryEmployee.get());
}

public void findHighestSalaryEmployee2() {
    Optional<Employee> highestSalaryEmployee = getAllEmployees().stream()
            .filter(employee -> "上海公司".equals(employee.getSubCompany()))
            .max(Comparator.comparingInt(Employee::getSalary));
    System.out.println(highestSalaryEmployee.get());
}
```

## 分组

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202406121351914.png)

```Java
public void groupBySubCompany() {
    // 按照子公司维度将员工分组
    Map<String, List<Employee>> resultMap =
            getAllEmployees().stream()
                    .collect(Collectors.groupingBy(Employee::getSubCompany));
    System.out.println(resultMap);
}
```