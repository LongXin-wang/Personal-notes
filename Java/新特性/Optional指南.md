- [Optional 对象](#optional-对象)
  - [Optional.of \& Optional.ofNullable](#optionalof--optionalofnullable)
  - [isPresent](#ispresent)
  - [ifPresent](#ifpresent)
  - [orElse \& orElseGet](#orelse--orelseget)
  - [map \& flatMap](#map--flatmap)
  - [filter](#filter)

# Optional 对象

## Optional.of & Optional.ofNullable

1）可以使用静态方法 empty() 创建一个空的 Optional 对象

```java
Optional<String> empty = Optional.empty();
System.out.println(empty); // 输出：Optional.empty
```

2）可以使用静态方法 of() 创建一个非空的 Optional 对象

```java
Optional<String> opt = Optional.of("沉默王二");
System.out.println(opt); // 输出：Optional[沉默王二]
```

3）可以使用静态方法 ofNullable() 建一个即可空又可非空的 Optional 对象


```java
String name = null;
Optional<String> optOrNull = Optional.ofNullable(name);
System.out.println(optOrNull); // 输出：Optional.empty
```


## isPresent

可以通过方法 isPresent() 判断一个 Optional 对象是否存在，如果存在，该方法返回 true，否则返回 false——取代了 obj != null 的判断。

```java
Optional<String> opt = Optional.of("沉默王二");
System.out.println(opt.isPresent()); // 输出：true

Optional<String> optOrNull = Optional.ofNullable(null);
System.out.println(optOrNull.isPresent()); // 输出：false


Optional<String> opt = Optional.of("沉默王二");
System.out.println(opt.isEmpty()); // 输出：false

Optional<String> optOrNull = Optional.ofNullable(null);
System.out.println(optOrNull.isEmpty()); // 输出：true
```

## ifPresent

ifPresent() 方法返回的是一个 Optional，所以可以使用 ifPresent() 方法对其进行处理。

```java
Optional<String> opt = Optional.of("沉默王二");
opt.ifPresent(str -> System.out.println(str.length()));

Java 9 后还可以通过方法 ifPresentOrElse(action, emptyAction) 执行两种结果，非空时执行 action，空时执行 emptyAction。
Optional<String> opt = Optional.of("沉默王二");
opt.ifPresentOrElse(str -> System.out.println(str.length()), () -> System.out.println("为空"));
```

## orElse & orElseGet

orElse() 方法用于返回包裹在 Optional 对象中的值，如果该值不为 null，则返回；否则返回默认值。该方法的参数类型和值的类型一致。

```java
String nullName = null;
String name = Optional.ofNullable(nullName).orElse("沉默王二");
System.out.println(name); // 输出：沉默王二

//orElseGet与orElse方法类似，区别在于orElse传入的是默认值，
//orElseGet可以接受一个lambda表达式生成默认值。
//输出: Default Value
System.out.println(empty.orElseGet(() -> "Default Value"));
//输出: Sanaulla
System.out.println(name.orElseGet(() -> "Default Value"));
```

## map & flatMap

```java
//map方法执行传入的lambda表达式参数对Optional实例的值进行修改。
//为lambda表达式的返回值创建新的Optional实例作为map方法的返回值。
Optional<String> upperName = name.map((value) -> value.toUpperCase());
System.out.println(upperName.orElse("No value found"));

//flatMap与map(Function)非常类似，区别在于传入方法的lambda表达式的返回类型。
//map方法中的lambda表达式返回值可以是任意类型，在map函数返回之前会包装为Optional。 
//但flatMap方法中的lambda表达式返回值必须是Optionl实例。 
upperName = name.flatMap((value) -> Optional.of(value.toUpperCase()));
System.out.println(upperName.orElse("No value found"));//输出SANAULLA
```

## filter

```java
//filter方法检查给定的Option值是否满足某些条件。
//如果满足则返回同一个Option实例，否则返回空Optional。
Optional<String> longName = name.filter((value) -> value.length() > 6);
System.out.println(longName.orElse("The name is less than 6 characters"));//输出Sanaulla

//另一个例子是Optional值不满足filter指定的条件。
Optional<String> anotherName = Optional.of("Sana");
Optional<String> shortName = anotherName.filter((value) -> value.length() > 6);
//输出: name长度不足6字符
System.out.println(shortName.orElse("The name is less than 6 characters"));
```