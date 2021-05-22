---
layout: post
title:  "Springboot jar에서 동작확인"
date:   2021-05-21 14:15
categories: spring, build
tags: [java, build]
---

### jar 빌드해서 동작 확인


~~~ 
~/IdeaProjects/lecture master*
❯ cd jpashop

~/IdeaProjects/lecture/jpashop master*
❯ `./gradlew clean build`

Welcome to Gradle 7.0!

Here are the highlights of this release:
 - File system watching enabled by default
 - Support for running with and building Java 16 projects
 - Native support for Apple Silicon processors
 - Dependency catalog feature preview

For more details see https://docs.gradle.org/7.0/release-notes.html

Starting a Gradle Daemon, 1 incompatible Daemon could not be reused, use --status for details

BUILD SUCCESSFUL in 13s
8 actionable tasks: 8 executed

~/IdeaProjects/lecture/jpashop master* 14s
❯ `cd build`

~/IdeaProjects/lecture/jpashop/build master*
❯ `ll`
total 8
staff    34B  5 21 14:24 bootJarMainClassName
staff    96B  5 21 14:24 classes
staff    96B  5 21 14:24 generated
staff   128B  5 21 14:24 libs
staff    96B  5 21 14:24 reports
staff    96B  5 21 14:24 resources
staff    96B  5 21 14:24 test-results
 staff   224B  5 21 14:24 tmp

~/IdeaProjects/lecture/jpashop/build master*
❯ `cd libs`

~/IdeaProjects/lecture/jpashop/build/libs master*
❯ `ll`
total 81440

-rw-r--r--  1  jpashop-0.0.1-SNAPSHOT.jar

~/IdeaProjects/lecture/jpashop/build/libs master*
❯ `java -jar jpashop-0.0.1-SNAPSHOT.jar`

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.0)
8  INFO 21705 --- [           main] o.s.b.a.ApplicationAvailabilityBean      : Application availability state ReadinessState changed to ACCEPTING_TRAFFIC
~~~