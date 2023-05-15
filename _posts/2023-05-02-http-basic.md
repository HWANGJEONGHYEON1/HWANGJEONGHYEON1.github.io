---
layout: post
title:  "Http Basic - 1"
date:   2023-05-02 23:20:21 +0900
categories: spring
---

> Http 기초지식 다시 공부

## 인터넷 프로토콜
- 지정한 IP주소에 데이터 전달
- 패킷이라는 통신단위로 전달 
- `출발질 IP, 목적지 IP, 데이터`를 가지고 전달
- 한계
    - 비연결성: 일단 보낸다, 대상이 없어도 전송
    - 비신뢰성: 중간에 사라질 수 있다. 패킷이 순서대로 안온다.
    - 프로그램 구분: 같은 IP를 사용하는 서버에서 통신

## TCP, UDP
- TCP/IP 패킷정보  
    - 출발지 PORT, 목적지 PORT, 전송, 순서 제어, 검증 정보
    - 연결 지향 : 3 way handshake
        1. syn (접속요청)
        2. syn + ack 
        3. ack (요청수락)
    - 데이터 전달 보증
        - 데이터 전송 시 서버에서 응답해줌
    - 순서 보장
    - 신뢰할 수 있는 프로토콜
    - 대부분 TCP 사용
- UDP
    - 데이터 전달보증X
    - 순서보장X
    - PORT 하나 추가만 됨 IP + PORT + 체크섬만 추가
    - 데이터 전달 및 순서가 보장되지는 않지만, 단순하고 빠름

## PORT, DNS
- PORT
    - 한번에 둘 이상을 연결해야한다면?
    - 같은 IP내에서 프로세스 구분
    - 0 ~65535 할당가능
- DNS
    - IP는 변경가능 
    - 도메인 명을 IP주소로 변환
    - 클라이언트 -> DNS 서버(구글.com 요청, 200.~.2 IP 응답), 클라이언트 IP로 요청-> 서버

## 웹 요청흐름
- DNS 조회하여 아이피 가져온다.
- HTTP 요청 메시지 생성
    - GET /search?query=1 HTTP/1.1
    - Host: www.google.com
- SOCKET 라이브러리를 통해 전달(TCP/IP 연결, 데이터 전달)
- TCP/IP 패킷 생성, HTTP 메시지
- HTTP 응답 메시지

## HTTP
- 역사
    - HTTP/0.9: GET만 지원, 헤더없음
    - HTTP/1.0: 메서드, 헤더 추가
    - HTTP/1.1: 가장 많이 사용
    - HTTP/2: 성능개선
    - HTTP/3: TCP 대신 UDP 사용, 성능개선
- 특징
    - 클라이언트 서버 구조
        - Request/Response 구조
    - 무상태 프로토콜(stateless), 비연결성
        - Stateful, Stateless 차이
            - 상태유지 - Stateful
            - 무상태
        - 서버가 클라이언트의 상태를 유지 X
        - 서버 확장성 높음(스케일 아웃)
        - 클라이언트가 추가 데이터 전송을 해야하는 단점 
    - HTTP 메시지
    - 확장가능

## HTTP METHOD
- HEAD: Get과 동일하지만 메시지 부분을 제외하고, 상태 줄과 헤더만 반환
- OPTIONS: 주로 CORS에서사용
- GET: 메시지 바디를 사용해서 데이터를 전달할 수 있지만, 지원하지 않는곳이 많다.
- POST
    - 새 리소스 생성
    - 요청 데이터 처리
    - 다른 메소드러 처리하기 애매한 경우
- PUT: 리소스를 대체, 리소스 없으면 생성 => 덮어버람, `클라이언트가 리소스를 식별`
- PATCH: 리소스 변경부분만 대체
- DELETE: 리소스 제거
- 속성
    - 안전
        - 호출해도 리소스가 변경하지 않는다.
    - 멱등
        - 몇 번을 호출해도 결과가 똑같아야한다.
        - GET, PUT, DELETE -> 요청을 여러번 해도 결과는 똑같다
        - POST : 두번 호출하면 결과가 바뀔 수 있다. 멱등이 아니다.
    - 캐시가능
        - GET, HEAD, POST, PATCH 캐시 가능
        - 실제로는 GET, HEAD정도만 캐시사용

## HTTP 상태코드
- 1xx: 요청이 수신되어 처리중

- 2XX: 요청 정상처리
    - 200 OK
    - 201 Created
        - Location 헤더 필드로 식별
    - 202 Accepted
        - 요청이 접수되었으나 처리가 완료되지 않음
    - 204 No Content
        - 서버가 요청을 성공했지만, 응답 페이로드 본문에 보낼 데이터가 없음

- 3XX: 요청을 완료하려면 추가 행동이 필요/리다이렉션
    - 영구 리다이렉션: 특정 리소스의 URI가 영구적으로 이동
    - 일시 리다이렉션: 일시적인 변경, PRG(Post/Redirect/Get)
        - PRG
            - POST 시 새로고침으로 인하여 중복 방지
            - 요청 후 GET으로 리다이렉트
            - 리다이렉트해도 다시 GET 중복으로 POST 요청을 안함
    - 특수 리다이렉션: 결과 대신 캐시사용
    - 300대 응답의 결과에 Location이 있으면 자동으로 그 위치로 이동(리다이렉트)
    - 301, 308 Moved Permanently(영구)
        - 리다이렉트시 요청 메서드가 Get으로 변하고, 본문이 제거 될 수 있음
        - 308은 처음 POST라면 그대로 유지한다. 메시지 유지
    - 302, 307, 303
        - 리소스의 URI가 일시적으로 변경
        - 검색 엔진 등에서 URL을 변경되면 안됨
        - 302 Found
            - 리다이렉트 시 요청 메서드가 Get으로 변하고, 본문이 제거될 수 있음 
        - 307 Temporary Redirect
            - 리다이렉트시 요청 메서드와 본문유지
        - 303 See Other
            - 리다이렉트 시 요청 메서드가 Get으로 변경
    - 304 Not Modified
        - 캐시를 목적으로 사용
        - 클라이언트에게 리소스가 수정되지 않았음을 알려줌, 따라서 클라이언트는 로컬PC에 저장된 캐시를 사용(캐시로 리다이렉트)
        - 304 응답은 응답에 메시지 바디를 포함하면 않된다.
        - GET, HEAD 요청 시 사용

- 4XX: 클라이언트 오류, 잘못된 문법등으로 서버가 요청을 수행할 수 없음
    - 400 Bad Request
        - 요청구문, 메시지 등 오류
        - 클라이언트는 요청 내용을 다시 검토해야함
    - 401 Unauthorized
        - 인증되지 않음
        - WWW-Authenticate 헤더와함께 인증방법 설명해야함
    - 403 Forbidden
        - 인증자격은 됬지만, 접근권한이 불충분
    
- 5XX: 서버 오류, 서버가 정상 요청을 처리하지 못함
    - 500 Internal Server Error
    - 503 Service Unavaliable
        - 서버가 일시적인 과부하 또는 예정된 작업으로 잠시 요청을 처리할 수 없음
        - Retry-After 헤더필드로 얼마뒤에 복구되는지 보낼 수 있음 -> 잘 안나옴


## HTTP 헤더
- HTTP 전송에 필요한 모든 부가정보
- 표현
    - Content-type: 표현 데이터의 형식
        - 미디어타입, 문자 인코딩
        - text/html, application/json, image/png
    - Content-Encoding: 표현 데이터의 압축방식
        - 데이터를 전달하는 곳에서 압축 후 인코딩 헤더 추가
        - gzip, deflate, identity (어떻게 압축했는지)
    - Content-Language: 표현 데이터의 자연언어
        - ko, en, en-US
    - Content-Length: 표현 데이터의 길이
        - 바이트단위
- 협상(컨텐츠 네고시에이션)
    - Accept: 클라이언트가 선호하는 미디어 타입 전달    
    - Accept-Charset: 클라이언트가 선호하는 문자 인코딩
    - 표현과 동일...
    - 협상 헤더는 요청시에만 사용
- 전송방식
    - 단순
        - Content-Length
    - 압축
        - Content-Encoding: gzip
    - 분할
        - Transfer-Encoding: chunked
        - Content-Length 넣으면 안됨
    - 범위
        - Content-Range: bytes 1001-2000/2000
- 일반정보
    - From
        - 일반적으로 사용 잘 안됨
        - 검색엔진 같은곳에서 사용
    - Referer
        - 현재 요청된 웹사이트의 이전 주소
        - A->B 이동하는 경우 Referer: A 를 포함해서 요청
        - 유입 경로 분석 가능
        - 요청에서 사용
    - Uset-Agent
        - 클라이언트의 애플리케이션 정보
        - 통계정보
        - 어떤 종류의 브라우저에서 장애가 발생 파악 가능
        - 요청에서 사용 
    - Server
        - 요청을 처리하는 ORIGIN 서버의 SW 정보
        - server: nginx
        - 응답에서 사용
    - Date
        - 메시지가 발생한 날찌와 시간
        - 응답에서 사용
- 특별한 정보
    - Host
        - 요청한 호스트 정보(도메인)
        - 요청에서 사용
        - 필수
        - 하나의 서버가 여러 도메인을 처리해야할 때
    - Location
        - 웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면 자동으로 이동 리다이렉트
    - Allow
        - 허용 가능한 HTTP 메서드
        - 405에서 응답에 포함해야함
        - Allow: GET, HEAD, PUT
- 인증
    - Authorization
        - 클라이언트에 인증 정보를 서버에 전달
        - Authorization: Basic xxxxxxx..
    - WWW-Authenticate
        - 리소스 접근 시 인증 방법 정의
        - 401 Unauthorized 응답과 함께 사용
        - WWW-Authenticate: Newauth realm="apps", type=1, title="login to app"...
- 쿠키
    - Set-Cookie: 서버에서 클라이언트로 쿠키 전달
    - Cookie: 클라이언트가 서버에서 받은 쿠키를 저장하고 Http 요청 시 전달 
    - `Http는 무상태 프로토콜`
    - 클라이언트와 서버는 서로 상태를 유지하지 않는다.
    - 사용처
        - 사용자 로그인 세션관리
        - 광고정보 트래킹
    - 쿠키 정보는 항상 서버에 전송됨
        - 네트워크 트래픽 추가유발
        - 최소한의 정보만 사용해야함
        - 서버에 전송하지않고, 웹 브라우저 내부에 데이터를 저장하고 싶으면 웹 스토리지 
    - 보안에 민감한 데이터는 저장하면 안됨
    - 생명주기
        - expires=Sat, 26-Dec-2020 04:32:11 GMT
        - Set-Cookie: max-age=3600
            - 0이나 음수를 지정하면 쿠키삭제
        - 만료날짜를 입력하면 해당 날짜까지 유지되지만, 만료날짜를 입력하지 않으면 브라우저 종료시 사라짐
    - 도메인
        - domain=aaa.org
        - 기준 도메인과 서브 도메인에 다 전송한다.
            - dev.aaa.org도 가능
    - 경로
        - path=/home
        - 이 경로를 포함한 하위 경로 페이지만 접근가능
        - 보통 `/`로 지정
    - Secure
        - Secure를 적용하면 https인 경우에만 적용
        - HttpOnly
            - Xss 공격방지
            - 자바스크립에서 접근 불가
            - http 전송에만 사용
        - SameSite
            - XSRF 공격방지
            - 요청 도메인과 쿠키에 설정된 도메인이 같은경우만 전송

## 캐시 기본 동작
- 캐시가 없을 때
    - 데이터가 변경되지 않아도 네트워크를 통해 데이터를 다운로드 받아야한다.
    - 브라우저 로딩속도가 느리다.
- 캐시 적용
    - cache-control: max-age=60
    - 브라우저 로딩속도 빠르다
    - 비싼 네트워크 비용을 줄인다.
- 캐시 시간 초과
    - 다시 다운로드를 받아야하나?
- 검증 헤더
    - 응답 Last-Modified: 2023 ~ => 데이터가 마지막 수정된 시간
    - 요청 if-modified-since: 2023 ~ 
    - 응답 요청 헤더 값이 같다면, HTTP/1.1 304 Not Modified, Http Body 없음
- 검증헤더와 조건부 요청
    - 캐시 유효시간이 초과해도, 서버의 데이터가 갱신되지 않으면  304 Not Modified, Http Body 없이 반환
    - 클라이언트는 서버가 보낸 응답 헤더 정보로 캐시의 메타 정보를 갱신
    - 클라이언트는 캐시에 저장되어 있는 데이터 재활용
    - 네트워크 다운로드가 발생하지만 용량이 적은 정보만 다운로드
- 검증헤더와 조건부 요청2
    - 검증헤더 : Last-Modified, ETag
        - 케시 데이터와 서버 데이터가 같은지 검증하는 데이터
    - 조건부 요청 헤더
        - 검즈헤더로 조건에 따른 분기
        - If-Modified-Since: Last-Modified 사용
        - If-None-Match: ETag 사용
        - 조건이 만족하면 200 OK
        - 조건 만족하지 않으면 304 Not Modified
- ETag
    - 캐시용 데이터에 임의의 고유한 버전을 달아둘 수 있음
        - ETag: "v1.0", ETag: "aerwa"
    - 데이터가 변경되면 이 이름을 바꿔서 변경
    - ETag만 보내서 같으면 유지, 다르면 다시 받는다.
    - cache-control: max-age=60, ETag="aaaaaa"
    - 캐시 제어 로직을 서버에서 완전히 관리

## 캐시 제어 헤더
- Cache-Control: 
    - max-age 캐시 유효 시간, 초단위
    - no-cache 데이터는 캐시해도 되지만, 항상 origin 서버에 검증하고 사용
    - no-store 데이터에 민감한 정보가 있으므로 저장하면 안됨
    - max-age 권장, expires는 max-age와 함께 사용되면 max-age로 채택

## 프록시 캐시
- Cache-Control: public
    - 응답이 public에 저장되어도됨
- Cache-Control: private
    - 사용자만을 위한것, private 캐시에 저장해야함(기본값)
- Cache-Control: s-maxage
    - 프록시 캐시에만 적용되는 max-age
- Age: 60 
    - 오리진 서버에서 응답 후 프록시 캐시내에 머문시간

## Cache-Control
- Cache-Control: no-cache, no-store, must-revalidate > 캐시가 절대 되면 안될 떄
- Pragma: no-cache
- no-cache
    - 데이터는 캐시해도 되지만, 항상 원서버에 검증해야함
- no-store
    - 데이터에 민감한 정보가 있으므로 저장하면 안됨
- must-revalidate
    - 캐시 만료 후 최초 조회시 원서버에 검증해야함
    - 원 서버 접근 실패시 반드시 오류가 발생함 - Gateway Timeout(504)