---
title: logback-config
date: 2018-05-25 11:47:40
tags: [logback]
cover: lblogo.jpg
categories: [tip]
subtitle: logback + log4jdbc 설정하기
---

# logback 설정하기
서버 상용화 단계에서 로그를 최적화하기 위해 공부한 내용을 정리한다.
한 번 설정하고 신경 끄면 잊어버릴 테니까 여기에 기록한다.

※ 참고 사이트
https://stackoverflow.com/questions/48657071/logback-logger-name-with-wildcard
https://logback.qos.ch/manual/

※ 소스
https://github.com/arcjjang/logback-log4jdbc-config

## 1) logging 시나리오
1.	Spring Boot Logback 기본 설정을 최대한 활용하고 싶다.
2.	Gateway 서버용 log.
3.	한동안 log를 통해 VOC를 처리해야 한다.
4.	Root log와 별도로 연동 서비스별로 Logging을 하고 싶다.
5.	DB 로깅은 log4jdbc를 사용한다.

## 2) log4jdbc 설정
application.properties
``` YAML
# DataSource
spring.datasource.driver-class-name=net.sf.log4jdbc.sql.jdbcapi.DriverSpy
spring.datasource.url=jdbc:log4jdbc:mariadb://?.?.?.?:3306/???
spring.datasource.username=???
spring.datasource.password=???
```

log4jdbc.log4j2.properties
``` YAML
log4jdbc.spylogdelegator.name=net.sf.log4jdbc.log.slf4j.Slf4jSpyLogDelegator
# log4jdbc.dump.sql.maxlinelength=0 를 삭제하면 1줄로 길게 나온다...
log4jdbc.dump.sql.maxlinelength=0
```

logback.xml
``` xml
<!-- ===================================== -->
<!-- log4jdbc 추천settings                     -->
<!-- ===================================== -->
<!--                       development production -->
<!-- jdbc.connection     :     WARN       WARN    -->
<!-- jdbc.audit          :     WARN       WARN    -->
<!-- jdbc.sqlonly        :     WARN       WARN    -->
<!-- jdbc.sqltiming      :     INFO       WARN    -->
<!-- jdbc.resultset      :     WARN       WARN    -->
<!-- jdbc.resultsettable :     INFO       WARN    -->
<logger name="jdbc" level="OFF"/>
<logger name="jdbc.sqlonly" level="OFF"/>
<logger name="jdbc.sqltiming" level="INFO"/>
<logger name="jdbc.audit" level="OFF"/>
<logger name="jdbc.resultset" level="OFF"/>
<logger name="jdbc.resultsettable" level="INFO"/>
<logger name="jdbc.connection" level="OFF"/>
```

## 3) Spring Boot Logback 기본 설정 활용
logback.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
        <file>logs/log.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/log.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>365</maxHistory>
            <totalSizeCap>40GB</totalSizeCap>
        </rollingPolicy>
    </appender>

<root level="INFO">
    <appender-ref ref="CONSOLE"/>
    <appender-ref ref="FILE"/>
</root>
```
-	Console log는 Spring Boot 기본값과 동일하다.
-	File log는 /logs 폴더에 10MB씩 날짜별로도 쌓인다.
-	ex) /logs/log.2018-05-24.0.log, log.log, log.0.log, log.1.log …

## 4) LogBack Filter 활용하기
### 4-1) 나만의 필터 만들기
KakaoFilter.java
``` java
package com.example.Filter;

import ch.qos.logback.classic.spi.ILoggingEvent;
import ch.qos.logback.core.filter.Filter;
import ch.qos.logback.core.spi.FilterReply;

public class KakaoFilter extends Filter<ILoggingEvent> {
    @Override
    public FilterReply decide(ILoggingEvent event) {
        if (event.getLoggerName().contains("Kakao")) {
            return FilterReply.ACCEPT;
        } else {
            return FilterReply.DENY;
        }
    }
}
```
logback.xml
``` xml
<appender name="KAKAO-FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">

    <filter class="com.example.Filter.KakaoFilter" />

    <encoder>
        <pattern>${FILE_LOG_PATTERN}</pattern>
    </encoder>
    <file>logs/kakao/log.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <fileNamePattern>logs/kakao/log.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
        <maxFileSize>10MB</maxFileSize>
        <maxHistory>365</maxHistory>
        <totalSizeCap>40GB</totalSizeCap>
    </rollingPolicy>
</appender>
```
-	별도의 의존성이 필요없다.

### 4-2) GEventEvaluator 필터 사용하기
logback.xml
``` xml
<!--groovy 의존성이 필요하다.-->
<appender name="NAVER-FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <filter class="ch.qos.logback.core.filter.EvaluatorFilter">
        <evaluator class="ch.qos.logback.classic.boolex.GEventEvaluator">
            <expression>
                <!--e.loggerName =~ "com.example\\..*\\.Naver*"-->
                e.loggerName =~ "com.example.*.Naver*"
            </expression>
        </evaluator>
        <OnMismatch>DENY</OnMismatch>
        <OnMatch>NEUTRAL</OnMatch>
    </filter>
    <encoder>
        <pattern>${FILE_LOG_PATTERN}</pattern>
    </encoder>
    <file>logs/naver/log.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <fileNamePattern>logs/naver/log.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
        <maxFileSize>10MB</maxFileSize>
        <maxHistory>365</maxHistory>
        <totalSizeCap>40GB</totalSizeCap>
    </rollingPolicy>
</appender>
```
gradle 의존성 추가
``` gradle
compile('org.codehaus.groovy:groovy-all:2.4.15')
```

### 4-3) JaninoEventEvaluator 필터 사용하기
logback.xml
``` xml
<!--janino 의존성이 필요하다...-->
<appender name="GOOGLE-FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <filter class="ch.qos.logback.core.filter.EvaluatorFilter">
        <evaluator> <!--필터를 명시 안 하면 기본 필터임-->
            <expression>return logger.contains("Google");</expression>
        </evaluator>
        <OnMismatch>DENY</OnMismatch>
        <OnMatch>NEUTRAL</OnMatch>
    </filter>
    <encoder>
        <pattern>${FILE_LOG_PATTERN}</pattern>
    </encoder>
    <file>logs/google/log.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <fileNamePattern>logs/google/log.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
        <maxFileSize>10MB</maxFileSize>
        <maxHistory>365</maxHistory>
        <totalSizeCap>40GB</totalSizeCap>
    </rollingPolicy>
</appender>
```
gradle 의존성 추가
``` gradle
compile('org.codehaus.janino:janino:3.0.8')
```

## 5) 전체 logback.xml 설정
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
        <file>logs/log.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/log.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>365</maxHistory>
            <totalSizeCap>40GB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <!--groovy 의존성이 필요하다.-->
    <appender name="NAVER-FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.core.filter.EvaluatorFilter">
            <evaluator class="ch.qos.logback.classic.boolex.GEventEvaluator">
                <expression>
                    <!--e.loggerName =~ "com.example\\..*\\.Naver*"-->
                    e.loggerName =~ "com.example.*.Naver*"
                </expression>
            </evaluator>
            <OnMismatch>DENY</OnMismatch>
            <OnMatch>NEUTRAL</OnMatch>
        </filter>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
        <file>logs/naver/log.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/naver/log.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>365</maxHistory>
            <totalSizeCap>40GB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <!--janino 의존성이 필요하다...-->
    <appender name="GOOGLE-FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.core.filter.EvaluatorFilter">
            <evaluator> <!--필터를 명시 안 하면 기본 필터임-->
                <expression>return logger.contains("Google");</expression>
            </evaluator>
            <OnMismatch>DENY</OnMismatch>
            <OnMatch>NEUTRAL</OnMatch>
        </filter>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
        <file>logs/google/log.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/google/log.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>365</maxHistory>
            <totalSizeCap>40GB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <appender name="KAKAO-FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">

        <filter class="com.example.Filter.KakaoFilter" />

        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
        <file>logs/kakao/log.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/kakao/log.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>365</maxHistory>
            <totalSizeCap>40GB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <!-- ===================================== -->
    <!-- log4jdbc 추천 settings                     -->
    <!-- ===================================== -->
    <!--                       development production -->
    <!-- jdbc.connection     :     WARN       WARN    -->
    <!-- jdbc.audit          :     WARN       WARN    -->
    <!-- jdbc.sqlonly        :     WARN       WARN    -->
    <!-- jdbc.sqltiming      :     INFO       WARN    -->
    <!-- jdbc.resultset      :     WARN       WARN    -->
    <!-- jdbc.resultsettable :     INFO       WARN    -->
    <logger name="jdbc" level="OFF"/>
    <logger name="jdbc.sqlonly" level="OFF"/>
    <logger name="jdbc.sqltiming" level="INFO"/>
    <logger name="jdbc.audit" level="OFF"/>
    <logger name="jdbc.resultset" level="OFF"/>
    <logger name="jdbc.resultsettable" level="INFO"/>
    <logger name="jdbc.connection" level="OFF"/>

    <root level="INFO">
        <appender-ref ref="NAVER-FILE"/>
        <appender-ref ref="GOOGLE-FILE"/>
        <appender-ref ref="KAKAO-FILE"/>
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>

</configuration>
```

## 6) 테스트 결과
{% asset_img result.png result %}