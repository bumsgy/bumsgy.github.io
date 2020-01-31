---
title: java Error 정리
permalink: /dev/java/java-error-summary
collection: dev
excerpt: {"java", "error"}
last_modified_at: 2019-08-30T17:00:00+09:00
redirect_from:
  - /theme-setup/
toc: true
---


### The hierarchy of the type {{class}} is inconsistent
이 에러가 발생시 extends를 하거나 했을 텐데, super class를 따라가보면, \\
밑에 항목인 The type {{class}} cannot be resolved. \\
It is indirectly referenced from required .class files\\
에러가 발생해 있을 것입니다. 

### The type {{class}} cannot be resolved. It is indirectly referenced from required .class files
해당 class를 찾을 수 없다고 하는 것이니, \\
해당 클래스 또는 클래스가 포함된 jar 파일이 import 되었는지\\
확인하고 조치해야 합니다.
