---
title: "linux 계열 명령어 정리"
permalink: /os/linux/linux-overview/
excerpt: "linux command list"
last_modified_at: 2019-09-03T15:53:52-04:00
redirect_from:
  - /theme-setup/
toc: true
---

### linux란
---
#### 종류
리눅스는 여러 종류의 리눅스가 있습니다.  
리눅스 자체가 opensource이다 보니, 여러 기업에서 이를 이용하여,  
자신들의 리눅스를 만들어 제공한 거죠. \\
Ubuntu, Fedora, Centos 등등이 있습니다.

#### 계열
또한, 계열이 있는데, 이를 알고 있어야 편합니다. \\
몇몇 명령어가 다르기 때문이죠. \\
크게 Debian 계열, RedHat 계열로 구분합니다. \\
Debian 계열에는 대표적으로 Ubuntu가 있습니다. \\
RedHat 계열에는 Centos, Fedora 등이 있습니다. \\
Redhat 계열에 OS 조회하는 명령어를 치면, rhel라고 적히는데, \\
이는 Red Hat Enterprise Linux의 줄임말입니다. [참조](https://www.lesstif.com/pages/viewpage.action?pageId=20775405)

#### 설치 명령어
debian 계열에서는 설치 시 apt-get을 사용하고, \\
RedHat 계열에서는 rpm(RedHat Package Manager) 또는 yum(Yellowdog Update Modified)을 사용합니다. \\
둘 다 패키지 관리자로서는 동일하지만, \\
rpm은 설치 시 의존성 문제가 발생 할 수 있으며, 이를 해결 한 것이 yum입니다. 

만약, 이를 모르면... 저처럼 멘붕에 빠질 수 있습니다. \\
( A 프로그램을 설치해야해서 검색을 해 보니.. yum으로 한다고? 그런데 yum이 없네? \\
  yum 설치를 검색 해 보니, apt-get을 사용하라고? 그런데 apt-get이 없네? \\
  apt-get 설치를 검색 해 보니, yum으로 설치하라고?... 응?응? )
   

파일 또는 폴더의 경로 입력 시, 맨 앞에 /를 입력한다면 root부터의 경로로 인식합니다. \\
/부터 시작하지 않으면, 현재 위치부터 상대 경로로 인식합니다.

#### 명령어 옵션
명령어들은 각기 다른 옵션들을 가지고 있습니다. \\
이런 옵션들은 잘 모를 경우, help 옵션을 붙이면 정보를 제공받을 수 있습니다.
```
  $ ls --help

  Usage: ls [OPTION]... [FILE]...
  List information about the FILEs (the current directory by default).
  Sort entries alphabetically if none of -cftuvSUX nor --sort is specified.

  Mandatory arguments to long options are mandatory for short options too.
    -a, --all                  do not ignore entries starting with .
    -A, --almost-all           do not list implied . and ..
        --author               with -l, print the author of each file
    -b, --escape               print C-style escapes for nongraphic characters
        --block-size=SIZE      with -l, scale sizes by SIZE when printing them;
                                e.g., '--block-size=M'; see SIZE format below
    -B, --ignore-backups       do not list implied entries ending with ~
    -c                         with -lt: sort by, and show, ctime (time of last
   --- 생략 ---

```
이 때, 위에 예를 보면 알 수 있듯이, \\
-a 와 --all 이 있습니다. all이 옵션인데, 이를 약어로 줄일 수 있으며, \\
약어는 -를 하나만 붙인다는 것을 알 수 있습니다. \\
또한, 약어의 경우는 붙여서도 사용 할 수 있습니다. 
```bash
  ls -al
```
여기에서 -al은 하나의 옵션이 아닌. -a 옵션과 -l 두 개의 옵션입니다.

#### Command Line 표시
관리자 계정일 경우, 커맨드라인의 맨 끝은 #으로 표시되고, \\
그렇지 않을 경우에는 $로 표시됩니다. \\
블로그 내에서도 명령어 앞에 이를 붙여서 관리자 계정인지 아닌지 \\
표시 할 예정입니다. 없다면, 상관없다는 이야기지요.