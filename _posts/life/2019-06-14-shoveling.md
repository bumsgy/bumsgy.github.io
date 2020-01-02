---
title: 오늘의 삽질일기
date: 2019-06-13
last_modified_at: 2019-06-13T13:00:00+09:00
tags: ["diary", "docker", "dev"]
category : ["lifelog", "2019", "06"]
description : 맨날 삽질이야... ㅡㅡ
toc: true
toc_label: "정리 해야 할 글들"
toc_icon: "cog"  # corresponding Font Awesome icon name (without fa prefix)
---

### 오늘의 삽질 시작.

intellij에서 SVN 연결은 한 번도 안 해 봤더니... 후우... \\
일단 plugin으로 있나? 확인 해 봤더니... 없네? \\
그래서 settings에서 봤더니. svn.exe 파일 설정 화면이 있네... \\
하려 하니.. svn이 없네? \\
설치를 뭐 해야 하지? \\
visualSVN은... 서버에 설치하는 녀석이고... \\
tortoiseSVN을 설치하자.... 해서 했...더...니!!!!! \\
svn.exe 자체가 없다... \\
탐색기에서 context menu(우측 마우스 클릭) 으로 사용하거나,  \\
자기들 툴을 이용해서 하게 되어있네... \\
그래서... tortoiseSVN를 cli로 이용하는 방법이 있나, \\
찾고... 또 찾고.. 해 봤지만... 안 됨.. \\
결국, silkySVN 설치로 해결... \\
후... 그녕 silkySVN 설치할 걸.... ㅠㅠㅠ 

### 그리고 멘붕.
집에 와서 올만에 토이플젝을 하려 했습니다. \\
DB를 docker container로 사용하고 있었는데... \\
docker 에러.... OTL \\
찾아보니, 비슷한 경험을 하신 분들이 계시더군요. \\
https://yoonbh2714.blogspot.com/2017/12/windows10-docker-start-error_35.html \\
docker 자체가 기동이 안 되는건데...
링크에 블로그 분은 hyper-v를 재시작하라 하셨고, \\
또, 다른 어떤 분은 hyper-v 관리자에 보면, 컴터가 등록되어 있지 않으니 \\
등록하면 된다 하셨는데.. \\
전 등록되어 있고, 재시작해도 안 되더군요. \\
윈도에선 불안불안한듯... \\
언제 백업하는 것도 알아 둬야 할 듯합니다. 휴... \\
결론. 다 날렸음. \\

18일 추가. docker 백업 및 복사 등등을 알아봤는데, \\
결론은 docker 명령어를 이용해야 하기 때문에, docker가 되어야 하네요. \\
문제는 docker 자체가 안 되고 있어서.... 정말 포기... 아흑.. 내 DB. 내 데이터.... 