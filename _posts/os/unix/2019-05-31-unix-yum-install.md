---
title: yum 설치
date: 2019-05-31
tags: ["Dev", "centos"]
category : ["Dev", "unix"]
description : "yum 설치 방법"
---

## Unix 계열에 yum이 설치되어 있지 않을 때, 설치 방법
Unix 계열에 yum이 설치되어 있지 않을 때, 이를 설치하는 방법에 대해 설명합니다.
일단, yum이란 무엇인가? --> 링크   는 한 번 정도 읽어보는 것이 좋을 듯 하네요.

yum 홈페이지 : [http://yum.baseurl.org/](http://yum.baseurl.org/)

* * * * * * * * * * * * * * * * * * * * * * * * * * * *

{% include toc.html %}
# 링크복사
1. 위의 페이지에 가서 다운로드 링크를 복사합니다.

# 다운로드
2. wget을 이용하여 다운로드 합니다.
```
   wget http://yum.baseurl.org/download/3.4/yum-3.4.3.tar.gz
```
   -. [wget이란?]()

# 압축해제
2. 확장자가 tar.gz이므로, tar 을 이용하여 압축을 해제합니다. 
   ex) 제가 받은 버전은 3.4.3으로서 파일명이  yum-3.4.3.tar.gz 이었습니다.
```
   tar -xvzf yum-3.4.3.tar.gz
```
    압축을 해제하면 폴더가 생성되므로, 해당 폴더로 이동 해 봅니다.
```
    cd yum-3.4.3
```
# install
3. ./configure 가서 make install을 하라고.. 하던데...
    해당 폴더가 없.... 일단 여기서 stop

