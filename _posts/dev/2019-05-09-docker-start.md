---
title: docker 시작하기
date: 2019-05-09
tags: ["Dev", "docker", "Setting"]
category : ["Dev", "docker"]
description : docker 시작하기
---

## docker 설치 및 개념 잡기 ( for windows)

1. 다운로드 
   https://docs.docker.com/docker-for-windows/
   다운로드 페이지에 들어가서 docker for windows 를 다운받는다.

2. 설치 실행.
   inst1.jpg
   Use Windows containers instead of Linux containers(this can be changed after installation) : 체크
   (( 윈도우 버젼이니 체크한다...라는 말이 있는데... 이 옵션에 대한 정확한 내용은 무엇인지 확인 필요. ))
   자신의 컴퓨터의 hyper-V 기능이 켜져 있어야 한다. 
   CMOS 셋업에서 셋팅 필요.

3. docker 로그인.
   트레이 아이콘을 이용.

4. CMD 창에서 
   inst2.jpg
   docker ps  입력. --> docker에 등록된 container List를 보여준다. 
   현재는 아무것도 없다.
