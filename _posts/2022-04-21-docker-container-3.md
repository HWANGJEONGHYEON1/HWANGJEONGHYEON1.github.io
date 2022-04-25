---
layout: post
title:  "도커 컨테이너 빌드업 - 3"
date:   2022-04-21 18:20:21 +0900
categories: docker
---

> # 도커 컨테이너 빌드업 - 3

## 운영체제레벨 가상화
- 호스트 운영체제를 공유하고 애플리케이션에 필요한 환경을 패키징 하는 것
    - 하드웨어 레벨 가상화: 하이퍼바이저 등을 이용한 가상머신의 방식
    - 운영체제 레벨 가상화: 컨테이너 기반의 애플리케이션 서비스 방식


## 도커를 이용한 서비스 개발 과정
1. 어플리케이션 개발
2. 베이스 이미지를 이용한 Dockerfile 작성 => 다양한 구동명령어와 어플리케이션 코드, 라이브러리, 여러 도구를 이 파일에 포함시킨다.
3. Dockerfile build를 통한 새로운 이미지 생성 => docker build, 로그를 화면에서 확인하여 오류를 확인할 수 있다.
4. 생성된 이미지를 이용한 컨테이너 실행 
    4-1. docker image를 통해 생성된 이미지를 확인하고 docker run을 한다.
    4-2. 도커 컴포즈를 이요한 다중 컨테이너 실행 => docker-compose.yml을 통해 다중 컨테이너 실행순서, 네트워크, 의존성 등을 통합관리할 수 있고 MSA 서비스개발도 가능하다.
5. 컨테이너 어플리케이션 서비스 테스트 
    5-1. 연결하는 IP와 포트 번호를 이용하여 웹 브라우저를 이용한 페이지 연결을 확인할 수 있어야한다.
    5-2. 마이크로서비스 테스트
6. 로컬 및 원격 저장소에 이미지 저장 => 로컬 및 원격에 있는 이미지 저장소에 생성한 이미지를 저장 하여 팀과의 공유를 한다.
7. 깃허브등을 이용한 Dockerfile 관리 => 도커 허브사이트와 연동하게되면 자동화된 빌드 기능을 이용한 이미지 생성도 가능하다.
8. 동일 환경에서의 지속적 어플리케이션 개발 수행 => 1~7 과정을 업무용 애플리케이션 이미지를 지속적으로 개발, 운영및 관리할 수 있다.


## 도커 이미지 명령어
- 도커 이미지 내려받기
    - docker pull: 도커 허브 레지스트리에서 로컬로 도커이미지 내려받기
    - docker push: 로컬에 있는 도커 이미지를 도커 허브 레지스트리에 업로드
    - docker login/logout: 업로드를 하기 전 도커 허브계정으로 로그인하기
- 도커 이미지 세부 정보조회
    - docker image inspect [OPTIONS] IMAGE [IMAGE..]
        - 명령의 출력결과는 JSON 언어 형태로 출력되는 정보가 많기 때문에 포맷 옵셜을 이용하여 원하는 정보만 출력할 수 있다.
        - 주요 정보
            - Image ID: "id"
            - 생성일: "Created"
            - 도커 버전: "DockerVersion"
            - CPU 아키텍처: "Archiecture"
            - 이미지 다이제스트 정보: "RootFS"
    - docker image inspect --format="{{ .Config}}" httpd
        - json으로 Config에 해당하는 정보가 나온다.
- 도커 개인 저장소 push/pull
    - docker login
    - docker push #{아이디}/httpd:3.0
    - docker pull #{아이디}/httpd:3.0

- 도커 삭제
    - docker image rm ${이미지이름}
    - docker image rm ubuntu:14.04 // 최신 버전을 제외한 것은 반드시 태그명 명시
    - docker image rm -f ${이미지id} // 관련 이미지 삭제 몇글자만 입력시 다 삭제
    - docker rmi ${docker images -q} // 이미지 전체삭제
    - docker rmi ${docker images | grep ubuntu} // 특정 이미지 이름이 포함된 것만 삭제
    - docker rmi ${docker images | grep -v ubuntu} // 특정 이미지 이름이 포함된 것만 제외하고 삭제
    - docker image prune -a // 사용중이 아닌 모든 이미지 제거


## 컨테이너는 프로세스
- 컨테이너는 도커 이미지를 기반으로 만들어지는 snapshot.
- 이 스냅샷은 읽기 전용의 이미지를 복제한 것, 그 위에 읽고 쓰기가 가능한 컨테이너 레이어를 결합하면 컨테이가 된다.
- 컨테이너 실행
    - docker create -it --name container-test1 ubuntu:latest // run과 달리 container 내부접근을 하지 않고 생성만 수행
    - docker ps -a
    
    ```
    CONTAINER ID   IMAGE                       COMMAND                  CREATED          STATUS                        PORTS                                                                                                    NAMES
    e7b0f3888e91   ubuntu:latest               "bash"                   4 seconds ago    Created
    ```
    - docker start container-test1 // status를 보면 created로 변경
    - docker attach container-test // docker attach는 실행중인 어플리케이션 컨테이너에 대한 단순 조회 작업 수행 시 유용
    - docker run 의 특징은 호스트 서버에 이미지가 없어도 로컬에 존재하지않는 이미지를 도커 허브에 자동으로 다운로드하며, 마지막에 컨테너에 실행할 명령을 입력하면 컨테이너 동작과 함께 처리된다.

## SQL 테스트 
- docker pull mysql:5.7
- docker images | grep mysql
- docker run -it mysql:5.7 /bin/bash
- /etc/init.d/mysql start
- db 생성 후
- cd /var/lib/mysql
- ls
- dockerdb 생성

## 컨테이너 모니터링 도구 cAdvisor 컨테이너 실행
- 서비스 운영을 하면서 필요한 시스템을 모니터링 하면서 특이사항 있을 때 대응하기 위해 모니터링수행
```
docker run \
 --volume=/:/rootfs:ro \
 --volume=/var/run:/var/run:rw \
 --volume=/sys:/sys:ro \
 --volume=/var/lib/docker/:/var/lib/docker:ro \
 --publish=9559:8081 \
 --detach=true \
 --name=cadvisor \
 google/cadvisor:latest
```