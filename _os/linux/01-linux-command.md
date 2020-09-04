---
title: "linux 계열 명령어 정리"
permalink: /os/linux/linux-command/
excerpt: "linux command list"
last_modified_at: 2019-07-18T15:53:52-04:00
redirect_from:
  - /theme-setup/
toc: true
---

### 공통
---
#### 종료 & 재부팅 
  종료 및 재부팅은 poweroff, halt, reboot 등의 명령어가 있지만, \\
  그냥 shutdown 하나만 알고 계셔도 됩니다. 옵션으로 모두 할 수 있거든요
```bash
// 지금 바로 종료하고, 컴퓨터를 종료한다. now 대신 0을 해도 됩니다.
shutdown -h now
// 10분 후 종료.
shutdown -h +10
// 지금 바로 종료 후 재부팅한다.
shutdown -r now
```

\\ <!-- https://btyy.tistory.com/48 -->

#### 압축 & 압축풀기
##### 압축
  압축은 gzip과 b... 가 있습니다.

##### 압축해제
   참고로 tar의 경우, 파일명으로 폴더를 생성 후, 해당 폴더 안에 압축을 해제 해 줍니다. \\
   많이 사용하는 옵션들입니다. \\
   이 정보들은 tar --help 로 확인하실 수 있습니다. \\
   x : -x, --extract, --get   >> extract files from an archive : 압축파일로부터 압축을 해제합니다. \\
   v : -v, --verbose   >> verbosely list files processed : 진행되는 상황을 화면에 표시합니다 \\
   z : -z, --gzip, --gunzip, --ungzip   >> filter the archive through gzip : gzip으로 압축해제합니다.  \\
   f : -f, --file=ARCHIVE   >> use archive file or device ARCHIVE

```bash
  tar -xvzf apache-tomcat-8.0.52.tar.gz
  mv apache-tomcat-8.0.53 /usr/local
  cd /usr/local/
  ln -s apache-tomcat-8.0.53 tomcat
```
  

#### 삭제
삭제 명령어로는 rm을 사용합니다. \\
가장 많이 사용하는 옵션은 -rf로서, \\
-r 옵션 은 하위디렉토리, 폴더 내 파일이 있을 경우, 같이 삭제합니다. \\
( 옵션을 안 써준다면, 하위 디렉토리나 파일이 있으면 삭제가 되지 않습니다. ) \\
-f 옵션은 삭제 시 확인 과정을 거치지 않습니다.  \\
관리자 계정으로 로그인해서 rm -rf / 를 입력하는 실수는 하지 않도록 주의합니다.  \\
( 이렇게 하면 모든 데이터가 삭제됩니다. 주의, 또 주의!!  \\
  모든 데이터에는 OS도 포함됩니다. )

```bash
  rm -rf <삭제를 원하는 폴더, 또는 파일의 경로>
```

#### 이동, 파일 & 폴더명 변경
파일, 폴더 이동과 파일명, 폴더명 변경은 동일하게 mv를 사용합니다. \\
아니! 나는 파일명을 변경하려는데, 폴더 이동이 되는건 아냐? \\
이런 생각을 했기에 테스트를 해 봤는데, 해당 위치에 폴더가 있다면 \\
폴더로 처리, 아니면 파일로 처리. 이렇게 되어 있는 듯합니다. 
```bash
  mv apache-tomcat-8.0.53 /usr/local
```
위의 예는 현재 폴더 하위에 있는 apache-tomcat-8.0.53 폴더를 \\
/usr/local(루트 밑)  폴더 하위로 이동이란 의미가 됩니다. \\
/usr/local 폴더가 있거든요~ 

#### 버젼 확인 
```bash
  cat /etc/*-release
  grep . /etc/*-release
```

#### 현재 계정 확인
시스템에 설정된 모든 계정 리스트가 리스트됩니다.
```bash
  cat /etc/passwd
```

#### 프로그램 설치
##### yum 
yum은 프로그램 설치를 하는 것이므로, 관리자의 권한이 있어야 합니다. \\
관리자 계정 또는 sudo 를 앞에 붙여서 실행하도록 합니다. 

  설치 가능 리스트 확인 
  ```bash
    $ sudo yum list | grep {확인하고자 하는 프로그램명}
  ```
  설치된 프로그램 확인
  ```bash
    $ sudo yum list installed | grep {확인하고자 하는 프로그램명}
  ``` 

##### dnf 
dnf는 centos 8 버전부터 지원되는 것으로, yum 대신 사용하라고 합니다. \\
사용법은 yum과 거의 동일한듯합니다.


#### 파일 위치 확인 
  파일 위치 확인
  ```bash
    which {파일명}
  ```
  링크 일 경우, 해당 파일 위치 확인
  ```bash
    readlink {링크명}
  ```


#### 방화벽 확인 및 관리
##### iptables 

##### firewall-cmd
centos7 이상에서는 iptables를 쓰지 않고, \\
firewall-cmd란 명령어를 사용합니다. \\
  포트 확인
  ```bash
    # firewall-cmd --list-all
  ```
  포트 추가 
  ```bash
    # firewall-cmd --permanent --zone=public --add-port=8088/tcp
    // 추가 한 내용을 바로 적용하고 싶다면 reload 해 줘야 한다.
    # firewall-cmd --reload
  ```
  참조 URL [^1]


### dmesg 
network message 내용 출력

1. 




2. 유선 또는 무선 인터넷 사용 \\

```bash
  ifconfig <interface명> up     // 인터페이스를 작동시킵니다.
  또는,
  ip link set dev <interface명> up      // 인터페이스를 작동시킵니다.  중간의 dev는 어떤 역할인지 확인 필요

  ifconfig <interface명>  // 해당 interface명에 대한 정보만 표시.

  centos 7.x 대. 최소 설치 시 ifconfig는 되지 않습니다. 대신 if 명령어를 사용하면 됩니다.
  물론, 사용 할 수도 있습니다. net-tools를 설치하면 됩니다.

  // TODO
  버젼의 차이인지, 최소 설치 때문인지는 확인 해 봐야 합니다.
```

3. sed
https://linuxstory1.tistory.com/entry/SED-%EB%AA%85%EB%A0%B9%EC%96%B4-%EC%82%AC%EC%9A%A9%EB%B2%95



https://blog.work6.kr/60
https://withjeon.com/2018/03/09/wifi-in-linux/
https://wikidocs.net/3200

그냥 바로 무선 설정을 하고 싶지만, wpa_supplicant 명령어가 있어야 하는데, 
centos에 이것이 없다. 이를 설치를 해야 무선을 쓸 수 있고, 그러려면 일단 유선을... ;;;





[^1]: [zetawiki](https://zetawiki.com/wiki/%EB%A6%AC%EB%88%85%EC%8A%A4_%ED%8F%AC%ED%8A%B8_%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4_%EB%AA%A9%EB%A1%9D_%ED%99%95%EC%9D%B8)




-- sed
https://linuxstory1.tistory.com/entry/SED-%EB%AA%85%EB%A0%B9%EC%96%B4-%EC%82%AC%EC%9A%A9%EB%B2%95


: --help 보는 방법.
: 기본 디렉토리 구조.



/* branch는 master에서 시작

git remote add upstream https://github.com/mmistakes/minimal-mistakes.git

git fetch upstream

git merge upstream/master

*/



:: 랜카드 기동.
ip addr
ifup 랜카드명


:: 기동 설정 유지
https://devhoma.tistory.com/101

::: 디렉토리 구조
https://webdir.tistory.com/101?category=561456 


:: bash 한글 표시
https://byeonely.tistory.com/8 


:: bash 형식 생성
http://bashrcgenerator.com/


:: 폴더 표시 색상 변경
https://www.lesstif.com/pages/viewpage.action?pageId=24445156 

:: root vim 
http://home.zany.kr:9003/board/bView.asp?bCode=11&aCode=3377



:: vim 환경변수
http://kwonnam.pe.kr/wiki/vim#%ED%99%98%EA%B2%BD%EB%B3%80%EC%88%98_%ED%99%95%EC%9D%B8  

:: 유저그룹
https://btyy.tistory.com/35


cryptogen generate --config=./crypto-config.yaml








1.4.1 ==> 맨 마지막 버전이 홀수 인 것. : unstable































운영체제 정보 관련 : http://blog.naver.com/PostView.nhn?blogId=choigohot&logNo=40193643022 