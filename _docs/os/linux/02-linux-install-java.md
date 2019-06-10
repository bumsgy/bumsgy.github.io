---
title: "linux java 설치"
permalink: /docs/linux-install-java/
excerpt: "linux java install"
last_modified_at: 2019-06-10T10:00:00-04:00
redirect_from:
  - /theme-setup/
toc: true
---

### 설치 전 확인

1. yum을 이용하여, 설치 가능한 java버전을 조회 해 봅니다.
     ```unix
      yum list java*jdk*

      // jdk만 보고 싶다면.
      yum list java*jdk-devel
    ```

여기서 
java-버전-openjdk 패키지가 JRE,
java-버전-openjdk-devel 패키지가 JDK

jdk, 즉 devel은 jdk에 의존성이 있어,
설치 시 jdk가 없다면, 먼저 설치하게 된다.

### 설치 
1. 설치 가능 목록 중 원하는 버전으로 설치합니다.
    ```unix
      yum install <원하는 설치 버전>
    ```

### 확인
1. 설치가 잘 되었는지 확인 해 봅니다.
    ```unix
      java -version
    ```