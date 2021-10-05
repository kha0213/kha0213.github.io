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

## 1. man
메뉴얼 소개 명령어이다.

## 2. ls, cat, head, tail
파일 목록/내용 조회 명령어이다.

## 3. find, grep
파일 탐색에 관련된 명령어이다. find는 주로 파일이나 디렉토리의 위치를 찾을 때 쓰고
grep는 해당 파일 안의 내용을 찾을 때 쓴다.
### 3-1 find
find 명령어는 주로 파일의 위치를 찾을 때 사용한다. 파일의 이름, 생성시간, 소유자, 크기, 위치 등을 기반한 검색으로 찾을 수 있다.   
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

```markdown

```

표현식에는 연산자를 사용하여 두 개 이상의 표현식 조합이 가능하다.   
-a 나 -and로 둘 다 만족하는 조건을 찾고, -o, -or로 둘 중 하나 만족하는 조건을 찾을 수 있다. (default는 and이다.)
{: .notice--info}

## 4. apt
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
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td></td>
        <td></td>
        <td></td>
    </tr>
</table>