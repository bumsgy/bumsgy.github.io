---
title: Hyperledger Fabric OS setting
permalink: /docs/dev/hlf/hlf-OS_setting/
excerpt: "hyperledger fabric"
last_modified_at: 2019-08-30T17:00:00+09:00
redirect_from:
  - /theme-setup/
toc: true
---

### CentOS 7.x 설치.

뿅뿅뿅~ USB에 넣어서 설치~ \\
최소 설치 \\
나중에 유저를 생성 해 줘도 무방하지만, 저는 관리자 계정 이외, Hyperledger Fabric 계졍을 같이 생성 해 주었습니다. \\
조금이나마, 편하게 하고 싶어, 관리자 권한도 부여했습니다. \\
(( 그냥 유저 계정일 때, 어떠한 차이점이 있는지는 차후에 한 번 해 봐야겠네요. ))

### IP Setting 

최소 설치의 경우, 이더넷이 활성화되어 있지 않습니다. 
이를 활성화 해 줘야 합니다.

  ``` bash 
      // 인터페이스명 확인
      ip addr 

      // 인터페이스 활성화
      ifconfig <interface명> up
  ```

이제 인터넷이 연결되었으니, 이제 셋팅 작업을 해야겠죠?
셋팅을 하려면 일단 에디터가 필요할 듯합니다.


### 진행 전 Check 
먼저 로그인은 root가 아닌, 같이 생성한 계정으로 진행합니다. \\ 
저는 Hyperledger fabric을 할 거니, fabric으로 만들었습니다.

쉘은 명령어 앞에 #, $도 같이 표시 할 예정입니다. \\ 
다들 아시겠지만, #은 root, $는 이외의 계정입니다. 

### Yum 
설치는 yum을 이용하여 설치를 진행 할 예정입니다. 
1. 먼저 저장소를 업데이트합니다. \\ 
   yum은 root로 진행해야 하기 때문에 명령어 앞에 sudo를 붙입니다.
  ```bash
    $ sudo yum update
  ```

### VI 설치
1. vim을 설치합니다.
  ```bash
    $  sudo yum install vim
  ```










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
