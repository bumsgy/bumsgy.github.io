---
title: "intellij란"
permalink: /tools/intellij/intellij-setting
excerpt: "intellij"
tags: ["tools", "intellij", "setting"]
date: 2019-03-05 10:30:00 +0900
last_modified_at: 2020-01-24T17:00:00+09:00
redirect_from:
  - /theme-setup/
toc: true
toc_label: "2020.01.16"
description : intellij란?
---

### intellij 설치 후 최초 셋팅
Intellij 설치 시 최초 설정해야 할 부분들에 대한 내용입니다. \\
물론, 필수는 아니겠지만, 설정을 하지 않을 경우, 오류가 발생할 경우가 많다거나 \\
좀더 편하기 사용하기 위한 설정들입니다. 

1. 먼저 CharacterSet 설정이 필요합니다. \\
  최근에는 UTF-8을 대부분 사용합니다. \\
  File > setting 또는 Ctrl + Alt + S를 눌러, Setting 메뉴를 엽니다. \\
  ![setting 화면](/assets/postImg/1.png) 
  + 검색으로 encoding 또는 Editor > File Encoding에 들어갑니다.
  + Global Encoding 및 Project Encoding을 UTF-8로 설정합니다.
  + 이 때, Transparent native-to-ascii conversion을 체크 해 줍니다. \\
    UTF-8로 작성한 properties 파일 내에 한글이 있을 경우, \\
    한글은 아스키코드로 보여지게 되는데, 체크를 하게되면, \\
    properties 파일 내의 내요이 한글로 제대로 나오게 됩니다.
  + 또한, UTF-8 파일 생성 시 BOM은 없도록 합니다. (마지막 항목) \\
    ((BOM 이미지 링크))
2. ignore 파일 설정
3. plug-in 설치

구입을 했을 경우, 설정 정보는 intellij 서버에 저장되어 \\
타PC에 설치 시 자동으로 설치됩니다.





