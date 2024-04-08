- [简介](#简介)
- [重叠构造器模式](#重叠构造器模式)
- [JavaBeans 模式](#javabeans-模式)
- [静态工厂模式](#静态工厂模式)
- [Builder 模式](#builder-模式)
  - [Lombok: @Builder](#lombok-builder)

# 简介

当我们需要创建一个复杂的对象时，使用静态工厂或者构造器的方式就显得特别笨拙和丑陋，因为它们有个共同的局限性：它们都不能很好地扩展到大量的可选参数。考虑用一个Person类来描述一个人，除了姓名，性别，生日，邮箱等必要的属性外，还有很多可选的属性，比如：身高，学历，绰号，体重，通讯地址等等。对于这样的类，我们应该如何创建对象呢？无论是常见的重叠构造器模式还是JavaBeans模式，无法很好解决

# 重叠构造器模式

```Java
public class Person {
    private String name;    // required
    private String sex;     // required
    private Date date;      // required
    private String email;       // required

    private int height;     // optional
    private String edu;     // optional
    private String nickName;     // optional
    private int weight;     // optional
    private String addr;     // optional

    public Person(String name, String sex, Date date, String email) {
        this(name, sex, date, email, 0);
    }

    public Person(String name, String sex, Date date, String email, int height) {
        this(name, sex, date, email, height, null);
    }

    public Person(String name, String sex, Date date, String email, int height, String edu) {
        this(name, sex, date, email, height, edu, null);
    }

    public Person(String name, String sex, Date date, String email, int height, String edu, String nickName) {
        this(name, sex, date, email, height, edu, nickName, 0);
    }

    public Person(String name, String sex, Date date, String email, int height, String edu, String nickName, int
            weight) {
        this(name, sex, date, email, height, edu, nickName, weight, null);
    }

    public Person(String name, String sex, Date date, String email, int height, String edu, String nickName, int
            weight, String addr) {
        this.name = name;
        this.sex = sex;
        this.date = date;
        this.email = email;
        this.height = height;
        this.edu = edu;
        this.nickName = nickName;
        this.weight = weight;
        this.addr = addr;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                ", date=" + date +
                ", email='" + email + '\'' +
                ", height=" + height +
                ", edu='" + edu + '\'' +
                ", nickName='" + nickName + '\'' +
                ", weight=" + weight +
                ", addr='" + addr + '\'' +
                '}';
    }
}
```

# JavaBeans 模式

> JavaBeans 模式是一种用于创建可重用组件的约定，它是指遵循特定命名和编程模式的 Java 类。JavaBeans 类通常具有私有字段，公共的无参数构造函数，以及公共的 setter 和 getter 方法，这些方法用于设置和获取私有字段的值。

```Java
public class Person {
    private String name;    // required
    private String sex;     // required
    private Date date;      // required
    private String email;       // required

    private int height;     // optional
    private String edu;     // optional
    private String nickName;     // optional
    private int weight;     // optional
    private String addr;     // optional

    public void setName(String name) {
        this.name = name;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public void setDate(Date date) {
        this.date = date;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public void setEdu(String edu) {
        this.edu = edu;
    }

    public void setNickName(String nickName) {
        this.nickName = nickName;
    }

    public void setWeight(int weight) {
        this.weight = weight;
    }

    public void setAddr(String addr) {
        this.addr = addr;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                ", date=" + date +
                ", email='" + email + '\'' +
                ", height=" + height +
                ", edu='" + edu + '\'' +
                ", nickName='" + nickName + '\'' +
                ", weight=" + weight +
                ", addr='" + addr + '\'' +
                '}';
    }
}

Person p2 = new Person();
p2.setName("livia");
p2.setSex("girl");
p2.setDate(new Date());
p2.setEmail("livia@tju.edu.cn");
p2.setHeight(163);
p2.setEdu("NCU");
p2.setNickName("pig");
p2.setWeight(100);
p2.setAddr("北京市");
System.out.println(p2);
```

# 静态工厂模式

> 静态工厂模式是一种创建型设计模式，它使用静态方法来创建对象，而不是通过构造函数。静态工厂方法通常有自己的名称，通过这个名称我们可以清楚地了解创建的对象的含义。它可以用来隐藏创建对象的具体逻辑，可以根据需要返回缓存的实例，也可以返回子类的实例，从而实现更大的灵活性。

```Java
public class Car {
    private String brand;
    private String model;

    private Car(String brand, String model) {
        this.brand = brand;
        this.model = model;
    }

    public static Car createCar(String brand, String model) {
        // 可能包含逻辑用于返回缓存的实例
        return new Car(brand, model);
    }
}
```

# Builder 模式

- Person类的构造方法是私有的: 也就是说，客户端不能直接创建User对象，而是通过Builder来创建。
- 静态内部类 Builder 是 Builder 的内部类，它使用Builder构造Person对象。

```java
public class Person {
    private final String name;    // required
    private final String sex;     // required
    private final Date date;      // required
    private final String email;       // required

    private final int height;     // optional
    private final String edu;     // optional
    private final String nickName;     // optional
    private final int weight;     // optional
    private final String addr;     // optional

    // 私有构造器，因此Person对象的创建必须依赖于Builder
    private Person(Builder builder) {
        this.name = builder.name;
        this.sex = builder.sex;
        this.date = builder.date;
        this.email = builder.email;
        this.height = builder.height;
        this.edu = builder.edu;
        this.nickName = builder.nickName;
        this.weight = builder.weight;
        this.addr = builder.addr;
    }

    public static class Builder{
        private final String name;    // required，使用final修饰
        private final String sex;     // required，使用final修饰
        private final Date date;      // required，使用final修饰
        private final String email;       // required，使用final修饰

        private int height;     // optional，不使用final修饰
        private String edu;     // optional，不使用final修饰
        private String nickName;     // optional，不使用final修饰
        private int weight;     // optional，不使用final修饰
        private String addr;     // optional，不使用final修饰

        public Builder(String name, String sex, Date date, String email) {
            this.name = name;
            this.sex = sex;
            this.date = date;
            this.email = email;
        }

        // 返回Builder对象本身，链式调用
        public Builder height(int height){
            this.height = height;
            return this;
        }

        // 返回Builder对象本身，链式调用
        public Builder edu(String edu){
            this.edu = edu;
            return this;
        }

        // 返回Builder对象本身，链式调用
        public Builder nickName(String nickName){
            this.nickName = nickName;
            return this;
        }

        // 返回Builder对象本身，链式调用
        public Builder weight(int weight){
            this.weight = weight;
            return this;
        }

        // 返回Builder对象本身，链式调用
        public Builder addr(String addr){
            this.addr = addr;
            return this;
        }

        // 通过Builder构建所需Person对象，并且每次都产生新的Person对象
        public Person build(){
            return new Person(this);
        }
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                ", date=" + date +
                ", email='" + email + '\'' +
                ", height=" + height +
                ", edu='" + edu + '\'' +
                ", nickName='" + nickName + '\'' +
                ", weight=" + weight +
                ", addr='" + addr + '\'' +
                '}';
    }
}

Person.Builder builder = new Person.Builder("rico", "boy", new Date(), "rico@tju.edu.cn");
Person p1 = builder.height(173).addr("天津市").nickName("书呆子").build();
```

## Lombok: @Builder

> Lombok 是一个用于简化 Java 开发的工具包，它提供了一些注解和特性，可以帮助我们简化代码编写。其中一个注解是 @Builder，它可以用来生成 Builder 模式的类。

```Java
@Builder
public class Person {
    private String name;    // required
    private String sex;     // required
    private Date date;      // required
    private final String email;   // required
    private final int height;     // optional
    private final String edu;     // optional
    private final String nickName; // optional
    private final int weight;     // optional
    private final String addr;     // optional
}
```