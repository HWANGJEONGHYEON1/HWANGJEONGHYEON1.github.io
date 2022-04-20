---
layout: post
title:  "도커 컨테이너 빌드업 - 2"
date:   2022-04-20 22:20:21 +0900
categories: docker
---

> # 도커 컨테이너 빌드업 - 2

## 도커 명령어
* docker pull busybox
* docker images
~~~
CONTAINER ID   IMAGE                 COMMAND                  CREATED         STATUS                       PORTS                                NAMES
4732412d8726   busybox               "sh"                     5 seconds ago   Exited (0) 4 seconds ago                                          gracious_cartwright
~~~

* docker ps -a : 실행된 모든 컨테이너의 정보를 제공.
* docker run busybox echo 'Hello World' =>  Hello World 출력
    * 출력된 값은 도커 컨테이너 내부에서 출력되 값
    * Nginx 웹 어플리케이션 서버라면, 환경 구성, 설치, 서비스 시작을 하지않고, 도커를 이용하여 Nginx 서비스를 배포할 수 있다.
* docker [system] info : 실행중이 정보를 출력할 수 있음
* docker info --format '{{json .}}' : json으로 출력
* docker system df
~~~
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          24        3         7.796GB   6.97GB (89%)
Containers      5         0         3.352MB   3.352MB (100%)
Local Volumes   4         2         1.039GB   508.3MB (48%)
Build Cache     45        0         574.1MB   574.1MB
~~~

* 터미널1) docker system events : 도커서버에서 발생하는 도커 관련 이벤트 정보를 표시
* 터미널2) docker run -itd -p 80:80 --name=webapp nginx 
* docker system events --filter #{param} : 이벤트 옵션 필터(많은 정보가 포함되기 때문에 필터를 걸어 볼 수 있다.)
    * 도커 데몬에 문제가 있다면 컨테이너에도 영향을 끼치므로, 이러한 로그들을 통해 원인정보를 파악하는것이 중요.
    * param
        * type=image
        * event=stop
        * contatiner=webapp
        * contatiner=webapp --filter 'event=stop'
