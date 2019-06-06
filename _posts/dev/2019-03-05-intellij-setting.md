---
title: intellij 초기 셋팅
date: 2019-03-05
tags: ["Dev", "intellij", "Setting"]
category : "intellij"
description : intellij 초기 셋팅
---

## intellij 설치 후 최초 셋팅
Intellij 설치 시 최초 설정해야 할 부분들에 대한 내용입니다. 
최초 설정이 필요합니다.

1. 먼저, 캐릭터셋 설정이 필요합니다.  
   최근에는 UTF-8을 대부분 사용합니다.  
   FILE > settings 또는 Ctrl + Alt + S 를 눌러, Setting 메뉴를 엽니다.  
   ![setting 화면](/assets/postImg/1.PNG)
   + 검색으로 encoding 또는 Editor > File Encoding 에 들어갑니다.  
   + Global Encoding 및 Project Encoding을 UTF-8로 설정합니다.  
   + 하단에 properties files들에 대해서도 UTF-8로 설정합니다.  
   + 이 때, Transparent native-to-ascii conversion을 체크 해 줍니다.  
     UTF-8로 작성한 properties 파일 내에 한글이 있을 경우, 한글은 아스키코드로 보여지게 되는데,  
     체크를 하게 되면, properties 파일 내의 내용이 한글로 제대로 나오게 됩니다.  
   + 또한, UTF-8 파일 생성 시 BOM은 없도록 합니다.( 마지막 항목 )  
     (( BOM 링크..는 나중에 ))  

* * * * * * * * * * * * * * * * * * * * * * * * * * * *

2. ignore 파일 설정

3. plug-in 설치
 - 
 - 
구입을 했을 경우, 설정 정보는 intellij에 저장되어 
타 PC에 설치 시 바로 공유가 됩니다. 




<!-- more -->

## 여기가 상세 부분?


``` 
여기 밑이 코드...
뭐든지 상관없나? 

```