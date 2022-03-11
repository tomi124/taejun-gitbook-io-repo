# Linux basic command

## 기본 명령어 <a href="#ea-b8-b0-eb-b3-b8-eb-aa-85-eb-a0-b9-ec-96-b4" id="ea-b8-b0-eb-b3-b8-eb-aa-85-eb-a0-b9-ec-96-b4"></a>

### 경로 🔥 <a href="#ea-b2-bd-eb-a1-9c-f0-9f-94-a5" id="ea-b2-bd-eb-a1-9c-f0-9f-94-a5"></a>

#### CLI 상에서 나의 위치를 알려면? <a href="#cli-ec-83-81-ec-97-90-ec-84-9c-eb-82-98-ec-9d-98-ec-9c-84-ec-b9-98-eb-a5-bc-ec-95-8c-eb-a0-a4-eb-a9" id="cli-ec-83-81-ec-97-90-ec-84-9c-eb-82-98-ec-9d-98-ec-9c-84-ec-b9-98-eb-a5-bc-ec-95-8c-eb-a0-a4-eb-a9"></a>

`$ pwd`

#### 절대경로 <a href="#ec-a0-88-eb-8c-80-ea-b2-bd-eb-a1-9c" id="ec-a0-88-eb-8c-80-ea-b2-bd-eb-a1-9c"></a>

* 특정한 위치를 기준으로 경로를 설정
* 최상위 루트 (root): `/`
* 사용자의 홈 디렉토리 (user): `~/`

#### 상대경로 <a href="#ec-83-81-eb-8c-80-ea-b2-bd-eb-a1-9c" id="ec-83-81-eb-8c-80-ea-b2-bd-eb-a1-9c"></a>

* 현재 나의 위치를 기준으로 경로를 설정
* 현재 위치: `./`
* 상위 디렉토리: `../`
* 상위의 상위 디렉토리: `../../`

### 이동 <a href="#ec-9d-b4-eb-8f-99" id="ec-9d-b4-eb-8f-99"></a>

#### 디렉토리 이동 <a href="#eb-94-94-eb-a0-89-ed-86-a0-eb-a6-ac-ec-9d-b4-eb-8f-99" id="eb-94-94-eb-a0-89-ed-86-a0-eb-a6-ac-ec-9d-b4-eb-8f-99"></a>

`$ cd /var/log`

#### 사용자(본인) 홈 디렉토리 <a href="#ec-82-ac-ec-9a-a9-ec-9e-90-eb-b3-b8-ec-9d-b8-ed-99-88-eb-94-94-eb-a0-89-ed-86-a0-eb-a6-ac" id="ec-82-ac-ec-9a-a9-ec-9e-90-eb-b3-b8-ec-9d-b8-ed-99-88-eb-94-94-eb-a0-89-ed-86-a0-eb-a6-ac"></a>

`$ cd ~/`

#### 특정 사용자 홈 디렉토리 <a href="#ed-8a-b9-ec-a0-95-ec-82-ac-ec-9a-a9-ec-9e-90-ed-99-88-eb-94-94-eb-a0-89-ed-86-a0-eb-a6-ac" id="ed-8a-b9-ec-a0-95-ec-82-ac-ec-9a-a9-ec-9e-90-ed-99-88-eb-94-94-eb-a0-89-ed-86-a0-eb-a6-ac"></a>

`$ cd /Users/<특정사용자아이디>` (macOS)

### 디렉토리와 파일의 목록을 출력 <a href="#eb-94-94-eb-a0-89-ed-86-a0-eb-a6-ac-ec-99-80-ed-8c-8c-ec-9d-bc-ec-9d-98-eb-aa-a9-eb-a1-9d-ec-9d-84-e" id="eb-94-94-eb-a0-89-ed-86-a0-eb-a6-ac-ec-99-80-ed-8c-8c-ec-9d-bc-ec-9d-98-eb-aa-a9-eb-a1-9d-ec-9d-84-e"></a>

`$ ls -1`

`$ ls -l`

`$ ls -alF`

`$ ls /`

`$ ls /Users/<사용자아이디>`

`$ ll (alias ls -al)` → 가장 많이 사용하게될 명령어 😍

### Glob pattern 🔥 <a href="#glob-pattern-f0-9f-94-a5" id="glob-pattern-f0-9f-94-a5"></a>

임의의 길이를 가지는 문자열: \*\
임의의 한 글자: ?\
글자의 집합: \[abcw-z]

#### 예시 <a href="#ec-98-88-ec-8b-9c" id="ec-98-88-ec-8b-9c"></a>

`$ ls *`

`$ ls .*`

`$ ls [a-f]*.txt`

`$ ls *.tx?`

### 화면 청소 <a href="#ed-99-94-eb-a9-b4-ec-b2-ad-ec-86-8c" id="ed-99-94-eb-a9-b4-ec-b2-ad-ec-86-8c"></a>

`$ clear`

`Ctrl + L`

`Command + K` (terminal clear buffer)

### 지난 명령어 활용 <a href="#ec-a7-80-eb-82-9c-eb-aa-85-eb-a0-b9-ec-96-b4-ed-99-9c-ec-9a-a9" id="ec-a7-80-eb-82-9c-eb-aa-85-eb-a0-b9-ec-96-b4-ed-99-9c-ec-9a-a9"></a>

**조회**

`$ history`

**12번 명령 재실행**

`$ !12`

**커맨드의 prefix 로 재실행**

`$ !ls`

**바로 직전 명령 재실행**

`$ !!`

**바로 직전 명령의 argument**

`$ ls -alF` → `$ ls !$`

**직전 명령 편집**

`$ echo "hello world"`

`^world^java`

→ `$ echo "hello java"`

### 명령어 편집 🔥 <a href="#eb-aa-85-eb-a0-b9-ec-96-b4-ed-8e-b8-ec-a7-91-f0-9f-94-a5" id="eb-aa-85-eb-a0-b9-ec-96-b4-ed-8e-b8-ec-a7-91-f0-9f-94-a5"></a>

**명령어 맨 앞 또는 끝으로 이동**

`Ctrl + A`

`Ctrl + E`

**단어 단위 이동**

`Option + 방향키`

### 서버 접속 및 종료 🔥 <a href="#ec-84-9c-eb-b2-84-ec-a0-91-ec-86-8d-eb-b0-8f-ec-a2-85-eb-a3-8c-f0-9f-94-a5" id="ec-84-9c-eb-b2-84-ec-a0-91-ec-86-8d-eb-b0-8f-ec-a2-85-eb-a3-8c-f0-9f-94-a5"></a>

**게이트웨이 접속 방법**

`$ ssh <아이디>@hcon.nhnent.com`

`$ ssh <아이디>@gw-dev.godo.co.kr`

**종료**

`$ logout 또는 exit`

**사용자 조회**

`$ last`

### 커맨드 설명 보는 법 🔥 <a href="#ec-bb-a4-eb-a7-a8-eb-93-9c-ec-84-a4-eb-aa-85-eb-b3-b4-eb-8a-94-eb-b2-95-f0-9f-94-a5" id="ec-bb-a4-eb-a7-a8-eb-93-9c-ec-84-a4-eb-aa-85-eb-b3-b4-eb-8a-94-eb-b2-95-f0-9f-94-a5"></a>

`$ man —help`

`$ man ssh`

`$ man 2 open`

`$ man 3 opendir`

**질문**\
\


* man 에서 빠져나오는 방법은?
* man 다음 붙은 2, 3의 의미는?

#### 설명 보는 법 <a href="#ec-84-a4-eb-aa-85-eb-b3-b4-eb-8a-94-eb-b2-95" id="ec-84-a4-eb-aa-85-eb-b3-b4-eb-8a-94-eb-b2-95"></a>

**정답**\
\


* q
* man man

### 명령어가 있는지 확인하는 방법 <a href="#eb-aa-85-eb-a0-b9-ec-96-b4-ea-b0-80-ec-9e-88-eb-8a-94-ec-a7-80-ed-99-95-ec-9d-b8-ed-95-98-eb-8a-94-e" id="eb-aa-85-eb-a0-b9-ec-96-b4-ea-b0-80-ec-9e-88-eb-8a-94-ec-a7-80-ed-99-95-ec-9d-b8-ed-95-98-eb-8a-94-e"></a>

**which**\
\


* which lsof
* PATH 환경 변수에 포함되어야 탐색 가능

**whereis**\
\


* whereis lsof

### 문자열 출력하기 <a href="#eb-ac-b8-ec-9e-90-ec-97-b4-ec-b6-9c-eb-a0-a5-ed-95-98-ea-b8-b0" id="eb-ac-b8-ec-9e-90-ec-97-b4-ec-b6-9c-eb-a0-a5-ed-95-98-ea-b8-b0"></a>

**echo**\
\


* `$ echo "hello world"`
* `$ echo -n "hello world"; echo " java coffee"`

### 현재 접속해있는 서버의 이름과 OS 정보를 확인 <a href="#ed-98-84-ec-9e-ac-ec-a0-91-ec-86-8d-ed-95-b4-ec-9e-88-eb-8a-94-ec-84-9c-eb-b2-84-ec-9d-98-ec-9d-b4-e" id="ed-98-84-ec-9e-ac-ec-a0-91-ec-86-8d-ed-95-b4-ec-9e-88-eb-8a-94-ec-84-9c-eb-b2-84-ec-9d-98-ec-9d-b4-e"></a>

* `$ uname`
* `$ uname -a`

#### 비슷한 명령 <a href="#eb-b9-84-ec-8a-b7-ed-95-9c-eb-aa-85-eb-a0-b9" id="eb-b9-84-ec-8a-b7-ed-95-9c-eb-aa-85-eb-a0-b9"></a>

* `$ hostname`

### 나의 사용자 아이디를 알려면? <a href="#eb-82-98-ec-9d-98-ec-82-ac-ec-9a-a9-ec-9e-90-ec-95-84-ec-9d-b4-eb-94-94-eb-a5-bc-ec-95-8c-eb-a0-a4-e" id="eb-82-98-ec-9d-98-ec-82-ac-ec-9a-a9-ec-9e-90-ec-95-84-ec-9d-b4-eb-94-94-eb-a5-bc-ec-95-8c-eb-a0-a4-e"></a>

* `$ id`
* `$ whoami`
* `echo "$USER"`

### 파일 보기 🔥 <a href="#ed-8c-8c-ec-9d-bc-eb-b3-b4-ea-b8-b0-f0-9f-94-a5" id="ed-8c-8c-ec-9d-bc-eb-b3-b4-ea-b8-b0-f0-9f-94-a5"></a>

```
$ echo "title: log.log <Key Input: Return>
content1: hello world <Key Input: Return>
content2: linux command tutorial <Key Input: Return>
eof: end of file" >> log.log

OR

echo "title: log.log\ncontent1: hello world\ncontent2: linux command tutorial\neof: end of file" >> log.log
```

`$ vi log.log`

`$ cat log.log`

`$ more log.log`

`$ grep hello log.log | less -eMR`

### 파일 일부만 보기 <a href="#ed-8c-8c-ec-9d-bc-ec-9d-bc-eb-b6-80-eb-a7-8c-eb-b3-b4-ea-b8-b0" id="ed-8c-8c-ec-9d-bc-ec-9d-bc-eb-b6-80-eb-a7-8c-eb-b3-b4-ea-b8-b0"></a>

`$ head -1 log.log`

`$ tail -2 log.log`

`$ tail -f /var/log/lastlog` 💙

### 찾기 <a href="#ec-b0-be-ea-b8-b0" id="ec-b0-be-ea-b8-b0"></a>

`$ grep [문자열] [파일]`

`$ grep "tutorial" log.log`

`$ grep -r "bash tutorial" ./*.log`

### 잘라내서 쓰기 <a href="#ec-9e-98-eb-9d-bc-eb-82-b4-ec-84-9c-ec-93-b0-ea-b8-b0" id="ec-9e-98-eb-9d-bc-eb-82-b4-ec-84-9c-ec-93-b0-ea-b8-b0"></a>

`$ cut -d [구분자] -f [필드 번호 파일]`

`$ cat log.log | cut -d : -f 1`

`$ cat log.log | cut -d : -f 2`

### 파일 관련 명령어 <a href="#ed-8c-8c-ec-9d-bc-ea-b4-80-eb-a0-a8-eb-aa-85-eb-a0-b9-ec-96-b4" id="ed-8c-8c-ec-9d-bc-ea-b4-80-eb-a0-a8-eb-aa-85-eb-a0-b9-ec-96-b4"></a>

파일 생성, 복사, 이동, 삭제\
\


* touch
* cp
* mv
* rm

디렉토리 생성, 이동, 삭제\
\


* mkdir
* cp -r
* mv
* rmdir
* rm -r

***

## 자주 쓰는 명령어 <a href="#ec-9e-90-ec-a3-bc-ec-93-b0-eb-8a-94-eb-aa-85-eb-a0-b9-ec-96-b4" id="ec-9e-90-ec-a3-bc-ec-93-b0-eb-8a-94-eb-aa-85-eb-a0-b9-ec-96-b4"></a>

#### 디렉토리 이름 다루기 <a href="#eb-94-94-eb-a0-89-ed-86-a0-eb-a6-ac-ec-9d-b4-eb-a6-84-eb-8b-a4-eb-a3-a8-ea-b8-b0" id="eb-94-94-eb-a0-89-ed-86-a0-eb-a6-ac-ec-9d-b4-eb-a6-84-eb-8b-a4-eb-a3-a8-ea-b8-b0"></a>

`$ dirname /etc/passwd`

#### 파일 이름 구하기 <a href="#ed-8c-8c-ec-9d-bc-ec-9d-b4-eb-a6-84-ea-b5-ac-ed-95-98-ea-b8-b0" id="ed-8c-8c-ec-9d-bc-ec-9d-b4-eb-a6-84-ea-b5-ac-ed-95-98-ea-b8-b0"></a>

`$ basename /etc/passwd`

### 입력 개별 처리하기 🔥 <a href="#ec-9e-85-eb-a0-a5-ea-b0-9c-eb-b3-84-ec-b2-98-eb-a6-ac-ed-95-98-ea-b8-b0-f0-9f-94-a5" id="ec-9e-85-eb-a0-a5-ea-b0-9c-eb-b3-84-ec-b2-98-eb-a6-ac-ed-95-98-ea-b8-b0-f0-9f-94-a5"></a>

`xargs` 를 이용하여 라인 단위로 처리하기

`$ xargs [명령]`

`$ xargs -I % 명령 "%"`

`$ find /etc/init | xargs wc -l`

`$ find /etc/init | xargs -I % wc -l "%"`

### 정렬 <a href="#ec-a0-95-eb-a0-ac" id="ec-a0-95-eb-a0-ac"></a>

**정렬**\
`$ sort log.log`

**중복 제거**\
`$ uniq [파일]`

### 문자, 단어, 라인 수 세기 <a href="#eb-ac-b8-ec-9e-90-2c-eb-8b-a8-ec-96-b4-2c-eb-9d-bc-ec-9d-b8-ec-88-98-ec-84-b8-ea-b8-b0" id="eb-ac-b8-ec-9e-90-2c-eb-8b-a8-ec-96-b4-2c-eb-9d-bc-ec-9d-b8-ec-88-98-ec-84-b8-ea-b8-b0"></a>

`$ wc [파일]`

문자: `$ wc -m`

단어: `$ wc -w`

라인 수: `$ wc -l`

### 파일 탐색 🔥 <a href="#ed-8c-8c-ec-9d-bc-ed-83-90-ec-83-89-f0-9f-94-a5" id="ed-8c-8c-ec-9d-bc-ed-83-90-ec-83-89-f0-9f-94-a5"></a>

`$ find <디렉토리> -name <이름 패턴> -type <파일유형>`

`$ find ./log`

`$ find ./log -name "*.log" -ls`

`$ find ./log -type d -name "log"`

### Pattern-directed scanning and processing language <a href="#pattern-directed-scanning-and-processing-language" id="pattern-directed-scanning-and-processing-language"></a>

#### awk <a href="#awk" id="awk"></a>

`$ cat ./log/log.log | sed 's/world/bash/g'`

`$ cat ./log/log.log | awk -F ":" '{printf("%s\t%s\n", $1, $2)}'`

### Stream EDitor 🔥 <a href="#stream-editor-f0-9f-94-a5" id="stream-editor-f0-9f-94-a5"></a>

#### sed <a href="#sed" id="sed"></a>

`$ cat ./log/log.log | sed 's/[^log]/k/g' | grep k`

### 파일 묶기, 압축 <a href="#ed-8c-8c-ec-9d-bc-eb-ac-b6-ea-b8-b0-2c-ec-95-95-ec-b6-95" id="ed-8c-8c-ec-9d-bc-eb-ac-b6-ea-b8-b0-2c-ec-95-95-ec-b6-95"></a>

* tar
* zip / unzip

### 프로세스 <a href="#ed-94-84-eb-a1-9c-ec-84-b8-ec-8a-a4" id="ed-94-84-eb-a1-9c-ec-84-b8-ec-8a-a4"></a>

프로세스 목록 조회

`$ ps aux / pstree`

프로세스 종료

`$ kill <프로세스 id>`

`pkill <프로세스명>`

### 시스템 <a href="#ec-8b-9c-ec-8a-a4-ed-85-9c" id="ec-8b-9c-ec-8a-a4-ed-85-9c"></a>

시스템 자원 사용 현황 조회

`$ top`

`$ iostat 1`

디렉토리 크기 조회

`$ du`

디렉토리 전체 사용 현황 조회

`$ df`

### 네트워크 명령 <a href="#eb-84-a4-ed-8a-b8-ec-9b-8c-ed-81-ac-eb-aa-85-eb-a0-b9" id="eb-84-a4-ed-8a-b8-ec-9b-8c-ed-81-ac-eb-aa-85-eb-a0-b9"></a>

`$ ifconfig`

`$ ping shopby.co.kr`

`$ ping 115.88.123.139`

`$ getent hosts localhost` (local machine)

`$ getent hosts shopby.co.kr` (gateway)

`$ traceroute`

`$ wget -0 log.log https://naver.com`

## 실습 <a href="#ec-8b-a4-ec-8a-b5" id="ec-8b-a4-ec-8a-b5"></a>

* 사용자의 홈 디렉토리의 서브 디렉토리를 디스크 사용량 순으로 정렬해서 보려면?
* `./log/log.log` 파일 내용 중 “linux command tutorial” 를 “<사용자이름> completed!” 라고 변경하고 저장하기

#### 정답 <a href="#ec-a0-95-eb-8b-b5" id="ec-a0-95-eb-8b-b5"></a>

* `$ du -ks * | sort -rn`
* `$ ls | xargs -I % sed -i "s/linux command tutorial/$USER completed\!/g" %`

***

## 쉘 스크립트 <a href="#ec-89-98-ec-8a-a4-ed-81-ac-eb-a6-bd-ed-8a-b8" id="ec-89-98-ec-8a-a4-ed-81-ac-eb-a6-bd-ed-8a-b8"></a>

### shebang <a href="#shebang" id="shebang"></a>

* \#! : 어떤 인터프리터 프로그램을 사용할 것인가를 지정
* \#!/bin/bash
* \#!/usr/bin/env python (권장)

### 파일 작성 <a href="#ed-8c-8c-ec-9d-bc-ec-9e-91-ec-84-b1" id="ed-8c-8c-ec-9d-bc-ec-9e-91-ec-84-b1"></a>

```
#!/bin/bash

echo "hello world"
```

실행\
`$ bash test.sh`

`$ chmod a+x test.sh`

`$ ./test.sh`

***

### Reference <a href="#reference" id="reference"></a>

[https://tldp.org/LDP/abs/html/index.html](https://tldp.org/LDP/abs/html/index.html)
