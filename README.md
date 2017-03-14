# Spring Boot Logging

Logging is integral part of any application. I had setup my spring boot app and talking a step back to look at the basic stuff. Spring boot provides basic logging and in here I had to log all the messages in `JSON` format so that the messages are consumed by Splunk downstream.

## Starting points
Going back to [Spring docs](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html), here is what I found. 
Additional logging at [Spring boot logging](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-logging.html)

## Libraries discourse:

### Logback
Logback is 10x times faster than log4j and natively implements slf4j. At present time, logback is divided into three modules, *logback-core*, *logback-classic* and *logback-access*.
The **logback-core** module lays the groundwork for the other two modules. The **logback-classic** module can be assimilated to a significantly improved version of log4j. Moreover, **logback-classic** natively implements the SLF4J API so that you can readily switch back and forth between logback and other logging frameworks such as log4j or java.util.logging (JUL).
The **logback-access** module integrates with Servlet containers, such as Tomcat and Jetty, to provide HTTP-access log functionality. Note that you could easily build your own module on top of **logback-core**.
### Log4j 2
Apache Log4j 2 is an upgrade to Log4j that provides significant improvements over its predecessor, Log4j 1.x, and provides many of the improvements available in Logback while fixing some inherent problems in Logback’s architecture.
### SLF4J
The Simple Logging Facade for Java (SLF4J) serves as a simple facade or abstraction for various logging frameworks (e.g. `java.util.logging`, `logback`, `log4j`) allowing the end user to plug in the desired logging framework at deployment time.
### Logstash
Logstash is an open source, server-side data processing pipeline that ingests data from a multitude of sources simultaneously, transforms it, and then sends it to your favorite “stash.” 

## Prerequisite
Going by the doc, `jcl-over-slf4j` is a required module. It is a transitive dependency of `spring-boot-starter-logging`. I had this in my pom file by the XML dependency below.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```
Once I included `spring-boot-starter-logging`, the transitive dependencies included are `logback-classic`, `jcl-over-slf4j`, `jul-to-slf-4j` and `log4j-over-slf4j`.
I also had a logback-config.xml in the resources
```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration scan="true">
    <include resource="org/springframework/boot/logging/logback/base.xml"/>

    <logger name="org.myapp" level="DEBUG"/>

    <logger name="javax.activation" level="WARN"/>
    <logger name="javax.mail" level="WARN"/>
    <logger name="javax.xml.bind" level="WARN"/>
    <logger name="ch.qos.logback" level="WARN"/>
    <logger name="com.codahale.metrics" level="WARN"/>
    <logger name="com.netflix" level="WARN"/>
    <logger name="com.netflix.discovery" level="INFO"/>
    <logger name="com.ryantenney" level="WARN"/>
    <logger name="com.sun" level="WARN"/>
    <logger name="com.zaxxer" level="WARN"/>
    <logger name="io.undertow" level="WARN"/>
    <logger name="org.apache" level="WARN"/>
    <logger name="org.apache.catalina.startup.DigesterFactory" level="OFF"/>
    <logger name="org.bson" level="WARN"/>
    <logger name="org.hibernate.validator" level="WARN"/>
    <logger name="org.springframework" level="WARN"/>
    <logger name="org.springframework.web" level="WARN"/>
    <logger name="org.springframework.security" level="WARN"/>
    <logger name="org.springframework.cache" level="WARN"/>
    <logger name="org.thymeleaf" level="WARN"/>
    <logger name="org.xnio" level="WARN"/>
    <logger name="kafka" level="WARN"/>
    <logger name="org.I0Itec" level="WARN"/>
    <logger name="springfox" level="WARN"/>
    <logger name="sun.rmi" level="WARN"/>
    <logger name="sun.rmi.transport" level="WARN"/>
    <logger name="org.springframework.ws.client.MessageTracing.sent" level="TRACE"/>
    <logger name="org.springframework.ws.client.MessageTracing.received" level="TRACE"/>
    <logger name="org.springframework.ws.server.MessageTracing" level="TRACE"/>
 
    
    <contextListener class="ch.qos.logback.classic.jul.LevelChangePropagator">
        <resetJUL>true</resetJUL>
    </contextListener>

    <root level="#logback.loglevel#">
        <appender-ref ref="CONSOLE"/>
    </root>
	<property name="LOG_PATH" value="json-log.json" />
    <appender name="jsonAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_PATH}</File>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        	<fieldNames>
                <caller>caller</caller>
                <callerClass>class</callerClass>
                <callerMethod>method</callerMethod>
                <callerFile>file</callerFile>
                <callerLine>line</callerLine>
            </fieldNames>
            <includeCallerInfo>true</includeCallerInfo>
            <enableContextMap>true</enableContextMap>
            <shortenedLoggerNameLength>20</shortenedLoggerNameLength>
            <throwableConverter class="net.logstash.logback.stacktrace.ShortenedThrowableConverter">
                <maxDepthPerThrowable>20</maxDepthPerThrowable>
                <maxLength>1000</maxLength>
                <shortenedClassNameLength>30</shortenedClassNameLength>
                <rootCauseFirst>true</rootCauseFirst>
            </throwableConverter>
            <provider class="net.logstash.logback.composite.loggingevent.LoggingEventPatternJsonProvider">
                <pattern>{"application_name":"userInteractionLogApp","relativeTime":"#asLong{%relative}"}</pattern>
            </provider>
            <provider class="net.logstash.logback.composite.loggingevent.LoggingEventNestedJsonProvider">
                <fieldName>nested</fieldName>
                <providers>
		            <provider class="net.logstash.logback.composite.loggingevent.RawMessageJsonProvider">
		                <fieldName>customRawMessage</fieldName>
		            </provider>
                </providers>
            </provider>
            <provider class="net.logstash.logback.composite.loggingevent.ArgumentsJsonProvider">
                <includeNonStructuredArguments>true</includeNonStructuredArguments>
                <nonStructuredArgumentsFieldPrefix>prefix</nonStructuredArgumentsFieldPrefix>
            </provider>
            <provider class="net.logstash.logback.composite.loggingevent.ContextNameJsonProvider"/>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
            <maxIndex>1</maxIndex>
            <fileNamePattern>${LOG_PATH}.%i</fileNamePattern>
        </rollingPolicy>
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>1MB</MaxFileSize>
        </triggeringPolicy>
    </appender>
    <logger name="jsonLogger" additivity="false" level="DEBUG">
        <appender-ref ref="jsonAppender"/>
    </logger>
    <root level="INFO">
        <appender-ref ref="jsonAppender"/>
    </root>
</configuration>
```

I am looking to setup Json encoder and looking at the [Logstash JSON Encoder](https://github.com/logstash/logstash-logback-encoder) to encode the log messages using JSON so that they can eventually be consumed by Splunk.

To encode the messages in JSON as suggested by the doc, I included the following dependency in my `pom.xml`
```xml
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
</dependency>
```

To log using JSON format, you must configure logback to use either as per the JSON encoder doc:

* an appender provided by the logstash-logback-encoder library, OR
* an appender provided by logback (or another library) with an encoder or layout provided by the logstash-logback-encoder library


Code references:
* http://stackoverflow.com/questions/6020545/java-util-logging-logger-to-logback-using-slf4j
* http://stackoverflow.com/questions/1975939/read-environment-variables-from-logback-configuration-file

Debugging logs of Logback can be enabled by setting the property `logback.debug` to `true`. This can be done by setting `-Dlogback.debug=true` from command line.

I got this working by creating a project from Spring Initialiazr
