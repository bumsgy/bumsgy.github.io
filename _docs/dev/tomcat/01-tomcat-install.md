---
title: VS Code. git 셋팅하기
date: 2019-05-09
tags: ["Dev", "tools", "Setting"]
category : ["Dev", "vscode"]
description : vscode에 git 셋팅하는 방법
---

1. git 설치 \\
   git이 설치되어 있지 않다면 git을 먼저 설치

2. vs code 실행 \\
   File > Preferences > Settings... 선택 \\ 
   ( 또는 Ctl + , 단축키를 눌러도 됩니다. ) \\
   선택 시 Settings 파일이 열리게 됩니다. 

3. Settings 파일 내용 중 Extension > git 을 선택합니다.  \\
   설정 내용 중 Path 항목이 있습니다. \\ 
   만약, git을 기본 설정 그대로 설치 안 했다면, 경로가 다르겠죠~ \\ 
   그럴 경우, 밑에 있는 'edit in setting.json' 버튼을 클릭하여,\\
   설정 파일을 엽니다. \\
   해당 내용 안에 "git.path": git경로 \\
   이 때, 경로는 "로 감싸고, 윈도우일 경우에는 \ 를 2번씩 써 주세요. \\ 
   이스케이프 문자이기 때문에~~( 다 아실테지만 혹시나~ )
