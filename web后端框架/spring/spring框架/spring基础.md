- [Spring 架构](#spring-架构)
- [IOC 控制反转、DI依赖注入](#ioc-控制反转di依赖注入)
  - [理解IOC](#理解ioc)
  - [IOC 配置方式](#ioc-配置方式)
    - [XML配置](#xml配置)
    - [Java配置](#java配置)
    - [注解配置](#注解配置)
  - [DI 依赖注入](#di-依赖注入)
    - [XML 方式](#xml-方式)
      - [setter方式](#setter方式)
      - [构造函数](#构造函数)
    - [注解注入](#注解注入)
      - [属性注入装配](#属性注入装配)
      - [Lombok 注解](#lombok-注解)
- [AOP](#aop)
  - [AOP的配置方式](#aop的配置方式)
    - [XML Schema配置方式](#xml-schema配置方式)
    - [AspectJ注解方式](#aspectj注解方式)

# Spring 架构

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405181407153.png)

# IOC 控制反转、DI依赖注入

- Spring框架管理这些Bean的创建工作，即由用户管理Bean转变为框架管理Bean，这个就叫**控制反转 - Inversion of Control (IoC)**
- Spring 框架托管创建的Bean放在哪里呢？ 这便是**IoC Container**;
- Spring 框架为了更好让用户配置Bean，必然会引入**不同方式来配置Bean？ 这便是xml配置，Java配置，注解配置**等支持
- Spring 框架既然接管了Bean的生成，必然需要**管理整个Bean的生命周期**等；
- 应用程序代码从Ioc Container中获取依赖的Bean，注入到应用程序中，这个过程叫 **依赖注入(Dependency Injection，DI)** ； 所以说控制反转是通过依赖注入实现的，其实它们是同一个概念的不同角度描述。通俗来说就是**IoC是设计思想，DI是实现方式**
- 在依赖注入时，有哪些方式呢？这就是构造器方式，@Autowired, @Resource, @Qualifier... 同时Bean之间存在依赖（可能存在先后顺序问题，以及**循环依赖问题**等）

> Flask 是一个基于 Python 的轻量级 Web 框架，每当有请求进来时，它会创建一个新的应用上下文(current_app指向app)和请求上下文，并在这些上下文中创建相应的对象。而 Java 中的 Bean 是一个单例，即在应用启动时就被创建，并在整个应用的生命周期中保持不变。

> 这种差异主要是由于 Python 和 Java 在语言层面上的设计和实现不同所造成的。Java 的对象创建和管理是由 JVM 来完成的，而 Python 中的对象创建和管理则是由解释器（Interpreter）来完成的。在 Python 中，每个请求都需要创建相应的对象，因为 Python 中的对象是动态创建的，并且每个对象都有自己的状态和行为。而在 Java 中，Bean 是静态创建的，因此它们在整个应用的生命周期中都是可用的。

## 理解IOC

**控制反转IOC是框架管理bean，将bean放在IoC容器中，DI则从IoC容器中获取bean注入到程序中**

传统应用程序都是由我们在类内部主动创建依赖对象，从而导致类与类之间高耦合，难于测试；有了IoC容器后，**把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是 松散耦合，这样也方便测试，利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活**。

DI—Dependency Injection，即依赖注入：组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405181412742.png)

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405181412852.png)

## IOC 配置方式

### XML配置

顾名思义，就是将bean的信息配置.xml文件里，通过Spring加载文件为我们创建bean。这种方式出现很多早前的SSM项目中，将第三方类库或者一些配置工具类都以这种方式进行配置，主要原因是由于第三方类不支持Spring注解。

- **优点**： 可以使用于任何场景，结构清晰，通俗易懂
- **缺点**： 配置繁琐，不易维护，枯燥无味，扩展性差

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- services -->
    <bean id="userService" class="tech.pdai.springframework.service.UserServiceImpl">
        <property name="userDao" ref="userDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>
    <!-- more bean definitions for services go here -->
</beans>
```

### Java配置

创建一个配置类， 添加@Configuration注解声明为配置类

创建方法，方法上加上@bean，该方法用于创建实例并返回，该实例创建后会交给spring管理，方法名建议与实例名相同（首字母小写）。注：实例类不需要加任何注解

```Java
@Configuration
public class BeansConfig {

    /**
     * @return user dao
     */
    @Bean("userDao")
    public UserDaoImpl userDao() {
        return new UserDaoImpl();
    }

    /**
     * @return user service
     */
    @Bean("userService")
    public UserServiceImpl userService() {
        UserServiceImpl userService = new UserServiceImpl();
        userService.setUserDao(userDao());
        return userService;
    }
}
```

### 注解配置

通过在类上加注解的方式，来声明一个类交给Spring管理，Spring会自动扫描带有@Component，@Controller，@Service，@Repository这四个注解的类，然后帮我们创建并管理，前提是需要先配置Spring的注解扫描器。

- **优点**：开发便捷，通俗易懂，方便维护。
- **缺点**：具有局限性，对于一些第三方资源，无法添加注解。只能采用XML或JavaConfig的方式配置

---

1. 对类添加@Component相关的注解，比如@Controller，@Service，@Repository
2. 设置ComponentScan的basePackage, 比如`<context:component-scan base-package='tech.pdai.springframework'>`, 或者`@ComponentScan("tech.pdai.springframework")`注解，或者 `new AnnotationConfigApplicationContext("tech.pdai.springframework")`指定扫描的basePackage.

---

xml与注解最佳实践：

- xml用来管理bean；
- 注解只负责完成属性的注入；
- 我们在使用的过程中，只需要注意一个问题：必须让注解生效，就需要开启注解的支持。

    <!--指定要扫描的包，这个包下的注解会生效-->

    <context:component-scan base-package="com.awen"/>

    <[context:annotation-config/](context:annotation-config/)>

```Java
@Service
public class UserServiceImpl {

    /**
     * user dao impl.
     */
    @Autowired
    private UserDaoImpl userDao;

    /**
     * find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return userDao.findUserList();
    }

}
```

## DI 依赖注入

> https://www.cnblogs.com/liuawen/p/12854071.html

### XML 方式

#### setter方式

**在XML配置方式中**，property都是setter方式注入，比如下面的xml:

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- services -->
    <bean id="userService" class="tech.pdai.springframework.service.UserServiceImpl">
        <property name="userDao" ref="userDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>
    <!-- more bean definitions for services go here -->
</beans>
```

本质上包含两步：

1. 第一步，需要new UserServiceImpl()创建对象, 所以需要默认构造函数
2. 第二步，调用setUserDao()函数注入userDao的值, 所以需要setUserDao()函数，但是比较麻烦，所以推荐构造函数注入



#### 构造函数

**在XML配置方式中**，`<constructor-arg>`是通过构造函数参数注入，比如下面的xml:

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- services -->
    <bean id="userService" class="tech.pdai.springframework.service.UserServiceImpl">
        <constructor-arg name="userDao" ref="userDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>
    <!-- more bean definitions for services go here -->
</beans>
```

```Java
public class UserServiceImpl {

    /**
     * user dao impl.
     */
    private final UserDaoImpl userDao;

    /**
     * init.
     * @param userDaoImpl user dao impl
     */
    public UserServiceImpl(UserDaoImpl userDaoImpl) {
        this.userDao = userDaoImpl;
    }

    /**
     * find user list.
     *
     * @return user list
     */
    public List<User> findUserList() {
        return this.userDao.findUserList();
    }

}

```

### 注解注入

- @Resource：其作用与Autowired一样。其区别在于@Autowired默认按照Bean类型装配，而@Resource默认按照Bean实例名称进行装配。@Resource中有两个重要属性：name和type。Spring将name属性解析为Bean实例名称，type属性解析为Bean实例类型。如果指定name属性，则按实例名称进行装配；如果指定type属性，则按Bean类型进行装配；如果都不指定，则先按Bean实例名称装配，如果不能匹配，再按照Bean类型进行装配；如果都无法匹配，则抛出NoSuchBeanDefinitionException异常。
- @Qualifier：与@Autowired注解配合使用，会将默认的按Bean类型装配修改为按Bean的实例名称装配，Bean的实例名称由@Qualifier注解的参数指定。在上面几个注解中，虽然@Repository、@Service与@Controller功能与@Component注解的功能相同，但为了使标注类本身用途更加清晰，建议在实际开发中使用@Repository、@Service与@Controller分别对实现类进行标注

#### 属性注入装配

```Java
public class SysUserController extends BaseController {
    @Autowired
    private ISysUserService userService;

    @Resource
    private ISysRoleService roleService;
}
```

#### Lombok 注解


当使用@AllArgsConstructor & @RequiredArgsConstructor 等注解时，Spring IoC容器会使用构造函数注入，根据构造函数的参数类型来确定需要注入的依赖。这意味着在使用@AllArgsConstructor注解的类中，Spring IoC容器会自动识别构造函数的参数，并注入相应的依赖。

```Java
@AllArgsConstructor
public class Student{

    private String name;

    // 被final修饰
    private final String age;

    @NonNull
    private String sex;
}

public class Student
{

    public Student(String name, String age, String sex)
    {
        if(sex == null)
        {
            throw new NullPointerException("sex is marked non-null but is null");
        } else
        {
            this.name = name;
            this.age = age;
            this.sex = sex;
            return;
        }
    }

    private String name;
    private final String age;
    private String sex;
}


@RequiredArgsConstructor
public class Student
{
    private String name;

    // 被final修饰
    private final String age;

    @NonNull
    private String sex;
}

public class Student
{

    public Student(String age, String sex)
    {
        if(sex == null)
        {
            throw new NullPointerException("sex is marked non-null but is null");
        } else
        {
            this.age = age;
            this.sex = sex;
            return;
        }
    }

    private String name;
    private final String age;
    private String sex;
}



```

# AOP

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405181507259.png)

**动态织入**和**静态织入**：

1. 动态织入的方式是在运行时动态将要增强的代码织入到目标类中，这样往往是通过动态代理技术完成的，如Java JDK的动态代理(Proxy，底层通过反射实现)或者CGLIB的动态代理(底层通过继承实现)，Spring AOP采用的就是基于运行时增强的代理技术
2. ApectJ采用的就是静态织入的方式。ApectJ主要采用的是编译期织入，在这个期间使用AspectJ的acj编译器(类似javac)把aspect类编译成class字节码后，在java目标类编译时织入，即先编译aspect类再编译目标类。

## AOP的配置方式

### XML Schema配置方式

```Java
package tech.pdai.springframework.aspect;

import org.aspectj.lang.ProceedingJoinPoint;

/**
 * @author pdai
 */
public class LogAspect {

    /**
     * 环绕通知.
     *
     * @param pjp pjp
     * @return obj
     * @throws Throwable exception
     */
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("-----------------------");
        System.out.println("环绕通知: 进入方法");
        Object o = pjp.proceed();
        System.out.println("环绕通知: 退出方法");
        return o;
    }

    /**
     * 前置通知.
     */
    public void doBefore() {
        System.out.println("前置通知");
    }

    /**
     * 后置通知.
     *
     * @param result return val
     */
    public void doAfterReturning(String result) {
        System.out.println("后置通知, 返回值: " + result);
    }

    /**
     * 异常通知.
     *
     * @param e exception
     */
    public void doAfterThrowing(Exception e) {
        System.out.println("异常通知, 异常: " + e.getMessage());
    }

    /**
     * 最终通知.
     */
    public void doAfter() {
        System.out.println("最终通知");
    }

}

```

```XML
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd
 http://www.springframework.org/schema/aop
 http://www.springframework.org/schema/aop/spring-aop.xsd
 http://www.springframework.org/schema/context
 http://www.springframework.org/schema/context/spring-context.xsd
">

    <context:component-scan base-package="tech.pdai.springframework" />

    <aop:aspectj-autoproxy/>

    <!-- 目标类 -->
    <bean id="demoService" class="tech.pdai.springframework.service.AopDemoServiceImpl">
        <!-- configure properties of bean here as normal -->
    </bean>

    <!-- 切面 -->
    <bean id="logAspect" class="tech.pdai.springframework.aspect.LogAspect">
        <!-- configure properties of aspect here as normal -->
    </bean>

    <aop:config>
        <!-- 配置切面 -->
        <aop:aspect ref="logAspect">
            <!-- 配置切入点 -->
            <aop:pointcut id="pointCutMethod" expression="execution(* tech.pdai.springframework.service.*.*(..))"/>
            <!-- 环绕通知 -->
            <aop:around method="doAround" pointcut-ref="pointCutMethod"/>
            <!-- 前置通知 -->
            <aop:before method="doBefore" pointcut-ref="pointCutMethod"/>
            <!-- 后置通知；returning属性：用于设置后置通知的第二个参数的名称，类型是Object -->
            <aop:after-returning method="doAfterReturning" pointcut-ref="pointCutMethod" returning="result"/>
            <!-- 异常通知：如果没有异常，将不会执行增强；throwing属性：用于设置通知第二个参数的的名称、类型-->
            <aop:after-throwing method="doAfterThrowing" pointcut-ref="pointCutMethod" throwing="e"/>
            <!-- 最终通知 -->
            <aop:after method="doAfter" pointcut-ref="pointCutMethod"/>
        </aop:aspect>
    </aop:config>

    <!-- more bean definitions for data access objects go here -->
</beans>
```


### AspectJ注解方式

基于XML的声明式AspectJ存在一些不足，需要在Spring配置文件配置大量的代码信息，为了解决这个问题，Spring 使用了@AspectJ框架为AOP的实现提供了一套注解。

Spring AOP的实现方式是动态织入，动态织入的方式是在运行时动态将要增强的代码织入到目标类中，这样往往是通过动态代理技术完成的；**如Java JDK的动态代理(Proxy，底层通过反射实现)或者CGLIB的动态代理(底层通过继承实现)**，Spring AOP采用的就是基于运行时增强的代理技术

| 注解名称        | 解释                                                                                                                                                                                                                                                               |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| @Aspect         | 用来定义一个切面。                                                                                                                                                                                                                                                 |
| @pointcut       | 用于定义切入点表达式。在使用时还需要定义一个包含名字和任意参数的方法签名来表示切入点名称，这个方法签名就是一个返回值为void，且方法体为空的普通方法。                                                                                                               |
| @Before         | 用于定义前置通知，相当于BeforeAdvice。在使用时，通常需要指定一个value属性值，该属性值用于指定一个切入点表达式(可以是已有的切入点，也可以直接定义切入点表达式)。                                                                                                    |
| @AfterReturning | 用于定义后置通知，相当于AfterReturningAdvice。在使用时可以指定pointcut / value和returning属性，其中pointcut / value这两个属性的作用一样，都用于指定切入点表达式。                                                                                                  |
| @Around         | 用于定义环绕通知，相当于MethodInterceptor。在使用时需要指定一个value属性，该属性用于指定该通知被植入的切入点。                                                                                                                                                     |
| @After-Throwing | 用于定义异常通知来处理程序中未处理的异常，相当于ThrowAdvice。在使用时可指定pointcut / value和throwing属性。其中pointcut/value用于指定切入点表达式，而throwing属性值用于指定-一个形参名来表示Advice方法中可定义与此同名的形参，该形参可用于访问目标方法抛出的异常。 |
| @After          | 用于定义最终final 通知，不管是否异常，该通知都会执行。使用时需要指定一个value属性，该属性用于指定该通知被植入的切入点。                                                                                                                                            |
| @DeclareParents | 用于定义引介通知，相当于IntroductionInterceptor (不要求掌握)。                                                                                                                                                                                                     |


---

```Java
package com.study.spring.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.context.annotation.EnableAspectJAutoProxy;
import org.springframework.stereotype.Component;

/**
 * @author wanglongxin
 * @project spring study
 * @description
 * @date 2023/9/25 19:30:04
 */

@EnableAspectJAutoProxy
@Component
@Aspect
public class LogAspect {
    /**
     * define point cut.
     */
    @Pointcut("execution(* com.study.spring.service.*.*(..))")
    private void pointCutMethod() {
    }


    /**
     * 环绕通知.
     *
     * @param pjp pjp
     * @return obj
     * @throws Throwable exception
     */
    @Around("pointCutMethod()")
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("-----------------------");
        System.out.println("环绕通知: 进入方法");
        Object o = pjp.proceed();
        System.out.println("环绕通知: 退出方法");
        return o;
    }

    /**
     * 前置通知.
     */
    @Before("pointCutMethod()")
    public void doBefore() {
        System.out.println("前置通知");
    }


    /**
     * 后置通知.
     *
     * @param result return val
     */
    @AfterReturning(pointcut = "pointCutMethod()", returning = "result")
    public void doAfterReturning(String result) {
        System.out.println("后置通知, 返回值: " + result);
    }

    /**
     * 异常通知.
     *
     * @param e exception
     */
    @AfterThrowing(pointcut = "pointCutMethod()", throwing = "e")
    public void doAfterThrowing(Exception e) {
        System.out.println("异常通知, 异常: " + e.getMessage());
    }

    /**
     * 最终通知.
     */
    @After("pointCutMethod()")
    public void doAfter() {
        System.out.println("最终通知");
    }
}
```

