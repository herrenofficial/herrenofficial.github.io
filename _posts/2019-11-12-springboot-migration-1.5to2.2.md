---
layout: post
title: springboot 1.5 -> 2.2로 버전 업그레이드
date : 2019/11/12
avatar: "assets/img/profile/seha.jpg"
categories : [Springboot]
comments: true
tags: 
- Springboot
---
## 목차
- TOC
{:toc}

## 들어가며
안녕하세요, [공비서](https://www.gongbiz.kr/) 개발자 세하 입니다 😊   
저희 공비서 서비스는 springboot 1.5.9 버전 프레임워크를 이용해 만들어져 있는데요, 지난 [10월 16일에 릴리즈](https://spring.io/blog/2019/10/16/spring-boot-2-2-0)된 2.2으로의 버전 마이그레이션을 한 과정을 포스팅하려고 합니다.  
따라서 포스팅은 버전 **1.5.9 부터 2.2.1**(19.11.12 기준 최신 릴리즈)까지의 버전 마이그레이션하는 과정에 대한 내용으로 구성되어있습니다.    
[[공식 가이드]](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide)대로 했지만 업그레이드 과정에서 많은 문제를 겪었기 때문에 추후에 삽질을 줄이고자 이 포스팅을 작성하게 되었습니다.  

그리고 한번에 버전을 1.5 -> 2.2로 올려버리면 오류를 잡았음에도 불구하고 빌드가 되지 않았습니다.  
혹 이유를 아시는 분은 코멘트 남겨주세요🙇 
따라서 `1.5 -> 2.0 -> 2.2` 순서로 버전을 업그레이드 하도록 하겠습니다.

![logo]({{ "/assets/img/post/20191112/springBoot2Art.png"}})

## migration 필수 조건
* JDK 8 이상
* tomcat 8.5 이상(외부 톰캣 사용하는 경우)
* gradle 4.10 이상

메이져 버전 마이그레이션 시 지원이 중단된 몇가지 서드파티 라이브러리들의 버전을 업그레이드 혹은 대체를 해야할 수도 있습니다.  
[springboot 2.2 릴리즈 노트](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.2-Release-Notes) 에서 변경점을 더 확인할 수 있습니다.

## migration 순서
### 1. gradle 버전이 5.0 이 아닐경우 5.0-all 로 변경
스프링 2.2로의 마이그레이션을 위해선 **gradle 4.10이상**이어야 합니다.  
프레임워크 버전업과 더불어 저는 gradle 버전을 4.10보단 5로 맞췄습니다.  
`gradle > wrapper > gradle-wrapper.properties` 에서 아래와 같이 gradle 버전을 수정합니다.  
```
distributionUrl=https\://services.gradle.org/distributions/gradle-5.0-all.zip
```
gradle의 버전을 업그레이드하면 지원이 중단된 플러그인등이 발생할 수 있습니다.  
deprecated 된 플러그인들을 적절히 대체 및 업그레이드 합니다.  
(자바 코드상의 변경이 필요할 수도 있습니다)

### 2. build.gradle에 dependency-management를 추가
마이그레이션을 위한 의존성 관리 플러그인을 추가합니다.
```gradle
apply plugin: 'io.spring.dependency-management'
runtime("org.springframework.boot:spring-boot-properties-migrator")
```
### 3. springboot 버전 바꾸기
build.gradle의 springboot 버전을 변경합니다.
```gradle
springBootVersion = '2.2.0.RELEASE'
```

### 4. 라이브러리 교체와 클래스 변경
컴파일 오류가 발생했다면 해당되는 오류를 고칩니다.   
보통은 사용하던 서드파티 라이브러리가 deprecated 되어 대체 후 오류가 나는 경우가 많습니다. 
(ex:log4j 지원중단으로 log4j2로 고쳤습니다. 따라서 프로젝트에 사용중이던 logger -> log 로 변경되었습니다)  
버전 업그레이드로 인한 특정 라이브러리 설정의 변화로 발생하는 오류를 해결하는 방법은 다음의 링크에서 확인하실 수 있습니다.
* [common-application-properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)
* [공식문서 링크](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#servlet-specific-server-properties) 

>2.1 버전 부턴 mysql 드라이버 명이 변경되었습니다.
기존 : com.mysql.jdbc.Driver
바뀐것 : com.mysql.cj.jdbc.Driver

또한 몇가지 클래스및 패키지가 변경되었습니다.  
예를들어:  
>WebMvcConfigurerAdapter deprecated로 인한 변경
기존 : extends WebMvcConfigurerAdapter
변경 : implements WebMvcConfigurer

>SpringBootServletInitializer 패키지 이름의 변경
기존 : import org.springframework.boot.web.support.SpringBootServletInitializer;
변경 : import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

위와 같은 클래스 및 패키지가 변경되었습니다.  
이러한 변경은 IDE에서 잡아주거나 컴파일시 warn 알림으로 deprecated 되었다는 알림을 주기 때문에  
그에 맞춰 수정하거나, 2.0 릴리즈 노트에서 확인할 수 있습니다.
* [Spring Boot 2.0 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Release-Notes)

### 5. application.properties 수정
IDE에서 해당 키워드에 warning 표시를 나타내며 제안에 따라 수정할 수 있습니다.  
또는 [지원하는 의존성 버전](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#appendix) 문서를 보며 `application.properties`에서 기존의 키워드를 변경된 키워드로 수정합니다.

### 6. bean overriding
해당 변경은 2.1의 릴리즈 노트에서 다음과같이 소개하고 있습니다.[[링크]](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.1-Release-Notes#bean-overriding)
>Bean overriding has been disabled by default to prevent a bean being accidentally overridden. If you are relying on overriding, you will need to set spring.main.allow-bean-definition-overriding to true.

2.1 버전부터 빈의 우연한 재정의를 방지하기위해 default가 false 라고 합니다. 따라서, __been overriding을 사용하려면 true로 명시__해줘야 합니다.    
명시가 되어있지 않은 경우 다음과 같은 오류가 발생합니다.
```java
org.springframework.beans.factory.support.Bean Definition Override Exception
```
따라서 application.properties에 bean overriding을 허용하는 부분을 추가합니다.
```java
spring.main.allow-bean-definition-overriding=true
```

### 7. ‘KST’ is unrecognized or represents more than one time zone
~~~
The server time zone value ‘KST’ is unrecognized or represents more than one time zone :   
mysql-connector-java
~~~
위의 오류는 아래와 같이 db name 뒤에 `?characterEncoding=UTF-8&serverTimezone=Asia/Seoul` 를 추가합니다
~~~
jdbc:mysql://ip:port/dbname?characterEncoding=UTF-8&serverTimezone=Asia/Seoul
~~~

### 8. 버전 업그레이드 2.0 -> 2.2
빌드 및 run이 성공적으로 되면 `build.gradle`에서  `2.2.1.RELEASE` 로 springboot 버전을 변경합니다.
```gradle
springBootVersion = '2.2.1.RELEASE'
```
그 후 4~5번의 작업을 반복하며 오류를 잡습니다.  
2.0 -> 2.2 로 버전이 올라가며 변경된 점은 springboot 릴리즈 노트에서 확인하며 수정합니다.
* [Spring Boot 2.1 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.1-Release-Notes)
* [Spring Boot 2.2 Release Notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.2-Release-Notes)

### 9. properties-migrator 제거
빌드 및 run이 성공적으로 되면 build.gradle 에서 추가해뒀던 아래 코드를 제거 합니다.
```gradle
runtime("org.springframework.boot:spring-boot-properties-migrator")
```

### 10. mybatis.configuration.result-maps[0] 오류
혹 2.2.0 버전으로의 업그레이드를 한다면 다음과 같은 오류를 만날수도 있습니다.
```java
Caused by: org.springframework.boot.context.properties.bind.BindException: Failed to bind properties under 'mybatis.configuration.result-maps[0]' to org.apache.ibatis.mapping.ResultMap
```
다음과 같은 오류는 `2.2.0.RELEASE (2.2.0.M5 +)` 버전에서 발생하고 있는 이슈며 `2.2.0.M4`와 `2.2.1.RELEASE` 에선 해결되었습니다.  
해당 이슈는 스프링부트 깃허브 이슈 [[Binding fails in presence of a synthetic constructor #18670]](https://github.com/spring-projects/spring-boot/issues/18670) 에서 확인할 수 있습니다.

추가) 만약 swagger를 쓰는데 마이그레이션 시 오류 날 경우 2.9.1로 올려주셔야 됩니다.  
Lombok, Gradle, JPA 문제등은 [[honeymon 님의 스프링 부트 2.x 준비하는 개발자를 위한 안내서]](http://honeymon.io/tech/2019/06/17/spring-boot-2-start.html) 포스팅에서 자세히 알아보실 수 있습니다.

## 마치며
공비서는 이렇게 1.5 -> 2.2로의 마이그레이션을 마쳤습니다.  
아직 2.2 버전은 릴리즈한지 git s오래되지않아 마이그레이션에 대한 자료도 적으며 이슈도 많았습니다.  
하지만 2.2 버전으로 올라가며 [초당 처리량 20~30% 증가](https://www.slideshare.net/Pivotal/spring-boot-22), jUnit5 등등의 장점때문에 마이그레이션을 결심하게 되었습니다.  

읽어주셔서 감사합니다 :)

혹 내용이 잘못되었거나 보충해야 될 부분이 있다면 코멘트 남겨주세요 🙏

--- 

참고자료
* [[Spring Boot 2.2 Release Notes]](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.2-Release-Notes)
* [spring-boot-migration-java](https://altkomsoftware.pl/en/blog/spring-boot-migration-java/)
* [Spring-Boot-2.0-Migration-Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide#servlet-specific-server-properties)
* [[honeymon 님의 스프링 부트 2.x 준비하는 개발자를 위한 안내서]](http://honeymon.io/tech/2019/06/17/spring-boot-2-start.html)

>추가로 저희는 **[파이썬 백엔드 개발자](https://www.rocketpunch.com/jobs/63087/%ED%83%84%ED%83%84%ED%95%9C-%EC%8A%A4%ED%83%80%ED%8A%B8%EC%97%85%EC%97%90%EC%84%9C-Python-%EA%B0%9C%EB%B0%9C%EC%9E%90-%EA%B5%AC%EC%9D%B8)** 를 채용중입니다!
채용에 관심이 있는 분은 **[채용페이지](https://www.rocketpunch.com/companies/herren/jobs)**를  
확인해주세요 😊