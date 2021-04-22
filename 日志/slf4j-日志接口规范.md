>很多时候我们做项目都是从简单到复杂，也就是我们很可能一开始使用的是 JDKLog，之后业务复杂了需要使用 Log4J，这时候我们如何将原来写好的日志用新的日志框架输出呢？
>
>为了避免切换日志组件时要改动代码，这时候一个叫做 ==SLF4J==（Simple Logging Facade for Java，即Java简单日志记录接口集）的东西出现了。
>
>SLF4J 是一个==日志的接口规范==，它对用户提供了统一的日志接口，屏蔽了不同日志组件的差异。这样我们在编写代码的时候只需要看 SLF4J 这个接口文档即可，不需要去理会不同日志框架的区别。而当我们需要更换日志组件的时候，只需要==更换一个具体的日志组件Jar包==就可以了。

#### 1. slf4j+jdklog

>```xml
><dependency>
>    <groupId>org.slf4j</groupId>
>    <artifactId>slf4j-api</artifactId>
>    <version>1.7.21</version>
></dependency>
>
><dependency>
>    <groupId>org.slf4j</groupId>
>    <artifactId>slf4j-jdk14</artifactId>
>    <version>1.7.21</version>
></dependency>
>```
>
>

#### 2. slf4j+log4j

>需要依赖的jar包：slf4j-api.jar、slf4j-log4j12.jar、log4j.jar
>
>```xml
><dependency>
>    <groupId>org.slf4j</groupId>
>    <artifactId>slf4j-api</artifactId>
>    <version>1.7.21</version>
></dependency>
>
><dependency>
>    <groupId>org.slf4j</groupId>
>    <artifactId>slf4j-log4j12</artifactId>
>    <version>1.7.21</version>
></dependency>
>
><dependency>
>    <groupId>log4j</groupId>
>    <artifactId>log4j</artifactId>
>    <version>1.2.17</version>
></dependency>
>```
>
>```xml
><?xml version="1.0" encoding="UTF-8"?>
><!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">
>
><log4j:configuration xmlns:log4j='http://jakarta.apache.org/log4j/' >
>
>    <appender name="myConsole" class="org.apache.log4j.ConsoleAppender">
>        <layout class="org.apache.log4j.PatternLayout">
>            <param name="ConversionPattern"
>                   value="[%d{dd HH:mm:ss,SSS\} %-5p] [%t] %c{2\} - %m%n" />
>        </layout>
>        <!--过滤器设置输出的级别-->
>        <filter class="org.apache.log4j.varia.LevelRangeFilter">
>            <param name="levelMin" value="debug" />
>            <param name="levelMax" value="error" />
>            <param name="AcceptOnMatch" value="true" />
>        </filter>
>    </appender>
>
>    <!-- 根logger的设置-->
>    <root>
>        <priority value ="debug"/>
>        <appender-ref ref="myConsole"/>
>    </root>
></log4j:configuration>
>```
>
>

#### 3. slf4j+logback

>```xml
><dependency>
>    <groupId>org.slf4j</groupId>
>    <artifactId>slf4j-api</artifactId>
>    <version>1.7.21</version>
></dependency>
>
><dependency>
>    <groupId>ch.qos.logback</groupId>
>    <artifactId>logback-classic</artifactId>
>    <version>1.1.7</version>
></dependency>
>
><dependency>
>    <groupId>ch.qos.logback</groupId>
>    <artifactId>logback-core</artifactId>
>    <version>1.1.7</version>
></dependency>
>```
>
>```xml
><?xml version="1.0" encoding="UTF-8"?>
><configuration>
>    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
>        <layout class="ch.qos.logback.classic.PatternLayout">
>            <Pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</Pattern>
>        </layout>
>    </appender>
>    <logger name="com.chanshuyi" level="TRACE"/>
>
>    <root level="warn">
>        <appender-ref ref="STDOUT" />
>    </root>
></configuration>
>```
>
>

