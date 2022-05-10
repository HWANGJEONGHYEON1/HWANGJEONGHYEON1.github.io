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

## 도커 볼륨 확인
- 도커는 유니언 파일 시스템을 사용
- 하나의 이미지로부터 여러개의 컨테이너를 만들 수 있는 방법을 제공
- 디비, 웹 등 업무에서 사용하는 애플리케이션에서 발생하는 데이터에 접근하고 이것을 공유하기 위해서 도커 볼륨기능을 사용할 수 있다.
- 제공하는 서비스와 로직은 분리되어 있어야한다.
- 도커 볼륨은 컨테이너에서 생성, 재사용할 수 있고 호스트 운영체제에서 직접 접근이 가능하다.
- 데이터를 유지하기 위한 메커니즘을 제공한다.
- 일반적으로 컨테이너 내부의 데이터는 컨테이너의 라이프사이클과 연관되어 컨테이너 종료시 삭제
- 이를 독립적으로 사용하기 때문에 컨테이너가 삭제되도 볼륨은 삭제되지 않는다.
- 볼륨 생성방법
  - docker volume create ${name}
- 도커 볼륨은 명령어를 통해 관리가능
- 여러 컨테이너 간에 안전하게 공유가능
- 볼륨 드라이버를 통해 원격 호스트 및 클라우드 환경에 볼륨 내용을 저장하고 암화할 수 있다.
- 새 볼륨으로 지정될 영역에 데이터를 미리 채우고 컨테이너에 연결하면 컨테이너 내에서 바로 데이터 사용이 가능하다.

```
# 볼륨생성
 docker volume create my-appvol-1
my-appvol-1

# 생성된 볼륨 조회
 docker volume ls
DRIVER    VOLUME NAME
local     my-appvol-1

# 볼륨검사, 볼륨이 올바르게 생성되고 마운트까지 되었는지 검사
 docker volume inspect my-appvol-1
[
    {
        "CreatedAt": "2022-05-10T05:56:28Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-appvol-1/_data",
        "Name": "my-appvol-1",
        "Options": {},
        "Scope": "local"
    }
]

# --mount 옵션을 이용한 볼륨 지정
 docker run -d --name vol-test1 \
> --mount source=my-appvol-1,target=/app \
> ubuntu:20.04

# -v 옵션을 이용한 볼륨지정 
 docker run -d --name vol-test2 \
> -v my-appvol-1:/var/log \
> ubuntu:20.04
81117666a12ca809342049afab3f745f4088a3112e485cbcf453d529c11f957e

# 사전에 docker volume create를 하지 않아도 호스트 볼륨이름을 쓰면 자동 생성
 docker run -d --name vol-test3 \
> -v my-appvol-2:/var/log \
> ubuntu:20.04
f01404f7a4daef49222cf6ab92147c949243be7f64ba9deb63c521f8ab180b7c

# 
 docker inspect vol-test1
...
"Mounts": [
            {
                "Type": "volume",
                "Name": "my-appvol-1",
                "Source": "/var/lib/docker/volumes/my-appvol-1/_data",
                "Destination": "/app",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
...

 docker inspect --format="{{ .Mounts }}" vol-test1
[{volume my-appvol-1 /var/lib/docker/volumes/my-appvol-1/_data /app local z true }]

# 연결된 컨테이너가 있으면 에러
 docker volume rm my-appvol-1
Error response from daemon: remove my-appvol-1: volume is in use - [4858abd937271181cc47b7810dd92d3824951a96ad2a0fc17bc0a8b6eaf4698f, 81117666a12ca809342049afab3f745f4088a3112e485cbcf453d529c11f957e]

# docker stop 
 docker stop vol-test1 vol-test2
vol-test1
vol-test2
 docker rm vol-test1 vol-test2
vol-test1
vol-test2
 docker volume rm my-appvol-1
my-appvol-1

```

## bind mount
- 도커 볼륨기법에 비해 사용이 제한적
- 호스트 파일 시스템 절대경로: 컨테이너 내부 경로를 직접 마운트하여 사용한다.
- 사용자가 파일 또는 디렉터리를 생성하면 해당 호스트 파일 시스템의 소유자 권한으로 연결이되고 존재하지 않는 경우 자동 생성된다. 이때 자동생성된 디렉토리는 루트 사용자 소유가 된다.
- 컨테이너 실행 시 지정하여 사용되고, 컨테이너 제거 시 바인드 마운트는 해제되지만 호스트 디렉토리는 유지된다.

```
# --mount 옵션으로 사전에 생성한 경로와 마운트지정
 docker run -d -it --name bind-test1 \
--mount type=bind,source=/Users/we/jhhwang/target,target=/var/log \
centos:8
Unable to find image 'centos:8' locally
8: Pulling from library/centos
a1d0c7532777: Pull complete
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:8
d3c1d61f3f72cfb95e2664bab3e0ec8bfda66cb45ce8b1ea5f8eb8f6015a18e4

# -v 옵션으로 사전에 생성한 디렉터리와 바운드
 docker run -d -it --name bind-test2 \
-v /Users/we/jhhwang/target:/var/log \
centos:8
b31afd5ede881d67a1aa152a0c1d24d0285a2b1883fda794b169b432ae366b66

# 사전에 생성하지 않은 디렉터리와 바운드 마운트 지정
 docker run -d -it --name bind-test3 \
-v /Users/we/jhhwang/target2:/var/log \
centos:8
7a753601b3dfe33d3a1185998f13f21bdbe5a83ea6d45db75cdd85a0118ad12e

# 사전에 생성하지 않은 디렉터리에 읽기 전용 및 읽고 쓰기 바인드 마운트 지정
 docker run -d -it --name bind-test4 \
-v /Users/we/jhhwang/target_ro:/app1:ro \
-v /Users/we/jhhwang/target_ro:/app2:rw \
centos:8
435e6b73cb0dcdb24f77837ce8d6ca4a090963f3bfa5c12500e340084dba3202

# 바운드 마운트된 컨테이너 조회
 docker inspect --format="{{ .HostConfig.Binds}}" bind-test2
[/Users/we/jhhwang/target:/var/log]


```

## tmpfs mount
- 기존 방법은 컨테이너가 중지된 후에도 데이터를 유지할 수 있지만 tmpfs 마운트 방법은 임시적이며 호스트 메모리에서만 지속되므로 컨테이너가 중지되면 tmpfs 마운트가 제거되고 내부에 기록된 파일은 유지되지 않는다.
- 호스트 또는 컨테이너 쓰기 기능 계층에서 지속하지 않지만 중요한 파일을 임시로 사용하는 방법에 유용
- 컨테이너 실행 시 지정하여 사용하고, 컨테이너 제거 시 자동 해제

