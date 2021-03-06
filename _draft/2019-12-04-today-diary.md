---
title: 자아~ 백업받고 RDS를 일단 닫자~
date: 2019-12-03
last_modified_at: 2019-12-02T13:00:00+09:00
tags: ["diary", "glogen", "dev"]
category : ["diary", "glogen", "dev"]
description : 하으... 사람이 안 들어오네...
toc: true
toc_label: ""
toc_icon: "cog"  # corresponding Font Awesome icon name (without fa prefix)
---

### RDS를 없애야 할 듯해요.
설정 공유 및 컴파일 자동화를 위해, gradle을 사용하기로 함   - gradle과 gradle DSL는 다른 것인가?
1.1.1. https://stackoverflow.com/questions/38336576/what-does-dsl-mean-in-gradle
1.1.2. Simply, it stands for 'Domain Specific Language'. IMO, in gradle context, DSL gives you a gradle specific way to form your build scripts. More precisely, it's a plugin-based build system that defines a way of setting up your build script using (mainly) building blocks defined in various plugins.라는 답변을 찾음.
1.1.3. gradle과 gradle DSL이라고 쓰는 것은 다른 것을 뜻함은 아닌 듯함.검색 시 2가지 내용 모두 그대로 사용이 가능할 것으로 판단

   - DSL이란 무엇인가?
1) DSL이란
1.1.3.1. 특정 도메인(산업, 분야등 특정 영역)에 특화된 언어를 말한다. 
1.1.3.2. "문제 영역의 해결에는 그 영역의 언어를 전제로 둬야하며, 거기에서 프로그래밍 솔루션을 꺼내는 것이 중요하다." 라고 Dave Thomas가 한 말을 생각하면 이해하기 쉽다.
1.1.3.3. 특정 영역의 문제 해결에는 그 영역에 맞는 특화된 도구를 사용하자라는 것이다. 어찌보면 과도로 끝내도 될 일을 맥가이버칼을 들이대는 격이다. 그리고 표현 방식은 해당 도메인의 전문가가 이해할 수 있는 형태(고급 언어)여야 한다.
1.1.3.4. 실제로 Ruby처럼 그 코드가 일반 자연어를 읽는 것과 같이 쉽게 이해되기 때문에 도메인 전문가와 프로그래머가 아이디어를 공유하기 좋고 거기에 Lisp 계열이어서 DSL에 많이 활용된다.
1.1.3.5. 그리고 DSL은 개발 생산성과 도메인 전문가와 커뮤니케이션을 원할하게 하기 위해서도입되는 경향이 많지만, No Silver Bullet.
1.1.3.6. 참고로 UNIX에서도 특정한 응용 영역의 문제를 해결하기 위해 그 영역에만 적용할 수 있는 특수한 언어를 만들어 문제를 해결하는 오랜 전통이 존재한다. 이런 언어를 "작은 언어(little language)", 또는 "미니 언어(mini language)"라고 부르는데 DSL도 이와 유사하다고 볼 수 있다.
2) 내부 DSL과 외부 DSL
가) 내부 DSL
1.1.3.6.1. 호스트 언어 구문을 이용하여 자체적으로 의존하는 무언가를 만드는 경우에 해당된다.
1.1.3.6.2. 내부 DSL에서는 API와 DSL의 경계가 모호해 비슷하게 생각하는 경향이 있다. 좀 더 일반 사용자가 알아보기 쉬운 API가 내부 DSL로 생각해도 될 듯 하다.
1.1.3.6.3. 호스트 언어 능력과 지금까지 사용하던 도구를 그대로 사용할 수 있다는 점,처리 결과를 쉽게 예측할 수 있어서 해당 언어를 잘 알면 친근할 수 있다.
1.1.3.6.4. 형태 : 메타 프로그래밍의 형태로 언어에 미니 언어를 만들 수 있다.  : 원래 언어로 새로운 구문으로 도입 된다. 그래서 언어 확장을 일으켜 다른 언어가 된다. : 인라인 코드 형태로 표현될 수도 있다.
1.1.3.6.5. 적합한 언어 : Lisp, Ruby, Smalltalk
나) 외부 DSL
(1) 호스트 언어와 다른 언어 (XML, Makefile과 같은 고유 형식)에서 생성된 DSL.
(2) GUI 도구를 제공해 주는 것이 특징.
(3) 외부 DSL에서는 DSL과 범용 언어(GPL : General Purpose Language)과의 경계가 모호해지는 경향이 있다. : 그 차이는 언어 작성자와 언어 사용자의 목적에 있다. 특정 영역에서 언어의 작성자가   아닌 사용자의 목적에 부합하는, 이해를 할 수 있으면 외부 DSL이다.
(4) 외부 DSL 개발자가 자유롭게 DSL의 형식을 결정할 수 있다.
(5) 형태 : 실행 파일에서 DSL 을 동적 로딩할 수 있다. : DSL 컴파일러를 만들어서 표현할 수 있다. : DSL을 범용 언어로 코드로 변환한다.
(6) 적합한 언어 : Java, C#, C++
2) DSL의 장점과 단점
1.1.3.7. 장점
1.1.3.7.1. 반복이 제거되고 비슷한 처리 코드는 자동 생성(템플릿) 된다.
1.1.3.7.2. 프로그래밍 코드의 양이 적고 가독성이 높다.
1.1.3.7.3. 특정 프로그래머(lay programer - martin fowler)들과 커뮤니케이션이 쉽다. : XML, CSS, SQL 등
1.1.3.8. 단점
1.1.3.8.1. 설계가 어렵다.
1.1.3.8.2. 잘 설계가 되지 않는다면 읽기 어려운 코드가 될 수 있다.
1.1.3.8.3. 하위 호환성을 유지해야 한다.
3) 우리 주변에 있는 DSL
가) java  : ANT, Maven, struts-config.xml, Seasar2 S2DAO,   HQL(Hibernate Query Language),  JMock
가) Ruby : Rails Validations, Rails ActiveRecord, Rake, RSpec, Capistrano, Cucumber
나) 기타 : SQL, CSS, Regular Expression(정규식), Make, graphviz





intellij.... 하나의 프로젝트 밑에.. 모듈을 여러개 두어서....
그 모듈이 동일하지만, 하나는 java, 하나는 kotlin... 이렇게...
jar export는 모듈 단위로... 안 되나?
