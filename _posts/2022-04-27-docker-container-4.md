---
layout: post
title:  "도커 컨테이너 빌드업 - 4"
date:   2022-04-27 18:20:21 +0900
categories: docker
---

> # 도커 컨테이너 빌드업 - 4

## 파이썬 프로그래밍 환경을 제공 2
- 컨테이너 내부에 소스코드, 구성 정보등을 변경하는 경우 docker cp가 유용
```
# nginx 컨테이너 생성
 docker run -d -p 8010:80 --name=webserver10 nginx:latest
# nginx 컨테이너의 nginx.conf를 호스트로 복사
 docker cp webserver10:/etc/nginx/nginx.conf ./nginx.conf
# 구성정보 변경 후 재 설정
 docker cp nginx.conf webserver10:/etc/nginx/nginx.conf
# 도커 재시작
 docker restart webserver10 
```

## node.js 테스트 환경을 위한 컨테이너 실행

```node
var http = require('http');
var content = function(req, resp) {
  resp.end("GoodMoring vancouvor" + "\n");
  resp.writeHead(200);
}

var web = http.createServer(content);
web.listen(8002);
```

```
# 노드 컨테이너 실행
 docker pull node

 docker run -d -it -p 8002:8002 --name=nodejs_test node

 docker cp nodejs_test.js nodejs_test:/nodejs_test.js
 docker exec -it nodejs_test node /nodejs_test.js

# 도커 컨테이너 이름 변경
 docker rename nodejs_test nodeapp
```

## docker commit 
- 일반적으로 도커 이미지를 통해 컨테이너를 생성하지만, docker commit을 통해 컨테이너를 이미지로 생성할 수도 있다.

```

docker run -it --name webserver7 -d -p 8008:80 nginx:latest
docker cp index.html webserver8:/usr/share/nginx/html/index.html
docker diff webserver7

C /var
C /var/cache
C /var/cache/nginx
A /var/cache/nginx/client_temp
A /var/cache/nginx/fastcgi_temp
A /var/cache/nginx/proxy_temp
A /var/cache/nginx/scgi_temp
A /var/cache/nginx/uwsgi_temp
C /run
A /run/nginx.pid
C /etc
C /etc/nginx
C /etc/nginx/conf.d
C /etc/nginx/conf.d/default.conf


docker run -it --name webserver7 -d -p 8008:80 nginx:latest

# docker commit 으로 이미지를 생성 -a(author) 
 docker commit -a "jpub" webserver7 webfront:1.0

docker images | grep webfront
webfront                   1.0       1545bd6feef2   9 seconds ago   141MB

# 도커 저장소에 올린다.
docker login
docker tag webfront:1.0 ${도커아이디}/webfront:1.0
docker push ${도커아이디}/webfront:1.0

# 실행 
docker run -it --name webserver9 -d -p 8009:80 ${도커아이디}/webfront:1.0
```