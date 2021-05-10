---
title:  "Linux Create MariaDB Server"

categories:
  - linux
tags:
  - linux
  - mariaDB
  - Server
last_modified_at: 2021-04-26T00:40:00-00:00
---

# Linux Quick start

## 1. Install WSL 
[wsl설치 : https://docs.microsoft.com/ko-kr/windows/wsl/install-win10](https://docs.microsoft.com/ko-kr/windows/wsl/install-win10)   
윈도우 상에서 Linux를 편하게 다루는 WSL이라는 도구가 있다. Microsoft에서 정식으로 지원하는 것이니 이제 윈도우에서도 편하게 Linux를 다룰 수 있다.   
우분투를 깔고 MariaDB Server를 설치해보자.   
설치 가이드 대로 따라 하면 된다.

## 2. Install Ubuntu


## 3. Install mariaDB
1. sudo apt update && sudo apt-get -y upgrade   
apt-get은 우분투(+데미안) 계열의 리눅스에서 쓰이는 패키지 관리 명령어 도구이다.   
해당 명령어는 레지파토리에서 업데이트 패키지가 있는지 체크하고 설치된 패키지를 업데이트하는 것이다.   
[참고 : https://blog.outsider.ne.kr/346](https://blog.outsider.ne.kr/346)   
   
2. sudo apt-get install -y mariadb-server
자동으로 mariadb 까지 설치가 된다.   
접속은 mysql -u root -p 로 확인해 보자.   
* mariaDB 10.4버전 부터는 sudo mysql 하여하 한다.
[참조 : https://www.nemonein.xyz/2019/07/2254/](https://www.nemonein.xyz/2019/07/2254/)   
  
```xml
sudo mysql
```
에서 권한이 없어서 아래와 같은 에러가 발생했다.   
* ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock'   

[참고 : https://jimnong.tistory.com/807](https://jimnong.tistory.com/807)   
  
windows PowerShell 에서
```xml
wsl --list --verbose
```
로 상태 확인하고
```xml
wsl -t ubuntu
```
로 종료하였다. 하지만 역시 인증 되지 않아 다음과 같은 방법을 사용했다.   
[참고 : https://thinkpro.tistory.com/16](https://thinkpro.tistory.com/16)   
$sudo visudo 에서 권한을 추가하였다.   
하지만 역시 실행이 되지 않았다.   

```xml
mysql <!--이 명령어도 실행이 되지 않아서-->
sudo service mysql restart <!--서버를 다시 실행시키고-->    
sudo mysql -u root -p mysql <!--이걸로 mysql이 들어가졌다.-->
sudo mysql <!--최종 실행 -->
```
😀결과
```markdown
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 39
Server version: 10.3.25-MariaDB-0ubuntu0.20.04.1 Ubuntu 20.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

## 4. mariaDB setting
[참고 : https://jdm.kr/blog/132](https://jdm.kr/blog/132)     
```sql
create database db_name;
create user 'username'@'ipaddress' identified by 'password';
grant all privileges on db_name.* to 'username'@'ipaddress';
flush privileges;

-- 확인
select host, user from mysql.user;
```
😀결과
```markdown
+--------------+---------+
| host         | user    |
+--------------+---------+
| (ip address) | username|
| localhost    | root    |
+--------------+---------+
```

* 외부에서도 접속 가능하게 변경
root 권한으로 돌아가서 /etc/mysql/mariadb.conf.d/ 폴더에서 50-server.cnf 파일 읽어서
bind-address 부분을 0.0.0.0 으로 변경하면 외부어디에서나 접속 가능하다. (기본은 127.0.0.1)
  
🐤Tip : ubuntu에서 root 비밀번호 잃어버렸을 때
1. 윈도우에서 cmd 접속
2. 우분투 접속시 root로 접속하도록 설정 변경
    * ubuntu.exe config --default-user root 
3. 우분투 실행
4. passwd 명령어로 패스워드 변경
5. 다시 윈도우 cmd에서 내 기본계정으로 설정 변경
    
