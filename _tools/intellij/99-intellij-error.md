---
title: "intellij란"
permalink: /tools/intellij/intellij-error
excerpt: "intellij"
tags: ["tools", "intellij", "error"]
date: 2029-02-19 13:00:00 +0900
last_modified_at: 2020-01-24T17:00:00+09:00
redirect_from:
  - /theme-setup/
toc: true
toc_label: "2020.02.19"
description : intellij 사용 시 error
---

### error: unmappable character for encoding MS949
Gradle을 이용하여, 테스트 등을 진행 할 경우, \\
해당 에러가 발생하며, 한글이 깨지는 것을 볼 수 있습니다. \\
Gradle은 현재 실행하는 시스템의 encoding 셋팅을 그대로 따라가는데, \\
윈도우는 기본적으로 MS949이기 때문에 이런 오류가 발생하게 됩니다. \\

![VM options 설정](https://github.com/bumsgy/bumsgy.github.io/blob/master/assets/images/refimg/tools/error_01.JPG)

이미지와 같이 gradle 설정 중 VM options을 설정 해 줍니다. \\

