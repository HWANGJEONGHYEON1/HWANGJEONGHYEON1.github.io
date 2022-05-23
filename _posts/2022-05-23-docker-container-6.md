---
layout: post
title:  "도커 컨테이너 빌드업 - 6"
date:   2022-05-23 12:20:21 +0900
categories: docker
---


## 웹 서비스 로그 정보 보호 및 분석을 위한 바인드 마운트 설정
- 컨테이너에서 실행되는 웹 어플리케이션 Nginx의 접근 기록 정보인 access.log를 통해 장애 시 장애 상황 정보를 파악하거나, 실시간 접근 로그를 분석할 수 있다.
- 호스트 운영체제에 바인드 마운트를 통해 수집된 access.log에 대해 리눅스 쉘 스크립트에서 주로 사용되는 awk를 이용하여 로그 분석을 할 수 있다.

```
 docker run -d -p 8011:80 \
> -v ~/docker-practice/nginx-log:/var/log/nginx \
> nginx:1.19
36de595106effee49eba11177921f168adcdc29bb352ee9deef31ae3525b19cd
 jhhwang 🔥   ~/docker-practice/nginx-log
 ll
total 0
-rw-r--r--  1 we  staff     0B  5 23 15:08 access.log
-rw-r--r--  1 we  staff     0B  5 23 15:08 error.log
 jhhwang 🔥   ~/docker-practice/nginx-log
 tail -f nginx-log/access.log
tail: nginx-log/access.log: No such file or directory
 ✘ jhhwang 🔥   ~/docker-practice/nginx-log
 cd ..
 jhhwang 🔥   ~/docker-practice
 tail -f nginx-log/access.log

```

- nginx, 아파치 웹 서비스의 access.log 패턴
    - >IP(1) - (2) - (3) [날짜/시간(4) +0900](5) "POST(6) /*(7) HTTP(8) 200(9) 사이즈(10)


## 컨테이너 간 데이터 공유를 위한 데이터 컨테이너 만들기
- 컨테이너 뷸륨으로 지정된 디렉토리로부터 볼륨 마운트를 할 수 있다.
- 여러 컨테이너를 공유해서 사용할 수 있도록 데이터 전용 컨테이너를 생성하여 (docker run으로 실행x) 공유 볼륨을 여러 컨테이너에서 연결할 것
- 이를 통해 데이터 컨테이너를 만들 수 있고, 컨테이너 내의 데이터베이스 백업, 복구 및 마이그레이션 등의 작업에 활용할 수 있다,

```
 docker run -d -p 8080:80 \
> -v `pwd`/web-volume:/usr/share/nginx/html \
> --name=dev-web nginx:1.19
1d7d5b6d4385478b918a97e3f8b72eaa6d45c716d39f3c5f24ad901b200691cd
 jhhwang 🔥   ~/docker-practice
 curl localhost:8080
<html>
<head><title>403 Forbidden</title></head>
<body>
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.19.10</center>
</body>
</html>
 jhhwang 🔥   ~/docker-practice
 ll
total 0
drwxr-xr-x  4 we  staff   128B  5 23 15:08 nginx-log
drwxr-xr-x  2 we  staff    64B  5 23 15:46 web-volume
 jhhwang 🔥   ~/docker-practice
 cd web-volume
 jhhwang 🔥   ~/docker-practice/web-volume
 ll
 jhhwang 🔥   ~/docker-practice/web-volume
 sudo vi index.html
Password:
 jhhwang 🔥   ~/docker-practice/web-volume
 curl localhost:8080
<h1> hello, this is Hwang's application </h1>

```