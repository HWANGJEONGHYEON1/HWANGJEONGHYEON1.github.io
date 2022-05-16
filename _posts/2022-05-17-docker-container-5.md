---
layout: post
title:  "도커 컨테이너 빌드업 - 5"
date:   2022-04-27 18:20:21 +0900
categories: docker
---

> # 

## 도커 볼륨 활용
- 지속성 유지
  - 디비컨테이너의 데이터 보호를 위해 볼륨을 지정할 수 있다. 만일 컨테이너의 장애로 인해 서비스가 중단되더라도 새로운 컨테이너의 동일 볼륨을 연결하면 디비의 table, data 모두 동일하게 지속할 수 있다.

```
# 볼륨을 포함한 MYSQL 컨테이너 실행
❯ docker run -it --name=mysql-vtest \
… ❯ -e MYSQL_ROOT_PASSWORD=mhylee \
… ❯ -e MYSQL_DATABASE=dockertest \
… ❯ -v mysql-data-vol:/var/lib/mysql -d \
… ❯ mysql:5.7

# MYSQL 컨테이너 접속 
mysql> use dockertest
Database changed
mysql> create table mytab(c1 int, c2 char);
Query OK, 0 rows affected (0.02 sec)

mysql> insert into mytab values(1, 'aa');
ERROR 1406 (22001): Data too long for column 'c2' at row 1
mysql> insert into mytab values(1, 'a');
Query OK, 1 row affected (0.01 sec)

mysql> select * from mytab;
+------+------+
| c1   | c2   |
+------+------+
|    1 | a    |
+------+------+
1 row in set (0.00 sec)

mysql> exit

❯ docker inspect --format="{{ .Mounts }}" mysql-vtest
[{volume mysql-data-vol /var/lib/docker/volumes/mysql-data-vol/_data /var/lib/mysql local z true }]

docker stop mysql-vtest
docker mysql-vtest



// mysql에 이상이 생겨 에러가 났다고 가정하고, 삭제를 하고 재 설치를 한다.

~
❯ docker rm mysql-vtest
mysql-vtest

~
❯ docker run -it --name=mysql-vtest \
-e MYSQL_ROOT_PASSWORD=mhylee \
-e MYSQL_DATABASE=dockertest \
-v mysql-data-vol:/var/lib/mysql -d \
mysql:5.7
c617d042d648ed8a317afaba8ec1be9d1ae322d8edcce02d825e3d0256342dc1

~
❯ docker exec -it mysql-vtest bash
root@c617d042d648:/# show databases;
bash: show: command not found
root@c617d042d648:/# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.38 MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databses;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'databses' at line 1
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| dockertest         |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql> use dockertest
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select * from mytab;
+------+------+
| c1   | c2   |
+------+------+
|    1 | a    |
+------+------+
1 row in set (0.00 sec)
```

- 환경 변수 사용은 컨테이너 구성에 영향을 미치기 때문에 반드시 확인해야한다.
```
# -e 
-e MYSQL_ROOT_PASSWORD=mhylee \ 
-e MYSQL_DATABASE=dockertest \
```