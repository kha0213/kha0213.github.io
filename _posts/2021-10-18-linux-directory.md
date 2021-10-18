---
title: "Linux file system"

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
last_modified_at: 2021-10-18T00:40:00-00:00
---

# Linux file system

## Types of files
<table>
    <tr>
        <th>파일의 종류</th>
        <th>설명</th>
    </tr>
    <tr>
        <td>일반 파일(regular file)</td>
        <td>일반 파일. (텍스트, 동영상 등 기본 파일)</td>
    </tr>
    <tr>
        <td>디렉터리(directory)</td>
        <td>디렉터리 파일 (윈도우의 폴더 개념)</td>
    </tr>
    <tr>
        <td>심볼릭 링크 파일 (symbolic link file)</td>
        <td>다른 파일을 가리키는 파일 (윈도우의 바로가기)</td>
    </tr>
    <tr>
        <td>블록 디바이스 파일(block device file)</td>
        <td>블록 디바이스를 제어하기 위한 파일</td>
    </tr>
    <tr>
        <td>문자 디바이스 파일(character device file)</td>
        <td>문자 디바이스를 제어하기 위한 파일</td>
    </tr>
    <tr>
        <td>파이프 파일(pipe file)</td>
        <td>파이프를 나타내는 파일. (프로세스간 통신에 사용됨)</td>
    </tr>
    <tr>
        <td>소켓(socket)</td>
        <td>소켓을 나타내는 파일. (프로세스간 통신에 사용됨)</td>
    </tr>
</table>
# Linux directory type
<table>
    <tr>
        <th>디렉터리</th>
        <th>설명</th>
    </tr>
    <tr>
        <td>/</td>
        <td>루트 디렉터리. 모든 디렉터리의 최상위 부모</td>
    </tr>
    <tr>
        <td>/bin</td>
        <td>모든 사용자가 사용할 수 있는 여러 가지 실행 파일 위치</td>
    </tr>
    <tr>
        <td>/sbin</td>
        <td>시스템 관리자 권한으로 실행해야 하는 실행 파일 위치</td>
    </tr>
    <tr>
        <td>/etc</td>
        <td>여러 가지 설정 파일</td>
    </tr>
    <tr>
        <td>/lib</td>
        <td>공유 라이브러리 디렉터리</td>
    </tr>
    <tr>
        <td>/home</td>
        <td>사용자들의 홈 디렉터리</td>
    </tr>
    <tr>
        <td>/mnt</td>
        <td>일시적으로 파일 시스템에 마운트하는 경우 사용하는 디렉터리</td>
    </tr>
    <tr>
        <td>/proc, /sys</td>
        <td>시스템 정보를 설정/조회 할 수 있는 디렉터리</td>
    </tr>
    <tr>
        <td>/tmp</td>
        <td>임시 디렉터리</td>
    </tr>
    <tr>
        <td>/usr</td>
        <td>사용자가 추가한 실행 파일, 라이브러리 등의 소프트웨어 저장</td>
    </tr>
    <tr>
        <td>/dev</td>
        <td>디바이스 드라이버가 사용하는 디바이스 파일 디렉터리</td>
    </tr>
</table>
