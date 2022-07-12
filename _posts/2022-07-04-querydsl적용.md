---
layout: post
title:  "QueryDSL 적용"
date:   2022-07-12 16:20:21 +0900
categories: java, spring, querydsl
---

> QueryDSL 적용 ( 스프링 부트 2.6 이상, Querydsl 5.0 지원 방법 추가)

## 기존 방법

```java
plugins {
    id 'com.ewerk.gradle.plugins.querydsl' version '1.0.10'
}
 
 
dependencies{
    implementation 'com.querydsl:querydsl-jpa'
}
 
 
def querydslDir = "$buildDir/generated/querydsl" as Object
 
querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}
 
 
// gradle 6.x 이상 버전에서 필수로 추가해야 하는 옵션
configurations {
    querydsl.extendsFrom compileClasspath
}
 
 
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
```


## 새로운 방법

```java
buildscript {
    ext {
        set('querydsl.version', '5.0.0')
    }
}
 
 
dependencies {
    implementation 'com.querydsl:querydsl-jpa'
    implementation 'com.querydsl:querydsl-core'
    annotationProcessor "com.querydsl:querydsl-apt:${dependencyManagement.importedProperties['querydsl.version']}:jpa" // querydsl JPAAnnotationProcessor 사용 지정
    annotationProcessor 'jakarta.persistence:jakarta.persistence-api' // java.lang.NoClassDefFoundError(javax.annotation.Entity) 발생 대응
    annotationProcessor 'jakarta.annotation:jakarta.annotation-api' // java.lang.NoClassDefFoundError (javax.annotation.Generated) 발생 대응
}
 
 
def querydslDir = "$buildDir/generated" as Object
task cleanQueryDsl {
    delete file(querydslDir)
}
```

## Q파일 생성
- gradle compileJava를 실행하면 Q클래스가 생성된다.

## Configuration

```java
@Configuration
public class QueryDslConfig
{
    @Bean
    public JPAQueryFactory jpaQueryFactory(EntityManager entityManager)
    {
        return new JPAQueryFactory(entityManager);
    }
}
```


## 참고
- http://honeymon.io/tech/2020/07/09/gradle-annotation-processor-with-querydsl.html