## log4j2日志配置介绍

#### 1.日志级别

all:  最高等级的，用于关闭所有日志记录

trace：追踪，就是程序推进一下，可以写个trace输出

debug：调试，一般作为最低级别，trace基本不用

info：输出重要的信息，使用较多

warn：警告，有些信息不是错误信息，但也要给程序员一些提示。

error：错误信息。用的也很多

fatal：致命错误。级别较高，这种级别不用调试了，重写吧

off： 关闭，不输出任何级别的日志

机制：如果一条日志信息的级别大于等于配置文件的级别，就记录

#### 2.输出源

CONSOLE（输出到控制台）、FILE（输出到文件）等。

#### 3.布局方式

SimpleLayout：以简单的形式显示

HTMLLayout：以HTML表格显示

PatternLayout：自定义形式显示

在Log4J2中基本采用PatternLayout自定义日志布局。

自定义格式：

%t：线程名称

%p：日志级别

%c：日志消息所在类名

%m：消息内容

%M：输出执行方法

%d：发生时间，%d{yyyy-MM-dd HH:mm:ss,SSS}，输出类似：2011-10-18 22:10:28,921

%x: 输出和当前线程相关联的NDC(嵌套诊断环境),尤其用到像java servlets这样的多客户多线程的应用中。

%L：代码中的行数

%n：换行

#### 4.配置使用

###### （1）jar包转换关系

![img](https://segmentfault.com/img/bVbni3r?w=1152&h=636)

下载了slf4j的压缩包后，这里下的版本是slf4j-1.7.25.zip，里面有很多jar文件。

slf4j的核心包是slf4j-api-1.7.25.jar，需要配合日志实现，才能将日志绑定另一种输出。我们先看看slf4j里的jar包的作用.

```tiki wiki
jcl-over-slf4j.jar  -->    (jcl    -> slf4j)    将Jakarta Commons Logging日志框架到 slf4j 的桥接  
jul-to-slf4j.jar    -->    (juc    -> slf4j)    将java.util.logging的日志桥接到 slf4j  
log4j-over-slf4j.jar-->    (log4j  -> slf4j)    将log4j 的日志，桥接到slf4j  
osgi-over-slf4j.jar -->    (osgi   -> slf4j)    将osgi环境下的日志，桥接到slf4j  
slf4j-android.jar   -->    (android-> slf4j)    将android环境下的日志，桥接到slf4j  
slf4j-api.jar       -->                         slf4j 的api接口jar包  
slf4j-ext.jar       -->                         扩展功能  
slf4j-jcl.jar       -->    (lf4j -> jcl )       slf4j 转接到 Jakarta Commons Logging日志输出框架  
slf4j-jdk14.jar     -->    (slf4j -> jul )      slf4j 转接到 java.util.logging，所以这个包不能和jul-to-slf4j.jar同时用，否则会死循环！！  
slf4j-log4j12.jar   -->    (slf4j -> log4j)     slf4j 转接到 log4j,所以这个包不能和log4j-over-slf4j.jar同时用，否则会死循环！！  
slf4j-migrator.jar  -->                         一个GUI工具，支持将项目代码中 JCL,log4j,java.util.logging的日志API转换为slf4j的写法  
slf4j-nop.jar       -->    (slf4j -> null)      slf4j的空接口输出绑定，丢弃所有日志输出  
slf4j-simple.jar    -->    (slf4j -> slf4j-simple ) slf4j的自带的简单日志输出接口  
```

这样整理之后，slf4j的jar包用途就比较清晰了。目前个人遇到的项目里，一般还会有log4j和log4j2混用的问题。因为log4j项目已经停止更新了，官方建议用log4j2。 然后，log4j2里也提供了对各类log的桥接支持，这里就只列举相关的几个jar包说明。

```tiki wiki
log4j-1.2-api.jar   -->    (log4j  -> log4j2)  将log4j 的日志转接到log4j2日志框架  
log4j-api.jar       -->    log4j2的api接口jar包  
log4j-core.jar      -->    (log4j2 -> log4j-core) log4j2的日志输出核心jar包  
log4j-slf4j-impl.jar-->    (slf4j  -> log4j2)  slf4j 转接到 log4j2 的日志输出框架  (不能和 log4j-to-slf4j同时用)  
log4j-to-slf4j.jar  -->    (log4j2 -> slf4j)   将 log4j2的日志桥接到 slf4j  (不能和 log4j-slf4j-impl 同时用)  
```

从这里就已经可以看到，日志框架之间的关系有点乱。
 因为log4j2和slf4j都能对接多种日志框架，所以这些包的依赖，作用，还有命名，都容易让人混淆。

###### （2）配置文件命名及位置

log4j 2.x版本不再支持像1.x中的.properties后缀的文件配置方式，2.x版本配置文件后缀名只能为".xml",".json"或者".jsn".

系统选择配置文件的优先级(从先到后)如下：

```tex
1.classpath下的名为log4j2-test.json 或者log4j2-test.jsn的文件.

2.classpath下的名为log4j2-test.xml的文件.

3.classpath下名为log4j2.json 或者log4j2.jsn的文件.

4.classpath下名为log4j2.xml的文件.
```

#### 5.配置文件节点解析

(1).根节点Configuration有两个属性:status和monitorinterval,有两个子节点:Appenders和Loggers(表明可以定义多个Appender和Logger).

        status:Configuration的属性，用来指定log4j本身的打印日志的级别.
    
        monitorinterval:Configuration的属性，用于指定log4j自动重新配置的监测间隔时间，单位是s,最小是5s.

(2).Appenders节点，常见的有三种子节点:Console、RollingFile、File.

    Console节点用来定义输出到控制台的Appender.
    
        name:Console的属性，指定Appender的名字.
    
        target:Console的属性，SYSTEM_OUT 或 SYSTEM_ERR,一般只设置默认:SYSTEM_OUT.
    
        PatternLayout:Console的子节点，输出格式，不设置默认为:%m%n.
    
    File节点用来定义输出到指定位置的文件的Appender.
    
        name:File的属性，指定Appender的名字.
    
        fileName:File的属性，指定输出日志的目的文件带全路径的文件名.
    
        PatternLayout:File的子节点，输出格式，不设置默认为:%m%n.
    
    RollingFile节点用来定义超过指定大小自动删除旧的创建新的的Appender.
    
        name:RollingFilede的属性，指定Appender的名字.
    
        fileName:RollingFilede的属性，指定输出日志的目的文件带全路径的文件名.
    
        filePattern:RollingFilede的属性，历史日志封存路径。其中%d{yyyyMMddHH}表示了封存历史日志的时间单位（目前单位为小时，yyyy表示年，MM表示月，dd表示天，HH表示小时，mm表示分钟，ss表示秒，SS表示毫秒）log4j2自动识别zip等后缀，表示历史日志需要压缩。
    
        PatternLayout:RollingFilede的子节点，输出格式，不设置默认为:%m%n.
    
        DefaultRolloverStrategy:RollingFilede的子节点，用来指定同一个文件夹下最多有几个日志文件时开始删除最旧的，创建新的(通过max属性)。
    
        Policies:RollingFilede的子节点，指定滚动日志的策略，就是什么时候进行新建日志文件输出日志.
    
            TimeBasedTriggeringPolicy:Policies子节点，基于时间的滚动策略，interval属性用来指定多久滚动一次，默认是1hour。modulate=true用来调整时间：比如现在是早上3am，interval是4，那么第一次滚动是在4am，接着是8am，12am...而不是7am.
    
            SizeBasedTriggeringPolicy:Policies子节点，基于指定文件大小的滚动策略，size属性用来定义每个日志文件的大小.
    
        Filters:RollingFilede的子节点，决定日志事件能否被输出。过滤条件有三个值：ACCEPT(接受), DENY(拒绝) or NEUTRAL(中立)(后面具体讲)
    
        ACCEP和DENY比较好理解就是接受和拒绝的意思，在使用单个过滤器的时候，一般就是使用这两个值。但是在组合过滤器中，如果用接受ACCEPT的话，日志信息就会直接写入日志文件,后续的过滤器不再进行过滤。所以，在组合过滤器中，接受使用NEUTRAL（中立），被第一个过滤器接受的日志信息，会继续用后面的过滤器进行过滤，只有符合所有过滤器条件的日志信息，才会被最终写入日志文件。
    
            ThresholdFilter:Filters的子节点
    
                level:ThresholdFilterde的属性，将被过滤的级别。
    
                onMatch:ThresholdFilterde的属性，默认值是NEUTRAL
    
                onMismatch:ThresholdFilterde的属性，默认是DENY

(3).Loggers节点，常见的有两种:Root和Logger.

    Root节点用来指定项目的根日志，如果没有单独指定Logger，那么就会默认使用该Root日志输出
    
        level:Root属性，日志输出级别，共有8个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < Fatal < OFF.
    
        appender-ref：Root的子节点，用来指定该日志输出到哪个Appender，通过ref指定.
    
    Logger节点用来单独指定日志的形式，比如要为指定包下的class指定不同的日志级别等。
    
        level:Logger属性，日志输出级别，共有8个级别，按照从低到高为：All < Trace < Debug < Info < Warn < Error < Fatal < OFF.
    
        name:Logger属性，用来指定该Logger所适用的类或者类所在的包全路径,继承自Root节点.
    
        additivity:Logger属性

appender-ref:Logger的子节点，用来指定该日志输出到哪个Appender,如果没有指定，就会默认继承自Root.如果指定了，那么会在指定的这个Appender和Root的Appender中都会输出，此时我们可以设置Logger的additivity="false"只在自定义的Appender中进行输出。

#### 6.xml典型例

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
    status : 这个用于设置log4j2自身内部的信息输出,可以不设置,当设置成trace时,会看到log4j2内部各种详细输出
    monitorInterval : Log4j能够自动检测修改配置文件和重新配置本身, 设置间隔秒数。

    注：本配置文件的目标是将不同级别的日志输出到不同文件，最大2MB一个文件，
    文件数据达到最大值时，旧数据会被压缩并放进指定文件夹
-->
<Configuration status="WARN" monitorInterval="600">

    <Properties>
        <!-- 配置日志文件输出目录，此配置将日志输出到tomcat根目录下的指定文件夹 -->
        <Property name="LOG_HOME">${sys:catalina.home}/WebAppLogs/SSHExample</Property>
    </Properties>

    <Appenders>

        <!--这个输出控制台的配置，这里输出除了warn和error级别的信息到System.out-->
        <Console name="console_out_appender" target="SYSTEM_OUT">
            <!-- 控制台只输出level及以上级别的信息(onMatch),其他的直接拒绝(onMismatch) -->
            <ThresholdFilter level="warn" onMatch="DENY" onMismatch="ACCEPT"/>  
            <!-- 输出日志的格式 -->
            <PatternLayout pattern="%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n"/>
        </Console>
        <!--这个输出控制台的配置，这里输出warn和error级别的信息到System.err，在eclipse控制台上看到的是红色文字-->
        <Console name="console_err_appender" target="SYSTEM_ERR">
            <!-- 控制台只输出level及以上级别的信息(onMatch),其他的直接拒绝(onMismatch) -->
            <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/>
            <!-- 输出日志的格式 -->
            <PatternLayout pattern="%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n"/>
        </Console>

        <!-- TRACE级别日志 -->
        <!-- 设置日志格式并配置日志压缩格式，压缩文件独立放在一个文件夹内，
        日期格式不能为冒号，否则无法生成，因为文件名不允许有冒号，此appender只输出trace级别的数据到trace.log -->
        <RollingRandomAccessFile name="trace_appender"
                                 immediateFlush="true" fileName="${LOG_HOME}/trace.log"
                                 filePattern="${LOG_HOME}/trace/trace - %d{yyyy-MM-dd HH_mm_ss}.log.gz">
            <PatternLayout>
                <pattern>%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n</pattern>
            </PatternLayout>
            <Policies><!-- 两个配置任选其一 -->

                <!-- 每个日志文件最大2MB -->
                <SizeBasedTriggeringPolicy size="2MB"/>

            </Policies>
            <Filters><!-- 此Filter意思是，只输出debug级别的数据 -->
                <!-- DENY，日志将立即被抛弃不再经过其他过滤器；
                       NEUTRAL，有序列表里的下个过滤器过接着处理日志；
                       ACCEPT，日志会被立即处理，不再经过剩余过滤器。 -->
                <ThresholdFilter level="debug" onMatch="DENY" onMismatch="NEUTRAL"/>  
                <ThresholdFilter level="trace" onMatch="ACCEPT" onMismatch="DENY"/> 
            </Filters>
        </RollingRandomAccessFile>

        <!-- DEBUG级别日志 -->
        <!-- 设置日志格式并配置日志压缩格式，压缩文件独立放在一个文件夹内，
        日期格式不能为冒号，否则无法生成，因为文件名不允许有冒号，此appender只输出debug级别的数据到debug.log -->
        <RollingRandomAccessFile name="debug_appender"
                                 immediateFlush="true" fileName="${LOG_HOME}/debug.log"
                                 filePattern="${LOG_HOME}/debug/debug - %d{yyyy-MM-dd HH_mm_ss}.log.gz">
            <PatternLayout>
                <pattern>%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n</pattern>
            </PatternLayout>
            <Policies><!-- 两个配置任选其一 -->

                <!-- 每个日志文件最大2MB -->
                <SizeBasedTriggeringPolicy size="2MB"/>

                <!-- 如果启用此配置，则日志会按文件名生成新压缩文件，
                即如果filePattern配置的日期格式为 %d{yyyy-MM-dd HH} ，则每小时生成一个压缩文件，
                如果filePattern配置的日期格式为 %d{yyyy-MM-dd} ，则天生成一个压缩文件 -->
<!--                 <TimeBasedTriggeringPolicy interval="1" modulate="true" /> -->

            </Policies>
            <Filters><!-- 此Filter意思是，只输出debug级别的数据 -->
                <!-- DENY，日志将立即被抛弃不再经过其他过滤器；
                       NEUTRAL，有序列表里的下个过滤器过接着处理日志；
                       ACCEPT，日志会被立即处理，不再经过剩余过滤器。 -->
                <ThresholdFilter level="info" onMatch="DENY" onMismatch="NEUTRAL"/>  
                <ThresholdFilter level="debug" onMatch="ACCEPT" onMismatch="DENY"/> 
            </Filters>
        </RollingRandomAccessFile>

        <!-- INFO级别日志 -->
        <RollingRandomAccessFile name="info_appender"
                                 immediateFlush="true" fileName="${LOG_HOME}/info.log"
                                 filePattern="${LOG_HOME}/info/info - %d{yyyy-MM-dd HH_mm_ss}.log.gz">
            <PatternLayout>
                <pattern>%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n</pattern>
            </PatternLayout>
            <Policies>
                <SizeBasedTriggeringPolicy size="2MB"/>
            </Policies>
            <Filters>
                <ThresholdFilter level="warn" onMatch="DENY" onMismatch="NEUTRAL"/>  
                <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/> 
            </Filters>
        </RollingRandomAccessFile>

        <!-- WARN级别日志 -->
        <RollingRandomAccessFile name="warn_appender"
                                 immediateFlush="true" fileName="${LOG_HOME}/warn.log"
                                 filePattern="${LOG_HOME}/warn/warn - %d{yyyy-MM-dd HH_mm_ss}.log.gz">
            <PatternLayout>
                <pattern>%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n</pattern>
            </PatternLayout>
            <Policies>
                <SizeBasedTriggeringPolicy size="2MB"/>
            </Policies>
            <Filters>
                <ThresholdFilter level="error" onMatch="DENY" onMismatch="NEUTRAL"/>  
                <ThresholdFilter level="warn" onMatch="ACCEPT" onMismatch="DENY"/> 
              </Filters>
        </RollingRandomAccessFile>

        <!-- ERROR级别日志 -->
        <RollingRandomAccessFile name="error_appender"
                                 immediateFlush="true" fileName="${LOG_HOME}/error.log"
                                 filePattern="${LOG_HOME}/error/error - %d{yyyy-MM-dd HH_mm_ss}.log.gz">
            <PatternLayout>
                <pattern>%5p [%t] %d{yyyy-MM-dd HH:mm:ss} (%F:%L) %m%n</pattern>
            </PatternLayout>
            <Policies>
                <SizeBasedTriggeringPolicy size="2MB"/>
            </Policies>
            <Filters>
                <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
              </Filters>
        </RollingRandomAccessFile>
    </Appenders>

    <Loggers>
        <!-- 配置日志的根节点 -->
        <root level="trace">
            <appender-ref ref="console_out_appender"/>
            <appender-ref ref="console_err_appender"/>
            <appender-ref ref="trace_appender"/>
            <appender-ref ref="debug_appender"/>
            <appender-ref ref="info_appender"/>
            <appender-ref ref="warn_appender"/>
            <appender-ref ref="error_appender"/>
        </root>

        <!-- 第三方日志系统 -->
        <logger name="org.springframework.core" level="info"/>
        <logger name="org.springframework.beans" level="info"/>
        <logger name="org.springframework.context" level="info"/>
        <logger name="org.springframework.web" level="info"/>
        <logger name="org.jboss.netty" level="warn"/>
        <logger name="org.apache.http" level="warn"/>

    </Loggers>

</Configuration>
```



