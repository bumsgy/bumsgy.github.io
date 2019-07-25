---
title: "blockchain"
permalink: /docs/quick-start-guide/
excerpt: " "
last_modified_at: 2019-07-17T13:00:00+09:00
redirect_from:
  - /theme-setup/
toc: true
---
2012. 바람의 검심
아오하라이드



> yum list | grep nodejs

> yum install epel-release

// 확인
> yum repolist 

> yum update ... 를 했어야 했는데...

> yum install nodejs

> node -v 
 : 확인 해 보니... 0.x 버젼? 경악!!
 : 이리 설치하면 안 된다고 하네.. ㅡㅡ

> yum remove nodejs

> yum list installed | grep nodejs

참조 : https://github.com/nodesource/distributions

이라는데, 잠깐... 다른 블로그 다른 방식을 먼저.

epel 저장소 추가 한 것부터.
> yum install npm nodejs

> yum update openssl

> npm install -g {{version}}
 : version은 특정버전을 직접 적거나, stable(안정화버전), latest(최신버전)

npm에서 에러... 나중에 다시 해 보는 걸로.
 https://m.blog.naver.com/PostView.nhn?blogId=gura2013&logNo=221334232514&proxyReferer=https%3A%2F%2Fwww.google.com%2F


 remove에서 다시 시작
> yum install -y gcc-c++ make     #의존성 패키지 설치

> curl -sL https://rpm.nodesource.com/setup_8.x | bash -   # nodejs 최신버전 저장소 설치

> yum install nodejs


### nvm 설치
https://github.com/nvm-sh/nvm

> curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash

> source ~/.bashrc

### gulp 설치
https://gulpjs.com/docs/en/getting-started/quick-start

// global로 설치
> npm install -g gulp


### golang 설치
https://golang.org/dl/

> yum install golang

> go env
GOPATH="/root/go"
GOROOT="/usr/lib/golang"

2개 확인

> vi ~/.bashrc
추가
export GOPATH="/root/go"
export GOROOT="/usr/lib/golang"

reload
> source ~/.bashrc

확인
> env | grep GO


### docker 설치
찾아도 잘 안 나옴.
일단 repo 표시되어서 밀어봄! ( epel )
> yum install docker 










### centos 6 여서 다시 7 설치
최소 설치.




인터넷 활성화
> ifup {랜카드ID}
https://devhoma.tistory.com/101


: epel 정보
http://faq.hostway.co.kr/Linux_ETC/7095


> yum update

> yum install epel-release
> yum list | grep nodejs
: 여전히 nodejs 버젼은... 1.x 대... ㄷㄷㄷ

> yum install -y gcc-c++ make
> curl -sL https://rpm.nodesource.com/setup_8.x | bash -

> yum repolist 
: repo로 추가된 것이 확인

> yum install nodejs

> node -v
> npm -v

: nvm 설치
> curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash

> yum install golang

이하 env 등록 다시.


> yum install docker 

> systemctl status docker
> systemctl enable docker


> yum install docker-compose
: docker-compose는 서비스가 아니네? ... 아닌게 맞...지? 흠흐..

> go get -u github.com/hyperledger/fabric-sdk-node

> go get -u github.com/hyperledger/fabric-samples

> go get -u github.com/hyperledger/fabric




> firewall-cmd --zone=public --add-port=5984/tcp --permanent
firewall-cmd --zone=public --add-port=7050/tcp --permanent
firewall-cmd --zone=public --add-port=7051/tcp --permanent
firewall-cmd --zone=public --add-port=7054/tcp --permanent
firewall-cmd --zone=public --add-port=8051/tcp --permanent

firewall-cmd --zone=public --add-port=4000/tcp --permanent



cd ~~~~~/hyperledger/fabric 
make native
: 해 주면 .build 등의 폴더가 생성. --> 이 아니네?
: 맞아!! 맞아!!! ㅋㅋ







/// 
secureCrt

