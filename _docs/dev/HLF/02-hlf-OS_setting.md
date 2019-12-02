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

   최소 설치의 경우, 이더넷이 활성화되어 있지 않습니다. 이를 활성화 해 줘야 합니다.

   ``` bash 
   // 인터페이스명 확인
   ip addr 

   // 인터페이스 활성화
   ifconfig <interface명> up
   ```

### 진행 전 Check 
먼저 로그인은 root가 아닌, 같이 생성한 계정으로 진행합니다. 저는 Hyperledger fabric을 할 거니, fabric으로 만들었습니다.

쉘은 명령어 앞에 #, $도 같이 표시 할 예정입니다. 다들 아시겠지만, #은 root, $는 이외의 계정입니다. 

### shell 설정
쉘 설정을 해 줍니다.

현재 경로가 모두 나오면 오히려 보기 어렵다고 하시는 분들도 계시지만, hyperledger는 경로 depth도 많을 뿐더러, 비슷한 명칭, 긴 directory명 때문에 저는 모두 표시하도록 했습니다.

   ```bash
      $ vi ~/.bashrc
   ```

안의 내용.

   ```vim
      ################################
      ### .bashrc ####################
      ################################

      # User specific aliases and functions

      alias rm='rm -i'
      alias cp='cp -i'
      alias mv='mv -i'
      alias ll='ls -al'
      alias vi='/usr/bin/vim'

      alias dcrm='docker rm -f $(docker ps -aq)'
      alias dcrmi='docker rmi -f $(docker images)'

      # Source global definitions
      if [ -f /etc/bashrc ]; then
          . /etc/bashrc
      fi

      LANG=ko_KR
      export LANG

      export PS1="\[\033[38;5;11m\]\u\[$(tput sgr0)\]\[\033[38;5;15m\]@\h:\[$(tput sgr0)\]\[\033[38;5;6m\]\w:\[$(tput sgr0)\]\[\033[38;5;15m\] \n\\$ \[$(tput sgr0)\]"

      export PATH=$PATH:"/usr/local/go/bin"
      PATH=$PATH:"/home/fabric/go/src/github.com/ldt-hf/tools/network/bin"

      export NVM_DIR="$HOME/.nvm"
      [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
      [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

      GOPATH="/home/fabric/go"
      GOROOT="/usr/local/go"

      export GOPATH
      export GOROOT
   ```




### Yum

설치는 yum을 이용하여 설치를 진행 할 예정입니다.

1. 먼저 저장소를 업데이트합니다. yum은 root로 진행해야 하기 때문에 명령어 앞에 sudo를 붙입니다.

   ```bash
     $ sudo yum update
   ```

### VI 설치
이제 인터넷이 연결되었으니, 이제 셋팅 작업을 해야겠죠? 셋팅을 하려면 일단 에디터가 필요할 듯합니다.

1. vim을 설치합니다.

   ```bash
      $  sudo yum install vim
   ```

### VI 설정

vim을 좀 더 편하게 쓰기 위해 설정을 합니다.

1. bash의 

### nodejs 설치

fabric은 client를 nodejs를 이용하기 때문에 nodejs를 설치해야 합니다.

   ```bash
   // 의존성 패키지 설치
   # yum install -y gcc-c++ make     

   // nodejs 최신버전 저장소 설치
   // centos에 저장소에 있는 버젼은 아주 낮기 때문에 nodejs 저장소를 직접 추가 해 줍니다.
   // bash 뒤에 - 는 오타 아닙니다. 
   # curl -sL https://rpm.nodesource.com/setup_8.x | bash -

   // 추가 된 저장소를 이용하여 nodejs를 설치합니다.
   # yum install nodejs
   ```

### NVM 설치

NVM은 간단하게 말하면, nodejs 버젼 관리 프로그램입니다. 

nodejs가 버전에 따라 오류가 많이 발생하기 합니다. 그렇다고, 버전이 다르다고 매번 nodejs를 삭제하고 다시 설치하고 할 수는 없으니, 자신이 사용할 버전을 선택 할 수 있게 해 줍니다.

설치 후 반영이 바로 되지 않기 때문에, 로그아웃 후에 다시 로그인 해 주세요. 

   ```bash
      $ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash

      $ nvm --version
   ```

### golang 설치

fabric chaincode는 go언어로 작성됩니다. 물론, java, nodejs 등으로도 가능하지만, 현재까지 공식 지원언어는 golang이며, 그렇기에 golang을 설치합니다.

1. 저장소를 설치합니다.

   golang 설치를 위해, epel-release 저장소를 설치하고 있으나, 버젼이 낮습니다. 일단은 이렇게 진행했지만, 버전이 낮아 안 되는 경우가 있다면, 있다면..... 다시 설치해야겠죠? ;;;

   ```bash
   // 저장소를 먼저 설치합니다.
   $ sudo yum install epel-release
   ```

2. golang을 설치합니다.

   golang 설치 시 권한으로 인하여, sudo를 써야 하는데, sudo를 쓰게 되면, 경로가 fabric의 home 하위가 아니게 됩니다. 그렇기 때문에 sudo 뒤에 -s를 붙여서 fabric의 home 밑에 설치되도록 합니다.

   ```bash
   $ sudo -s yum install golang
   ```

3. 경로 정보를 등록 해 줍니다.

   경로를 추가했다면, 현재 계정을 로그아웃했다가 다시 로그인해야 적용이 됩니다. 또는, source 명령어를 이용하는 방법도 있습니다.
   경로는 go lang 환경변수를 참조하여 설정합니다.

   ```bash
   $ go env
   
   $ vi .bashrc
   
   // 내용
   export GOPATH="/home/fabric/go"
   export GOROOT="/usr/lib/golang"

   // 로그아웃 후 로그인

   // 또는 설정 refresh
   $ source ~/.bashrc
   ```

### docker 설치
docker 또한 epel 저장소 등을 이용하게 되면 낮은 버전의 docker가 설치되기 때문에, docker 공식 홈페이지를 보고 진행합니다.

   ```bash
      $ sudo yum install -y yum-utils device-mapper-persistent-data lvm2

      $ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

      $ yum repolist

      $ sudo yum install docker-ce docker-ce-cli containerd.io

      $ sudo systemctl start docker

      $ sudo systemctl enable docker

      $ sudo yum install docker-compose

      $ sudo usermod -aG docker $USER
   ```





tar -xvzf go1.12.7.linux-amd64.tar.gz
  mv go /usr/local
  cd /usr/local/
  ln -s apache-tomcat-8.0.53 tomcat







# 권한
해당 폴더에 접근하고 싶다면? 
cd 폴더명을 쓸텐데, r 이면 될 듯하지만..
x 권한이 있어야 함.
해당 폴더에서 ls를 하려면 r 이 필요한것.









# root
# sudo 권한을 부여 해 주기 위함.
chmod u+w /etc/sudoers

# 모든 유저에서 권한 부여
%wheel        ALL=(ALL)       NOPASSWD: ALL

:::: 이거 sudo 안 됨. 어디서 된다는거지? ㅡㅡ;;; 계속 sudo 요구.



yo 설치
:

fabric@localhost:~/go/src/github.com/ldt-hf: 
$ yo express-no-stress ldt-api
? App description [My cool app] hlf client for ldt
? API Root [/api/v1] 
? Version [1.0.0] 
? OpenAPI spec version Swagger 2
? Linter Airbnb

이렇게 선택 설치.


API를 만들어야 하는데....


유저 등록.
유저마다의 매일 거래 정보 등록
유저마다의 거래 정보 조회. 이렇게 하면 되나?
