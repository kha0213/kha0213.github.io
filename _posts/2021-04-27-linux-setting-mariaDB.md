---
title:  "Linux Create MariaDB Server"

categories:
  - linux
tags:
  - linux
  - mariaDB
  - Server
last_modified_at: 2021-04-27T00:40:00-00:00
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


