---
title: 정리 해야 할 글이 많구나...
date: 2019-06-13
last_modified_at: 2019-06-13T13:00:00+09:00
tags: ["todo"]
category : ["lifelog", "2019", "06"]
description : 휴.... 해야 할 것이 너무 많구나... 
toc: true
toc_label: "정리 해야 할 글들"
toc_icon: "cog"  # corresponding Font Awesome icon name (without fa prefix)
---

### 넋두리...
매번 해야지 해야지 하면서도 하지 못 했던 블로그 글 정리. \\
큰 맘 먹고 이번에 해 보려고 하는데 \\
손을 대면 댈수록 모르는 것도 많고, 더 확인 해 봐야 하는 것도 많고, \\
정리해야 할 것들도 많고... \\
오히려 너무 많으니, 선뜻 손이 안 가는... 이런 악순환이 되네요. \\
이럴 땐 일단 무조건 하나씩!!! 해 나가야 하는게 맞겠죠? \\
해 봄세~ 아자아자~~!!! 

### 작성해야 할 글들.
1. yum 을 이용한 설치인데, \\
   인터넷이 안 되는 것을 상정. 또는 yum list에 없는 것으로 생각해서 \\
   파일을 먼저 다운로드 한 후, 인터넷을 통한 것이 아닌 해당 파일을 이용해서 설치하는 방법 

2. windows, linux 모두 환경 변수 설정 방법

3. tomcat 환경 설정
   : appbase, docbase 
   : jndi
   : 하나의 tomcat에 여러 개의 인스턴스


4. javascript
   : 배열에 추가하기 전에 값이 먼저 있는지 확인( 중복 체크 )
     --> indexOf를 쓰기도 하지만, 중복 체크 key가 여러개인 경우에는? (( 문자열 합치기를 하거나 하는 방법도 있지만... 그냥은 안 됨 ??? ))

5. vue..
   : v-for:key로 등록되어 있다면, 알아서 해당 배열에 중복 건은 제거 해 준다.

   : data object에 push를 할 때 중복체크를 해 주는가? 해 준다면 어느 기준으로? 
   : array의 push는 swallow(얕은) 복사인가? 



const a = [
     {"akey":"aa1",
      "bkey":"bb1",
      "ckey":"cc1",
     },
     {"akey":"aa2",
      "bkey":"bb2",
      "ckey":"cc2",
     },
     {"akey":"aa3",
      "bkey":"bb3",
      "ckey":"cc3",
     },
];
let b= [];

a.forEach(el => {
     b.push(el);
});

b[2].akey="bbbb";

// 데이터는 중복에 처리는 안 되겠지.
// 중복 시 v-for가.. key로 사용하는 값이 2개가 되어 버리니, 거기에 에러를 뱉어내는 것.