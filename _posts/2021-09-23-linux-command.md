---
title: "Linux command"

toc: true
toc_label: "go page"
toc_icon: "book"
toc_sticky : true

classes: wide

categories:
  - linux
tags:
  - linux
  - command
last_modified_at: 2021-09-23T00:40:00-00:00
---

# Linux command
리눅스에서는 명령어 하나에 여러 옵션들을 주어서 여러가지 기능들을 가능하게 한다.
대략적인 명령어 소개와 자세한 쓰임새를 알아보자.
<table>
    <tr>
        <th>Keyword</th>
        <th>Description</th>
    </tr>
    <tr>
        <td>man</td>
        <td>메뉴얼 조회</td>
    </tr>
    <tr>
        <td>ls, cat, head, tail</td>
        <td>파일 목록/내용 조회</td>
    </tr>
    <tr>
        <td>grep, find</td>
        <td>검색/탐색</td>
    </tr>
    <tr>
        <td>tar, zip/unzip, gzip/gunzip</td>
        <td>파일 압축/압축풀기</td>
    </tr>
    <tr>
        <td>date, cal</td>
        <td>시간/날짜</td>
    </tr>
    <tr>
        <td>sudo</td>
        <td>관리자 권한</td>
    </tr>
    <tr>
        <td>apt, yum</td>
        <td>패키지 매니저</td>
    </tr>
    <tr>
        <td>nano</td>
        <td>텍스트에디터</td>
    </tr>
    <tr>
        <td>echo, exit, history</td>
        <td>기타 명령어</td>
    </tr>
</table>


## 1. 기본 명령어
### man (--help)
### ls (ll)
### cd
### cat
### touch
### echo
echo 명령어는 텍스트를 출력스트림으로 보여준다.

echo $! : 마지막으로 백그라운드에서 실행된 명령어의 PID 값을 출력한다.
echo $? : 마지막으로 종료된 명령어의 종료 상태를 말해준다. (정상 종료인 경우 0)
echo $$ : 현재 쉘의 PID값을 출력한다.

### alias

## 2. 텍스트 처리

### head
파일의 앞 부분 출력하는 명령어이다.
<pre>
head [option]... [file]...
</pre>

* 자주 사용되는 옵션
  -c, --bytes=[-]NUM : NUM byte만 출력
  -n, --lines=[-]NUM : NUM line만 출력 (default 10라인)
  (-n에 -를 주게 되면 밑에 해당 NUM을 제외한 전체 출력 ex) head -n -20 :밑의 20라인 제외 후 처음부터 출력)
  ex) head -n 100 test.txt

### tail
head가 앞 부분이라면 tail은 파일의 뒷 부분을 출력해준다.
<pre>
tail [option]... [file]...
</pre>
* 자주 사용되는 옵션
  -f : 파일에 추가되는 내용 실시간 반영 출력 (로그 볼 때 중요)
  -F : -f와 같은 기능에 추가로 파일이 삭제 후 재생성 되어도 따라간다. (로그같이 자동으로 이름 바뀌고 새로 만들어질 때 유용) (--follow=name --retry와 같다.)
  -c, --bytes=[-]NUM : NUM byte만 출력
  -n, --lines=[-]NUM : NUM line만 출력 (default 10라인)
  (-n에 +로 주게 되면 해당 라인부터 끝까지 출력한다. ex) tail -n +10 :10번째 라인부터 끝까지 출력)

### wc (word count)
파일의 line, word, byte count 출력하는 명령어이다. 
<pre>
wc [option]... [file]...
</pre>
* 자주 사용되는 옵션
-l : 라인수 만 출력

😊TIP : wc 뒤에 여러 파일을 입력하면 각각 파일 정보와 전체 정보까지 나온다.

### nl (line number)
파일 내용을 라인 넘버와 함께 출력
<pre>
nl [option]... [file]...
</pre>

* 자주 사용되는 옵션
  -ba : 모든 라인에 대해 라인 넘버링 (default는 빈 라인은 건너뛰기 하고 넘버링 됨)
  -v N : 시작 라인 넘버를 N으로 지정
  -s : 라인 넘버 출력 후 출력할 separator 지정

### sort
정렬하여 출력한다.
<pre>
sort [option]... [file]...
</pre>
* 자주 사용되는 옵션
  위치 기준
  -k : key에 의한 정렬 수행
  F[.C][OPTS]
  -t : 필드 구분자(기본값은 공백 문자)
  ex) sort -t: -k 3 (:를 구분자로 3번째 구분을 기준으로 sort한다.)

정렬 기준
-f, --ignore-case : 대소문자 무시하고 출력
-g, --general-numeric-sort :
-n, --numeric-sort : 숫자를 기준으로 정렬 (ex 12와 2 이면 12가 뒤에)
-r, --reverse : 역순 출력
-u, --unique : (내용이 같은 경우 하나만 출력)

😊Tip : 마지막에 --debug를 붙이면 sort 기준을 확인 할 수 있다.

### uniq
중복된 내용을 제거하고 출력

<pre>
uniq [option]... [&lt;input&gt; [&lt;출력&gt;]]
</pre>

* 자주 사용되는 옵션
  -d, --repeated : 중복된 내용만 출력
  -u, --unique : 중복되지 않은 내용만 출력
  -i, --ignore-case : 대소문자

### cut
### tr
### sed
### awk

## 3. 검색

### find
find 명령어는 주로 파일이나 디렉토리의 위치를 찾을 때 사용한다. 파일의 이름, 생성시간, 소유자, 크기, 위치 등을 기반한 검색으로 찾을 수 있다.
<pre>
find [-H] [-L] [-P] [-Olevel] [-D debugopts] [path...] [expression]
</pre>

find 명령어의 기본 사용법은
find [경로] [옵션] [조건] [실행] 이다.


OPTION
-P        : 심볼릭 링크를 따라가지 않고, 심볼릭 링크 자체 정보 사용.
-L        : 심볼릭 링크에 연결된 파일 정보 사용.
-H        : 심볼릭 링크를 따라가지 않으나, Command Line Argument를 처리할 땐 예외.

EXPRESSION
<table>
    <tr>
        <td>name [PATTERN]</td>
        <td>지정된 문자열 패턴에 해당하는 파일 검색.</td>
    </tr>
    <tr>
        <td>empty</td>
        <td>빈 디렉토리 또는 크기가 0인 파일 검색.</td>
    </tr>
    <tr>
        <td>delete</td>
        <td>검색된 파일 또는 디렉토리 삭제.</td>
    </tr>
    <tr>
        <td>exec</td>
        <td>검색된 파일에 대해 지정된 명령 실행.</td>
    </tr>
    <tr>
        <td>path [PATTERN]</td>
        <td>지정된 문자열 패턴에 해당하는 경로에서 검색. </td>
    </tr>
    <tr>
        <td>print</td>
        <td>검색 결과를 출력. 검색 항목은 newline으로 구분. (기본 값)</td>
    </tr>
    <tr>
        <td>print0</td>
        <td>검색 결과를 출력. 검색 항목은 null로 구분.</td>
    </tr>
    <tr>
        <td>size [N]</td>
        <td>파일 크기를 사용하여 파일 검색.</td>
    </tr>    
    <tr>
        <td>type []</td>
        <td>지정된 파일 타입에 해당하는 파일 검색.</td>
    </tr>    
    <tr>
        <td>mindepth [LEVELS]</td>
        <td>검색을 시작할 하위 디렉토리 최소 깊이 지정. (조건 중 가장 먼저 걸어야 한다.)</td>
    </tr>
    <tr>
        <td>maxdepth [LEVELS]</td>
        <td>검색할 하위 디렉토리의 최대 깊이 지정. (조건 중 가장 먼저 걸어야 한다.)</td>
    </tr>
    <tr>
        <td>atime [N]</td>
        <td>파일 접근(access) 시각을 기준으로 파일 검색.</td>
    </tr>
    <tr>
        <td>ctime [N]</td>
        <td>파일 내용 및 속성 변경(change) 시각을 기준으로 파일 검색.</td>
    </tr>
    <tr>
        <td>mtime [N]</td>
        <td>파일의 데이터 수정(modify) 시각을 기준으로 파일 검색.</td>
    </tr>
</table>

표현식에는 연산자를 사용하여 두 개 이상의 표현식 조합이 가능하다.   
-a 나 -and로 둘 다 만족하는 조건을 찾고, -o, -or로 둘 중 하나 만족하는 조건을 찾을 수 있다. (default는 and이다.)
{: .notice--info}

### grep
### apropos
### locate
### which

## 시스템 정보

### ps
ps는 프로세스 관련 명령어이다.
ps [option] 으로 관련 명령을 볼 수 있고 가장 많이 사용하는 명령어는 ps -ef 이다.
<table>
    <tr>
        <th>옵션</th>
        <th>설명</th>
    </tr>
    <tr>
        <td>-A</td>
        <td>모든 프로세스를 출력한다.</td>
    </tr>
    <tr>
        <td>-a</td>
        <td>세션 리더를 제외하고 데몬 프로세서처럼 터미널에 종속되지 않은 모든 프로세스를 출력한다.</td>
    </tr>
    <tr>
        <td> -e </td>
        <td>커널 프로세스를 제외한 모든 프로세스를 출력해준다.</td>
    </tr>
    <tr>
        <td> -f </td>
        <td>풀 포맷으로 보여준다.</td>
    </tr>
    <tr>
        <td>-l</td>
        <td>긴 포맷으로 보여준다.</td>
    </tr>
    <tr>
        <td>-o [type]</td>
        <td> (pid, tty, time, cmd) 등을 타입으로 입력하여 해당 것에 대한 조회를 할 수 있다.</td>
    </tr>
    <tr>
        <td>-m</td>
        <td>프로세스들 뿐만 아니라 커널 스레드들도 보여준다.</td>
    </tr>
    <tr>
        <td>u</td>
        <td>프로세스의 소유자를 기준으로 출력한다.</td>
    </tr>
    <tr>
        <td>-u</td>
        <td>특정 프로세스 정보를 확일할 때 사용한다. (기본은 현재 사용자)</td>
    </tr>
    <tr>
        <td>-x</td>
        <td>로그인 상태에 있는 동안 아직 완료되지 않은 프로세서들을 보여준다.</td>
    </tr> 
</table>

일반적으로 ef옵션을 가장 많이 사용한다. (모든 프로세서를 풀 옵션으로 보기)

![ps]({{ site.baseurl }}/assets/images/study/linux/linux-ps.png)

이제 출력되는 결과의 뜻을 확인해보자.

<table>
    <tr>
        <th>항목</th>
        <th>의미</th>
    </tr>
    <tr>
        <td>UID (or USER)</td>
        <td>프로세스 소유자의 이름</td>
    </tr>
    <tr>
        <td>PID</td>
        <td>프로세스 식별 번호</td>
    </tr>
    <tr>
        <td>PPID</td>
        <td>부모 프로세스ID</td>
    </tr>
    <tr>
        <td>%CPU</td>
        <td>CPU 사용 비율의 추정치 (BSD계열)</td>
    </tr>
    <tr>
        <td>%MEM</td>
        <td>메모리의 사용 비율의 추정치 (BSD계열)</td>
    </tr>
    <tr>
        <td>VSZ</td>
        <td>K단위 또는 페이지 단위의 가상메모리 사용량</td>
    </tr>
    <tr>
        <td>RSS</td>
        <td>실제 메모리 사용량</td>
    </tr>
    <tr>
        <td>TTY</td>
        <td>프로세서와 연결된 터미널</td>
    </tr>
    <tr>
        <td>S (or STAT)</td>
        <td>현재 프로세스 상태코드</td>
    </tr>
    <tr>
        <td>TIME</td>
        <td>총 CPU 사용 시간</td>
    </tr>
    <tr>
        <td>COMMAND</td>
        <td>프로세스의 실행 명령행</td>
    </tr>
    <tr>
        <td>STIME</td>
        <td>프로세스가 시작된 시간</td>
    </tr>
    <tr>
        <td>C (or CP)</td>
        <td>짧은 기간 동안의 CPU 사용률</td>
    </tr>
    <tr>
        <td>F</td>
        <td>프로세스의 플래그</td>
    </tr>
    <tr>
        <td>PRI</td>
        <td>실제 실행 우선순위</td>
    </tr>
    <tr>
        <td>NI</td>
        <td>nice 우선순위 번호</td>
    </tr>
</table>

### kill
kill 명령어는 프로세스에 시그널을 보내주는 명령어 이다.
강제종료를 위해 SIGKILL 시그널을 보내면 프로그램이 무조건 즉시 종료하게 되지만 이는 좋지 않은 방법이다.   
SIGTERM이나 SIGINT 등의 종료 시그널은 시그널 핸들러를 등록하여 Socket close나 file close 같은 종료 작업을 한 뒤에 종료처리 하여야 하는데
SIGKILL은 핸들러를 두지 못해 즉시 종료하므로 저장중인 데이터가 유실될 우려가 있다.

<pre>
    kill [options] &lt;pid&gt; [...]
</pre>

kill 뒤에 시그널을 보내 명령을 내릴 수 있다.
주로 쓰이는 명령어는
kill <pid> : SIGTERM (종료) 한다. default 가 15번 이다.
kill -9 <pid> : 강제종료한다. (보통 SIGTERM 명령 먼저 보내보고 안될시 SIGKILL이 안전하다.)   
kill -l : 시그널 리스트를 확인한다.

![kill]({{ site.baseurl }}/assets/images/study/linux/kill.png)

### top
### lsof
### netstat
### sysctl
### df
### du
### dmidecode
### lscpu
### free

## vim

## 개발 도구
### diff
### patch
### ctags
### cscope
### strace

### apt
apt는 리눅스 패키지 매니저이다.
apt install	apt-get install	패키지 목록
apt remove	apt-get remove	패키지 삭제
apt purge	apt-get purge	패키지와 관련 설정 제거
apt update	apt-get update	레파지토리 인덱스 갱신
apt upgrade	apt-get upgrade	업그레이드 가능한 모든 패키지 업그레이드
apt autoremove	apt-get autoremove	불필요한 패키지 제거
apt full-upgrade	apt-get dist-upgrade	의존성 고려한 패키지 업그레이드
apt search	apt-cache search	프로그램 검색
apt show	apt-cache show	패키지 상세 정보 출력
apt list apt 목록 출력

# Linux all command
리눅스에서는 명령어 하나에 여러 옵션들을 주어서 여러가지 기능들을 가능하게 한다.
대략적인 명령어 소개와 자세한 쓰임새를 알아보자.
<table>
    <tr>
        <th>Keyword</th>
        <th>Description</th>
        <th>Example</th>
    </tr>
    <tr>
        <td>cd</td>
        <td>디렉토리 이동</td>
        <td>cd /home (home으로 이동)</td>
    </tr>
    <tr>
        <td>pwd</td>
        <td>현재 디렉토리 위치 출력</td>
        <td></td>
    </tr>
    <tr>
        <td>clear</td>
        <td>콘솔 화면 클리어</td>
        <td></td>
    </tr>
    <tr>
        <td>tail, head</td>
        <td>파일 끝에서(앞에서)부터 찾기</td>
        <td>tail -n 100 server.log</td>
    </tr>
    <tr>
        <td>more, less</td>
        <td>(주로) 긴 파일 읽기</td>
        <td>more -d server.log</td>
    </tr>
    <tr>
        <td>cat</td>
        <td>파일의 내용 전부를 화면에 출력한다.</td>
        <td>cat file1 file2 2개 파일 연결하여 출력</td>
    </tr>
    <tr>
        <td>grep</td>
        <td>각 파일에서 패턴이 일치하는 행을 찾아낸다</td>
        <td>grep -i 'hello word' file1 file2</td>
    </tr>
    <tr>
        <td>find</td>
        <td>해당 이름을 가진 파일의 위치를 찾는다.</td>
        <td>find /etc -name "*.conf"</td>
    </tr>
    <tr>
        <td>date</td>
        <td>날짜 관련 출력</td>
        <td>date +$D</td>
    </tr>
    <tr>
        <td>file</td>
        <td>해당 파일의 정보를 알 수 있다.</td>
        <td>file filelist.gz</td>
    </tr>
    <tr>
        <td>echo</td>
        <td>(메아리) 해당 내용을 출력한다.</td>
        <td>echo $PWD</td>
    </tr>
    <tr>
        <td>env</td>
        <td>환경변수를 확인한다.</td>
        <td>env</td>
    </tr>
    <tr>
        <td>cal</td>
        <td>달력을 확인한다.</td>
        <td>cal -A4</td>
    </tr>
    <tr>
        <td>history</td>
        <td>명령어 기록을 출력한다.</td>
        <td>history</td>
    </tr>
    <tr>
        <td>exit</td>
        <td>실행중인 쉘을 종료한다.</td>
        <td>exit</td>
    </tr>
    <tr>
        <td>sudo</td>
        <td>해당 계정으로 실행한다. (default 관리자 계정)</td>
        <td>sudo apt install hello</td>
    </tr>
    <tr>
        <td>apt</td>
        <td>패키지 매니저 관련 정보가 나타난다. list, install ...</td>
        <td>apt list | grep hello</td>
    </tr>
    <tr>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td> > </td>
        <td>파일로 저장해라</td>
        <td>find > filelist (find의 출력결과를 filelist라는 파일로 저장)</td>
    </tr>
</table>
