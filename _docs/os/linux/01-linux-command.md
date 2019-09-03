---
title: "linux 계열 명령어 정리"
permalink: /docs/os/linux/linux-command/
excerpt: "linux command list"
last_modified_at: 2019-07-18T15:53:52-04:00
redirect_from:
  - /theme-setup/
toc: true
---

### linux

1. 종류 \\
   리눅스는 여러 종류의 리눅스가 있습니다. \\
   리눅스 자체가 opensource이다 보니, 여러 기업에서 이를 이용하여, 자신들의 리눅스를 만들어 제공한 거죠. \\
   Ubuntu, Fedora, Centos 등등이 있습니다.

2. 계열 \\
   또한, 계열이 있는데, 이를 알고 있어야 편합니다. \\
   몇몇 명령어가 다르기 때문이죠. \\
   크게 Debian 계열, RedHat 계열로 구분합니다. \\
   Debian 계열에는 대표적으로 Ubuntu가 있습니다. \\
   RedHat 계열에는 Centos, Fedora 등이 있습니다.

   debian 계열에서는 설치 시 apt-get을 사용하고, \\
   RedHat 계열에서는 rpm(RedHat Package Manager) 또는 yum(Yellowdog Update Modified)을 사용합니다. \\
   둘 다 패키지 관리자로서는 동일하지만, \\
   rpm은 설치 시 의존성 문제가 발생 할 수 있으며, 이를 해결 한 것이 yum입니다. 

   만약, 이를 모르면... 저처럼 멘붕에 빠질 수 있습니다. \\
   ( A 프로그램을 설치해야해서 검색을 해 보니.. yum으로 한다고? 그런데 yum이 없네? \\
     yum 설치를 검색 해 보니, apt-get을 사용하라고? 그런데 apt-get이 없네? \\
     apt-get 설치를 검색 해 보니, yum으로 설치하라고?... 응?응? )
   

3. 파일 또는 폴더의 경로 입력 시, 맨 앞에 /를 입력한다면 root부터의 경로로 인식합니다. \\
   /부터 시작하지 않으면, 현재 위치부터 상대 경로로 인식합니다.


### 공통

1. 종료 \\
    https://btyy.tistory.com/48

    ```
    shutdown -h now     // 지금 바로 종료하고, 컴퓨터를 종료한다. now 대신 0을 해도 됩니다.
    shutdown -r now     // 지금 바로 종료 후 재부팅한다.
    ```

2. 압축 \\
  압축은 gzip과 b... 가 있습니다.

3. 압축해제 \\
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
  참조 blog[^1]

4. 삭제 \\
  삭제 명령어로는 rm을 사용합니다. \\
  가장 많이 사용하는 옵션은 -rf로서, \\
  -r 옵션 은 하위디렉토리, 폴더 내 파일이 있을 경우, 같이 삭제합니다. \\
  ( 옵션을 안 써준다면, 하위 디렉토리나 파일이 있으면 삭제가 되지 않습니다. ) \\
  -f 옵션은 삭제 시 확인 과정을 거치지 않습니다.  \\
  관리자 계정으로 로그인해서 rm -rf / 를 입력하는 실수는 하지 않도록 주의합니다.  \\
  ( 이렇게 하면 모든 데이터가 삭제됩니다. 주의, 또 주의!! )

    ```bash
      rm -rf <삭제를 원하는 폴더, 또는 파일의 경로>
    ```

5. 이동, 파일 & 폴더명 변경


6. 버젼 확인 
    ```bash
      cat /etc/*-release
      grep . /etc/*-release
    ```

7. 현재 계정 확인
    ```bash
      cat /etc/passwd
    ```

8. 설치된 것 확인
sudo yum list installed


### centos

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





[^1]: sample.




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

































###############
# 여기부터가 

## fabric 계정을 설치 시 같이 생성.
## admin 권한 소유.

# fabric 
sudo yum update

sudo yum install vim

# root
# sudo 권한을 부여 해 주기 위함.
chmod u+w /etc/sudoers

# 모든 유저에서 권한 부여
%wheel        ALL=(ALL)       NOPASSWD: ALL

:::: 이거 sudo 안 됨. 어디서 된다는거지? ㅡㅡ;;; 계속 sudo 요구.


yum install epel-release

sudo -s yum install golang


vi .bashrc
GOPATH="/home/fabric/go"
GOROOT="/usr/lib/golang"

source ~/.bashrc

sudo yum install -y gcc-c++ make


# root 권한에서 ... 이거는 fabric 에서 안 되네? sudo 해도 안 됨.
curl -sL https://rpm.nodesource.com/setup_8.x | bash -

sudo yum install nodejs

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash

nvm은 로그아웃 후 다시 들어올 것.

sudo yum install docker 

sudo systemctl start docker
sudo systemctl enable docker

sudo yum install docker-compose

newgrp - docker // 이걸로 어찌 되는 건지는 좀 더 확인해 봐야 할 듯.




tar -xvzf go1.12.7.linux-amd64.tar.gz
  mv go /usr/local
  cd /usr/local/
  ln -s apache-tomcat-8.0.53 tomcat







# 권한
해당 폴더에 접근하고 싶다면? 
cd 폴더명을 쓸텐데, r 이면 될 듯하지만..
x 권한이 있어야 함.
해당 폴더에서 ls를 하려면 r 이 필요한것.
