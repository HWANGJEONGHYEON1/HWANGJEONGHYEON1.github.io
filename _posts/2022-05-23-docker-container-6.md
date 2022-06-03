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

## 도커 컨테이너의 자원 사용에 대한 런타임 제약
- 서버 자원 모니터링
    - 서버 시스템 운영: 자원의 사용량(usage)
    - 활용도
    - CPU
    - 메모리
    - 디스크 I/O
    - 네트워크 트래픽
- 위의 목록들에 대해 문제점이 발생할 것 같은 곳을 찾아 예방하는것이 목적
- 도구
    - top
        - 리눅스 전체의 자원 소비량 및 개별 액티브 프로세스의 자원 사용량
    - htop
        - top 보다 향상된 자원 사용량 제공
    - sar
        - 다양한 옵션을 통해 시스템 전반의 사용량에 대한 세부적인 모니터링 제공, 주로 셀 스크립트에 포함하여 활용
    - vmstat, free
        - 메모리 성능 측정이 가능
    - dstat
        - 서버 전반의 자원 사용량에 대한 모니터링 제공, 개별 옵션으로 제어가능
    - iptraf-ng
        - 서버로 유입되는 네트워크 인터페이스별 패킷양, 프로토콜등을 통해 네트워크 트래픽 모니터링
- CPU 런타임 제약

```
# --cpu-shares

❯ docker run -d --name cpu_512 \
--cpu-shares 512\
leecloudo/stress:1.0 stress --cpu 4
invalid argument "512leecloudo/stress:1.0" for "-c, --cpu-shares" flag: strconv.ParseInt: parsing "512leecloudo/stress:1.0": invalid syntax
See 'docker run --help'.

~
❯ docker run -d --name cpu_512 \
--cpu-shares 512 \
leecloudo/stress:1.0 stress --cpu 4
05b70693ff1f3acded3559e31a33ed59ca8154dcaf34e98f978988d55c08e84d

```


## 도커 네트워크 
- 정의
    - 도커 설치 시 기본적으로 제공되는 docker0는 소프트웨어적으로 구현된 가상 이더넷 브리지 네트워크이고, 이것을 통해 격리된 컨테이너들의 상호 간 통신을 제공한다.
    - 별도의 브리지 네트워크를 생성하여 연결 값으로 설정하지 않는다면, docker0에 연결되어 172.17.0.0/16 CIDR 범위로 IP주소가 할당된다.
- 도커 컨테이너 및 서비스는 도커 네트워크를 통해 격리된 컨테이너 간의 네트워크 연결뿐만 아니라 도커 외의 다른 애플리케이션 워크로드와도 연결이 가능
- 도커 네트워크의 하위 시스템 연결을 위해 도커 네트워크 드라이버를 사용하여 상호 간 통신이 가능해진다.

```

❯ docker run -it -d --name cantainer1 ubuntu:14.04
0d403afdb9f2c6273752cb8a4e730a36470cbfe2266b946b9b74fa2a6a4b3e4a

~
❯ docker run -it -d --name cantainer2 ubuntu:14.04
77885e4dad69a2b9cc3cb22371368eb0666ecad6312b86fd202ea2a68673a300

❯ docker inspect -f "{{ .NetworkSettings.IPAddress }}" cantainer1
172.17.0.6

❯ docker inspect cantainer2 | grep Mac
            "MacAddress": ~
                    "MacAddress": ~

~
❯ docker exec cantainer1 route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         172.17.0.1      0.0.0.0         UG    0      0        0 eth0
172.17.0.0

```
