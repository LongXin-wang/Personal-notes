- [Maven 生命周期](#maven-生命周期)
  - [依赖管理](#依赖管理)
    - [依赖范围](#依赖范围)
    - [依赖传递](#依赖传递)
  - [依赖配置](#依赖配置)
    - [dependencies -  dependency](#dependencies----dependency)
    - [parent](#parent)
    - [dependencyManagement](#dependencymanagement)
    - [modules](#modules)
    - [properties](#properties)
  - [Maven 仓库](#maven-仓库)
- [Maven 使用](#maven-使用)
  - [pom.xml](#pomxml)
    - [基本配置](#基本配置)
    - [Maven坐标](#maven坐标)
  - [继承](#继承)
    - [父工程](#父工程)
    - [子工程](#子工程)
- [构建](#构建)
  - [build](#build)
    - [resources](#resources)
    - [plugins](#plugins)
- [Maven 的 settings 文件](#maven-的-settings-文件)

# Maven 生命周期

> https://javabetter.cn/maven/maven.html#%E4%BA%8C%E3%80%81maven-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E5%A4%A7%E7%9B%98%E7%82%B9

> https://ayayui.gitbooks.io/tutorialspoint-maven/content/book/maven_overview.html

总结一下 Maven 的优点，主要有以下 3 点：

- 依赖管理：Maven 能帮助我们解决软件包依赖的管理问题，不再需要提交大量的 jar 包、引入第三方库；
- 规范目录结构：Maven 标准的目录结构有助于项目构建的标准化，通过配置 profile 还可以根据不同的环境（开发环境、测试环境，生产环境）读取不同的配置文件；
- 方便集成：能够集成在 IDE 中更方便使用。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405171520568.png)

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405171520417.png)
****
## 依赖管理

### 依赖范围

其scope标签就是依赖范围的配置，默认是compile，可选配置有test、provided、runtime、system、import。

complie范围就是默认依赖范围，**此依赖范围对于编译、测试、运行三种classpath都有效。举个简单的例子，假如项目中有spring-core的依赖，那么spring-core不管是在编译，测试，还是运行都会被用到，因此spring-core必须是编译范围（构件**默认的是编译范围，所以依赖范围是编译范围的无须显示指定）

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405171521298.png)

### 依赖传递

Maven会解析各个直接依赖的POM，将那些必要的间接依赖，以传递性依赖的形式引入到当前的项目中

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405171523899.png)

**依赖可选**

项目中A依赖B，B依赖于X和Y，如果所有这三个的范围都是compile的话，那么X和Y就是A的compile范围的传递性依赖，但是如果我想X、Y不作为A的传递性依赖，不给它用的话，可以按照下面的方式配置可选依赖：

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405171524027.png)

```xml
<project>  
    <modelVersion>4.0.0</modelVersion>  
    <groupId>com.itwanger</groupId>  
    <artifactId>project-b</artifactId>  
    <version>1.0.0</version>  
    <dependencies>  
        <dependency>  
            <groupId>mysql</groupId>  
            <artifactId>mysql-connector-java</artifactId>  
            <version>5.1.10</version>  
            <optional>true</optional>  
        </dependency>  
        <dependency>  
            <groupId>postgresql</groupId>  
            <artifactId>postgresql</groupId>  
            <version>8.4-701.jdbc3</version>  
            <optional>true</optional>  
        </dependency>  
    </dependencies>  
</project>
```

**依赖排除**

有时候你引入的依赖中包含你不想要的依赖包，你想引入自己想要的，这时候就要用到排除依赖了，比如下图中spring-boot-starter-web自带了logback这个日志包，我想引入log4j2的，所以我先排除掉logback的依赖包，再引入想要的包就行了。

声明exclustion的时候只需要groupId和artifactId，不需要version元素，因为groupId和artifactId就能唯一定位某个依赖。

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<version>2.5.6</version>
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-logging</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<!-- 使用 log4j2 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-log4j2</artifactId>
	<version>2.5.6</version>
</dependency>
```

## 依赖配置

### dependencies -  dependency

- **groupId**, **artifactId**, **version** - 和基本配置中的 `groupId`、`artifactId`、`version` 意义相同。
- **type** - 对应 `packaging` 的类型，如果不使用 `type` 标签，maven 默认为 jar。
- **scope** - 此元素指的是任务的类路径（编译和运行时，测试等）以及如何限制依赖关系的传递性。有 5 种可用的限定范围：
    - **compile** - 如果没有指定 `scope` 标签，maven 默认为这个范围。编译依赖关系在所有 classpath 中都可用。此外，这些依赖关系被传播到依赖项目。
    - **provided** - 与 compile 类似，但是表示您希望 jdk 或容器在运行时提供它。它只适用于编译和测试 classpath，不可传递。
    - **runtime** - 此范围表示编译不需要依赖关系，而是用于执行。它是在运行时和测试 classpath，但不是编译 classpath。
    - **test** - 此范围表示正常使用应用程序不需要依赖关系，仅适用于测试编译和执行阶段。它不是传递的。
    - **system** - 此范围与 provided 类似，除了您必须提供明确包含它的 jar。该 artifact 始终可用，并且不是在仓库中查找。
- **systemPath** - 仅当依赖范围是系统时才使用。否则，如果设置此元素，构建将失败。该路径必须是绝对路径，因此建议使用 `propertie` 来指定特定的路径，如$ {java.home} / lib。由于假定先前安装了系统范围依赖关系，maven 将不会检查项目的仓库，而是检查库文件是否存在。如果没有，maven 将会失败，并建议您手动下载安装。
- **optional** - `optional` 让其他项目知道，当您使用此项目时，您不需要这种依赖性才能正常工作。
- **exclusions** - 包含一个或多个排除元素，每个排除元素都包含一个表示要排除的依赖关系的 `groupId` 和 `artifactId`。与可选项不同，可能或可能不会安装和使用，排除主动从依赖关系树中删除自己。

```Python
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
  ...
  <dependencies>
    <dependency>
     <groupId>org.apache.maven</groupId>
      <artifactId>maven-embedder</artifactId>
      <version>2.0</version>
      <type>jar</type>
      <scope>test</scope>
      <optional>true</optional>
      <exclusions>
        <exclusion>
          <groupId>org.apache.maven</groupId>
          <artifactId>maven-core</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
    ...
  </dependencies>
  ...
</project>
```

### parent

maven 支持继承功能。子 POM 可以使用 `parent` 指定父 POM ，然后继承其配置。

```Python
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>my-parent</artifactId>
    <version>2.0</version>
    <relativePath>../my-parent</relativePath>
  </parent>

  <artifactId>my-project</artifactId>
</project>

```

**relativePath** - 注意 `relativePath` 元素。在搜索本地和远程存储库之前，它不是必需的，但可以用作 maven 的指示符，以首先搜索给定该项目父级的路径。

### dependencyManagement

`dependencyManagement` 是表示依赖 jar 包的声明。即你在项目中的 `dependencyManagement` 下声明了依赖，maven 不会加载该依赖，`dependencyManagement` 声明可以被子 POM 继承。

`dependencyManagement` 的一个使用案例是当有父子项目的时候，父项目中可以利用 `dependencyManagement` 声明子项目中需要用到的依赖 jar 包，之后，当某个或者某几个子项目需要加载该依赖的时候，就可以在子项目中 `dependencies` 节点只配置 `groupId` 和 `artifactId` 就可以完成依赖的引用。

`dependencyManagement` 主要是为了统一管理依赖包的版本，确保所有子项目使用的版本一致，类似的还有`plugins`和`pluginManagement`。

### modules

```Python
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.codehaus.mojo</groupId>
  <artifactId>my-parent</artifactId>
  <version>2.0</version>
  <packaging>pom</packaging>

  <modules>
    <module>my-project</module>
    <module>another-project</module>
    <module>third-project/pom-example.xml</module>
  </modules>
</project>
```

### properties

```Python
<project>
  ...
  <properties>
    <maven.compiler.source>1.7<maven.compiler.source>
    <maven.compiler.target>1.7<maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
  </properties>
  ...
</project>
```

## Maven 仓库

**Maven 本地仓库**

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405171542836.png)

***Maven 远程仓库 & 私服**

默认情况下，本地仓库是被注释掉的，也就是空的，那么就必须得给 Maven 配置一个可用的远程仓库，否则 Maven 在 build（构建）的时候就无法去下载依赖。

中央仓库就是这样一个可用的远程仓库，里面包含了这个世界上绝大多数流行的开源 Java 类库，以及源码、作者信息、许可证信息等等。

不过，默认的中央仓库访问速度比较慢，通常我们会选择使用阿里的 Maven 远程仓库。

```xml
<repositories>
	<repository>
		<id>ali-maven</id>
		<url>http://maven.aliyun.com/nexus/content/groups/public</url>
		<releases>
			<enabled>true</enabled>
		</releases>
		<snapshots>
			<enabled>true</enabled>
			<updatePolicy>always</updatePolicy>
			<checksumPolicy>fail</checksumPolicy>
		</snapshots>
	</repository>
</repositories>
```

**仓库镜像**

如果仓库X可以提供仓库Y存储的所有内容，那么就可以认为X是Y的一个镜像。通常我们会在 settings.xml 文件中添加阿里云镜像：

通过 https://developer.aliyun.com/mvn/search 可以查看阿里云镜像 Maven 的地址

```xml
<mirrors>
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>        
    </mirror>
  </mirrors>
```

# Maven 使用

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <!-- The Basics -->
  <groupId>...</groupId>
  <artifactId>...</artifactId>
  <version>...</version>
  <packaging>...</packaging>
  <dependencies>...</dependencies>
  <parent>...</parent>
  <dependencyManagement>...</dependencyManagement>
  <modules>...</modules>
  <properties>...</properties>

  <!-- Build Settings -->
  <build>...</build>
  <reporting>...</reporting>

  <!-- More Project Information -->
  <name>...</name>
  <description>...</description>
  <url>...</url>
  <inceptionYear>...</inceptionYear>
  <licenses>...</licenses>
  <organization>...</organization>
  <developers>...</developers>
  <contributors>...</contributors>

  <!-- Environment Settings -->
  <issueManagement>...</issueManagement>
  <ciManagement>...</ciManagement>
  <mailingLists>...</mailingLists>
  <scm>...</scm>
  <prerequisites>...</prerequisites>
  <repositories>...</repositories>
  <pluginRepositories>...</pluginRepositories>
  <distributionManagement>...</distributionManagement>
  <profiles>...</profiles>
</project>
```

## pom.xml

POM：**P**roject **O**bject **M**odel，项目对象模型。和 POM 类似的是：DOM（Document Object Model），文档对象模型。它们都是模型化思想的具体体现。

```xml

<!-- 当前Maven工程的坐标 -->
<groupId>com.example</groupId>
<artifactId>demo</artifactId>
<version>0.0.1-SNAPSHOT</version>
<name>demo</name>
<description>Demo project for Spring Boot</description>
<!-- 当前Maven工程的打包方式，可选值有下面三种： -->
<!-- jar：表示这个工程是一个Java工程  -->
<!-- war：表示这个工程是一个Web工程 -->
<!-- pom：表示这个工程是“管理其他工程”的工程 -->
<packaging>jar</packaging>
<properties>
    <!-- 工程构建过程中读取源码时使用的字符集 -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
<!-- 当前工程所依赖的jar包 -->
<dependencies>
    <!-- 使用dependency配置一个具体的依赖 -->
    <dependency>
        <!-- 在dependency标签内使用具体的坐标依赖我们需要的一个jar包 -->
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <!-- scope标签配置依赖的范围 -->
        <scope>test</scope>
    </dependency>
</dependencies>

```

### 基本配置

- **project** - `project` 是 pom.xml 中描述符的根。
- **modelVersion** - `modelVersion` 指定 pom.xml 符合哪个版本的描述符。maven 2 和 3 只能为 4.0.0。

一般 jar 包被识别为： `groupId:artifactId:version` 的形式。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>org.codehaus.mojo</groupId>
  <artifactId>my-project</artifactId>
  <version>1.0</version>
  <packaging>war</packaging>
</project>
```

### Maven坐标

**在 maven 中，根据** **`groupId`****、**`artifactId`**、**`version` **组合成** **`groupId:artifactId:version`** **来唯一识别一个 jar 包。**

- **groupId** - 团体、组织的标识符。团体标识的约定是，它以创建这个项目的组织名称的逆向域名(reverse domain name)开头。一般对应着 java 的包结构。
- **artifactId** - 单独项目的唯一标识符。比如我们的 tomcat、commons 等。不要在 artifactId 中包含点号(.)。
- **version** - 一个项目的特定版本。
    - maven 有自己的版本规范，一般是如下定义 major version、minor version、incremental version-qualifier ，比如 1.2.3-beta-01。要说明的是，maven 自己判断版本的算法是 major、minor、incremental 部分用数字比较，qualifier 部分用字符串比较，所以要小心 alpha-2 和 alpha-15 的比较关系，最好用 alpha-02 的格式。
    - maven 在版本管理时候可以使用几个特殊的字符串 SNAPSHOT、LATEST、RELEASE。比如 `1.0-SNAPSHOT`。各个部分的含义和处理逻辑如下说明：
        - **SNAPSHOT** - 这个版本一般用于开发过程中，表示不稳定的版本。
        - **LATEST** - 指某个特定构件的最新发布，这个发布可能是一个发布版，也可能是一个 snapshot 版，具体看哪个时间最后。
        - **RELEASE** ：指最后一个发布版。
- **packaging** - 项目的类型，描述了项目打包后的输出，默认是 jar。常见的输出类型为：pom, jar, maven-plugin, ejb, war, ear, rar, par。

## 继承

Maven工程之间，A 工程继承 B 工程

- B 工程：父工程
- A 工程：子工程

本质上是 A 工程的 pom.xml 中的配置继承了 B 工程中 pom.xml 的配置。

- 在父工程中配置依赖的统一管理   使用`dependencyManagement`标签配置对依赖的管理  而实际上**被管理的依赖并没有真正被引入到工程**。
- 子工程中引用那些被父工程管理的依赖

### 父工程

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>net.javatv.maven</groupId>
    <artifactId>maven-demo-parent</artifactId>
    <!-- 当前工程作为父工程，它要去管理子工程，所以打包方式必须是 pom -->
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>

    <modules>
        <module>demo-module</module>
    </modules>

    <!-- 通过自定义属性，统一指定Spring的版本 -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

        <!-- 自定义标签，维护Spring版本数据 -->
        <spring.version>5.3.19</spring.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-beans</artifactId>
                <version>${spring.version}</version>
            </dependency>

            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-core</artifactId>
                <version>${spring.version}</version>
            </dependency>

            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>${spring.version}</version>
            </dependency>

            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aop</artifactId>
                <version>${spring.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

</project>

```

### 子工程

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <!-- 使用parent标签指定当前工程的父工程 -->
    <parent>
        <artifactId>maven-demo-parent</artifactId>
        <groupId>net.javatv.maven</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <!-- 子工程的坐标 -->
    <!-- 如果子工程坐标中的groupId和version与父工程一致，那么可以省略 -->
    <artifactId>demo-module</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
        </dependency>
    </dependencies>
    
</project>
```

# 构建

## build

build 可以分为 "project build" 和 "profile build"

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
  ...
  <!-- "Project Build" contains more elements than just the BaseBuild set -->
  <build>...</build>

  <profiles>
    <profile>
      <!-- "Profile Build" contains a subset of "Project Build"s elements -->
      <build>...</build>
    </profile>
  </profiles>
</project>
```

基本构建配置：

目标或阶段。如果给出了一个目标，它应该被定义为它在命令行中（如 jar：jar）。如果定义了一个阶段（如安装），也是如此。

**directory** ：构建时的输出路径。默认为：`${basedir}/target` 。

**finalName** ：这是项目的最终构建名称（不包括文件扩展名，例如：my-project-1.0.jar）

**filter** ：定义 `* .properties` 文件，其中包含适用于接受其设置的资源的属性列表（如下所述）。换句话说，过滤器文件中定义的“name = value”对在代码中替换$ {name}字符串。

```Xml
<build>
  <defaultGoal>install</defaultGoal>
  <directory>${basedir}/target</directory>
  <finalName>${artifactId}-${version}</finalName>
  <filters>
    <filter>filters/filter1.properties</filter>
  </filters>
  ...
</build>

```

### resources

资源的配置。资源文件通常不是代码，不需要编译，而是在项目需要捆绑使用的内容。

- **resources**: 资源元素的列表，每个资源元素描述与此项目关联的文件和何处包含文件。
- **targetPath**: 指定从构建中放置资源集的目录结构。目标路径默认为基本目录。将要包装在 jar 中的资源的通常指定的目标路径是 META-INF。
- **filtering**: 值为 true 或 false。表示是否要为此资源启用过滤。请注意，该过滤器 `* .properties` 文件不必定义为进行过滤 - 资源还可以使用默认情况下在 POM 中定义的属性（例如$ {project.version}），并将其传递到命令行中“-D”标志（例如，“-Dname = value”）或由 properties 元素显式定义。过滤文件覆盖上面。
- **directory**: 值定义了资源的路径。构建的默认目录是`${basedir}/src/main/resources`。
- **includes**: 一组文件匹配模式，指定目录中要包括的文件，使用*作为通配符。
- **excludes**: 与 `includes` 类似，指定目录中要排除的文件，使用*作为通配符。注意：如果 `include` 和 `exclude` 发生冲突，maven 会以 `exclude` 作为有效项。
- **testResources**: `testResources` 与 `resources` 功能类似，区别仅在于：`testResources` 指定的资源仅用于 test 阶段，并且其默认资源目录为：`${basedir}/src/test/resources` 。

```Xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <build>
    ...
    <resources>
      <resource>
        <targetPath>META-INF/plexus</targetPath>
        <filtering>false</filtering>
        <directory>${basedir}/src/main/plexus</directory>
        <includes>
          <include>configuration.xml</include>
        </includes>
        <excludes>
          <exclude>**/*.properties</exclude>
        </excludes>
      </resource>
    </resources>
    <testResources>
      ...
    </testResources>
    ...
  </build>
</project>

```

### plugins

- **groupId**, **artifactId**, **version** ：和基本配置中的 `groupId`、`artifactId`、`version` 意义相同。
- **extensions** ：值为 true 或 false。是否加载此插件的扩展名。默认为 false。
- **inherited** ：值为 true 或 false。这个插件配置是否应该适用于继承自这个插件的 POM。默认值为 true。
- **configuration** - 这是针对个人插件的配置，这里不扩散讲解。
- **dependencies** ：这里的 `dependencies` 是插件本身所需要的依赖。
- **executions** ：需要记住的是，插件可能有多个目标。每个目标可能有一个单独的配置，甚至可能将插件的目标完全绑定到不同的阶段。执行配置插件的目标的执行。
    - **id**: 执行目标的标识。
    - **goals**: 像所有多元化的 POM 元素一样，它包含单个元素的列表。在这种情况下，这个执行块指定的插件目标列表。
    - **phase**: 这是执行目标列表的阶段。这是一个非常强大的选项，允许将任何目标绑定到构建生命周期中的任何阶段，从而改变 maven 的默认行为。
    - **inherited**: 像上面的继承元素一样，设置这个 false 会阻止 maven 将这个执行传递给它的子代。此元素仅对父 POM 有意义。
    - **configuration**: 与上述相同，但将配置限制在此特定目标列表中，而不是插件下的所有目标。

```Xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <build>
    ...
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <version>2.6</version>
        <extensions>false</extensions>
        <inherited>true</inherited>
        <configuration>
          <classifier>test</classifier>
        </configuration>
        <dependencies>...</dependencies>
        <executions>...</executions>
      </plugin>
    </plugins>
  </build>
</project>
```

# Maven 的 settings 文件

