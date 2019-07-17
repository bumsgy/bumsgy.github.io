---
title: 허....탈...
date: 2019-07-04
last_modified_at: 2019-07-04T13:00:00+09:00
tags: ["diary", "springboot", "dev"]
category : ["diary", "springboot", "dev"]
description : 나 혼자 삽질하는것만 해도 힘든데...
toc: true
toc_label: ""
toc_icon: "cog"  # corresponding Font Awesome icon name (without fa prefix)
---

### springboot에 swagger 적용
어제 springboot에 swagger를 적용해 보고 있었습니다.
swagger2를 gradle로 import 해서 셋팅을 하려는데...
<details>
<summary>접기/펼치기</summary>

```java
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.lguplus.rcs.wt") )
                .paths(PathSelectors.any())
                .build()
                .apiInfo(apiInfo());
    }
```
</details>

여기서 documentation.type, selector 등등이 몽땅 에러.. \\
클래스를 못 찾는단다.. 아니, 클래스 따라가는 것도 되고, 다 되는데.. \\
당췌 왜 안 되는건지... 집에 가서도 이리저리 찾아봤지만... 다 에러... \\
오늘 출근해서 다시 프로젝트 만들고 해 보니 잘 되네요.. \\
예상은 이것저것 잘못 import 하면서 guava 버젼이 엉겨서 그런 것으로 예상하고 있습니다. \\
그나저나, gradle에 jar를 다 삭제하고 다시 올리려 해도 왜 이리 어려운 것인지.. \\
dependecies 내용 모두 주석처리하면 만사OK가 될 줄 알았건만 그렇지는 않더라구요. \\
방법을 좀 더 알아봐야 할 것 가네요. 



### spring boot에 logback 적용 .
	logback.xml 같이 환경 설정 파일을 만들때 logback이라고만 쓰면 안 되는듯.
	logback-dev.xml 등등으로 생성해야 정상적으로 load
