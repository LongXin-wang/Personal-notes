- [Lombok](#lombok)
- [继承Lombok](#继承lombok)
- [常用注解](#常用注解)
  - [@Getter/@Setter](#gettersetter)
  - [@ToString](#tostring)
  - [@Val](#val)
  - [@Data](#data)
  - [@Slf4j](#slf4j)
  - [@Builder](#builder)
- [Lombok 的处理流程](#lombok-的处理流程)

# Lombok

Lombok 是个好类库，可以为 Java 代码添加一些“处理程序”，让其变得更简洁、更优雅。在我看来，Lombok 最大的好处就在于通过注解的形式来简化 Java 代码，简化到什么程度呢？

# 继承Lombok

```XML
<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
	<version>1.18.6</version>
	<scope>provided</scope>
</dependency>

```

# 常用注解

## @Getter/@Setter

```Java
class CmowerLombok {
	@Getter @Setter private int age;
	@Getter private String name;
	@Setter private BigDecimal money;
}
```

字节码反编译

```Java
class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;

	public int getAge() {
		return this.age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public String getName() {
		return this.name;
	}

	public void setMoney(BigDecimal money) {
		this.money = money;
	}
}
```

## @ToString

```Java
@ToString
class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;
}
```

字节码反编译

```Java
class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;

	public String toString() {
		return "CmowerLombok(age=" + this.age + ", name=" + this.name + ", money=" + this.money + ")";
	}
}
```

## @Val

在编写 JavaScript 代码时，我一直觉得 var 这个变量声明类型用起来特别方便。Lombok 也提供了一个类似的。

```Java
class CmowerLombok {
	public void test() {
		val names = new ArrayList<String>();
		names.add("沉默王二");
		val name = names.get(0);
		System.out.println(name);
	}
}
```

字节码反编译

```Java
class CmowerLombok {
	public void test() {
		ArrayList<String> names = new ArrayList();
		names.add("沉默王二");
		String name = (String) names.get(0);
		System.out.println(name);
	}
}
```

## @Data

@Data 注解可以生成 getter / setter、equals、hashCode，以及 toString，是个总和的选项。

```Java
@Data
class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;
}
```

字节码反编译

```Java
class CmowerLombok {
	private int age;
	private String name;
	private BigDecimal money;

	public int getAge() {
		return this.age;
	}

	public String getName() {
		return this.name;
	}

	public BigDecimal getMoney() {
		return this.money;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public void setName(String name) {
		this.name = name;
	}

	public void setMoney(BigDecimal money) {
		this.money = money;
	}

	public boolean equals(Object o) {
		if (o == this) {
			return true;
		} else if (!(o instanceof CmowerLombok)) {
			return false;
		} else {
			CmowerLombok other = (CmowerLombok) o;
			if (!other.canEqual(this)) {
				return false;
			} else if (this.getAge() != other.getAge()) {
				return false;
			} else {
				Object this$name = this.getName();
				Object other$name = other.getName();
				if (this$name == null) {
					if (other$name != null) {
						return false;
					}
				} else if (!this$name.equals(other$name)) {
					return false;
				}

				Object this$money = this.getMoney();
				Object other$money = other.getMoney();
				if (this$money == null) {
					if (other$money != null) {
						return false;
					}
				} else if (!this$money.equals(other$money)) {
					return false;
				}

				return true;
			}
		}
	}

	protected boolean canEqual(Object other) {
		return other instanceof CmowerLombok;
	}

	public int hashCode() {
		int PRIME = true;
		int result = 1;
		int result = result * 59 + this.getAge();
		Object $name = this.getName();
		result = result * 59 + ($name == null ? 43 : $name.hashCode());
		Object $money = this.getMoney();
		result = result * 59 + ($money == null ? 43 : $money.hashCode());
		return result;
	}

	public String toString() {
		return "CmowerLombok(age=" + this.getAge() + ", name=" + this.getName() + ", money=" + this.getMoney() + ")";
	}
}
```

## @Slf4j

@Slf4j 可以用来生成注解对象，你可以根据自己的日志实现方式来选用不同的注解，比如说：@Log、@Log4j、@Log4j2、@Slf4j 等。

```java
@Slf4j
public class Log4jDemo {
    public static void main(String[] args) {
        log.info("level:{}","info");
        log.warn("level:{}","warn");
        log.error("level:{}", "error");
    }
}
```

字节码反编译

```java
public class Log4jDemo {
    private static final Logger log = LoggerFactory.getLogger(Log4jDemo.class);

    public Log4jDemo() {
    }

    public static void main(String[] args) {
        log.info("level:{}", "info");
        log.warn("level:{}", "warn");
        log.error("level:{}", "error");
    }
}
```

## @Builder

@Builder 注解可以用来通过建造者模式来创建对象，这样就可以通过链式调用的方式进行对象赋值，非常的方便。

```Java
@Builder
@ToString
public class BuilderDemo {
    private Long id;
    private String name;
    private Integer age;

    public static void main(String[] args) {
        BuilderDemo demo = BuilderDemo.builder().age(18).name("沉默王二").build();
        System.out.println(demo);
    }
}
```

字节码反编译

```Java
public class BuilderDemo {
    private Long id;
    private String name;
    private Integer age;

    public static void main(String[] args) {
        BuilderDemo demo = builder().age(18).name("沉默王二").build();
        System.out.println(demo);
    }

    BuilderDemo(final Long id, final String name, final Integer age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    public static BuilderDemo.BuilderDemoBuilder builder() {
        return new BuilderDemo.BuilderDemoBuilder();
    }

    public String toString() {
        return "BuilderDemo(id=" + this.id + ", name=" + this.name + ", age=" + this.age + ")";
    }

    public static class BuilderDemoBuilder {
        private Long id;
        private String name;
        private Integer age;

        BuilderDemoBuilder() {
        }

        public BuilderDemo.BuilderDemoBuilder id(final Long id) {
            this.id = id;
            return this;
        }

        public BuilderDemo.BuilderDemoBuilder name(final String name) {
            this.name = name;
            return this;
        }

        public BuilderDemo.BuilderDemoBuilder age(final Integer age) {
            this.age = age;
            return this;
        }

        public BuilderDemo build() {
            return new BuilderDemo(this.id, this.name, this.age);
        }

        public String toString() {
            return "BuilderDemo.BuilderDemoBuilder(id=" + this.id + ", name=" + this.name + ", age=" + this.age + ")";
        }
    }
}
```

# Lombok 的处理流程

- javac 对源代码进行分析，生成一棵抽象语法树（AST）
- javac 编译过程中调用实现了JSR 269 的 Lombok 程序
- Lombok 对 AST 进行处理，找到 Lombok 注解所在类对应的语法树（AST），然后修改该语法树，增加 Lombok 注解定义的相应树节点（所谓代码）
- javac 使用修改后的抽象语法树生成字节码文件

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405221117854.png)
