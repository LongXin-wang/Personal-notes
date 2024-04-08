- [java.util.logging 的使用方式](#javautillogging-的使用方式)
- [Log4j 的使用方式](#log4j-的使用方式)

#  java.util.logging 的使用方式

```java
package com.itwanger;

import java.io.IOException;
import java.util.logging.FileHandler;
import java.util.logging.Logger;
import java.util.logging.SimpleFormatter;

/**
 * @author 微信搜「沉默王二」，回复关键字 PDF
 */
public class JavaUtilLoggingDemo {
    public static void main(String[] args) throws IOException {
        Logger logger = Logger.getLogger("test");
        FileHandler fileHandler = new FileHandler("javautillog.txt");
        fileHandler.setFormatter(new SimpleFormatter());
        logger.addHandler(fileHandler);
        logger.info("细小的信息");
    }
}
```

# Log4j 的使用方式

第一步，在 pom.xml 文件中引入 Log4j 包：

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

第二步，在 resources 目录下创建 log4j.properties 文件，内容如下所示：

```xml
# 指定输出级别、输出地点
log4j.rootLogger = debug,stdout,D,E

输出信息到控制台
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n

输出DEBUG 级别以上的日志到=debug.log
log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
log4j.appender.D.File = debug.log
log4j.appender.D.Append = true
log4j.appender.D.Threshold = DEBUG 
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = %d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

输出ERROR 级别以上的日志到=error.log
log4j.appender.E = org.apache.log4j.DailyRollingFileAppender
log4j.appender.E.File =error.log 
log4j.appender.E.Append = true
log4j.appender.E.Threshold = ERROR 
log4j.appender.E.layout = org.apache.log4j.PatternLayout
log4j.appender.E.layout.ConversionPattern = %d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

```

第三步，写个使用 Demo：

```java
package com.itwanger;

import org.apache.log4j.LogManager;
import org.apache.log4j.Logger;

/**
 * @author 微信搜「沉默王二」，回复关键字 PDF
 */
public class Log4jDemo {
    private static final Logger logger = LogManager.getLogger(Log4jDemo.class);

    public static void main(String[] args) {
        // 记录debug级别的信息
        logger.debug("debug.");

        // 记录info级别的信息
        logger.info("info.");

        // 记录error级别的信息
        logger.error("error.");
    }
}


2020-10-20 20:53:27  [ main:0 ] - [ DEBUG ]  debug.
2020-10-20 20:53:27  [ main:3 ] - [ INFO ]  info.
2020-10-20 20:53:27  [ main:3 ] - [ ERROR ]  error.
```
