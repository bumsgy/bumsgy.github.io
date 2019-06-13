---
title: "linux 계열 명령어 정리"
permalink: /docs/os/linux/linux-command/
excerpt: "linux command list"
last_modified_at: 2019-04-18T15:53:52-04:00
redirect_from:
  - /theme-setup/
toc: true
---

### linux

1. 종류 \\
   리눅스는 여러 종류의 리눅스가 있습니다. \\
   리눅스 자체가 opensource이다 보니, 여러 기업에서 이를 이용하여, 자신들의 리눅스를 만들어 제공한 거죠. \\
   Ubuntu, Fedora, Centos 등등이 있습니다. \\

2. 계열 \\
   또한, 계열이 있는데, 이를 알고 있어야 편합니다. \\
   몇몇 명령어가 다르기 때문이죠. \\
   크게 Debian 계열, RedHat 계열로 구분합니다. \\
   Debian 계열에는 대표적으로 Ubuntu가 있습니다. \\
   RedHat 계열에는 Centos, Fedora 등이 있습니다. \\

   debian 계열에서는 설치 시 apt-get을 사용하고, \\
   RedHat 계열에서는 yum을 사용합니다. \\

3. 파일 또는 폴더의 경로 입력 시, 맨 앞에 /를 입력한다면 root부터의 경로로 인식합니다. \\
   /부터 시작하지 않으면, 현재 위치부터 상대 경로로 인식합니다. \\


### 공통
1. 종료 \\
```bash
  shutdown -h now     // 지금 바로 종료하고, 컴퓨터를 종료한다.
  shutdown -r now     // 지금 바로 종료 후 재부팅한다.
```
https://btyy.tistory.com/48 \\

2. 압축 \\
압축은 gzip과 b... 가 있습니다.


3. 압축해제 \\
```bash
  tar -xvzf apache-tomcat-8.0.52.tar.gz
  mv apache-tomcat-8.0.53 /usr/local
  cd /usr/local/
  ln -s apache-tomcat-8.0.53 tomcat
```
참고로 tar의 경우, 파일명으로 폴더를 생성 후, 해당 폴더 안에 압축을 해제 해 줍니다. \\

많이 사용하는 옵션들입니다. \\
이 정보들은 tar --help 로 확인하실 수 있습니다. \\
x : -x, --extract, --get                extract files from an archive : 압축파일로부터 압축을 해제합니다. \\
v : -v, --verbose                       verbosely list files processed : 진행되는 상황을 화면에 표시합니다 \\
z : -z, --gzip, --gunzip, --ungzip      filter the archive through gzip : gzip으로 압축해제합니다.  \\
f : -f, --file=ARCHIVE                  use archive file or device ARCHIVE :  \\

참조 blog[^1]

4. 삭제
  삭제 명령어로는 rm을 사용합니다. \ 
  가장 많이 사용하는 옵션은 -rf로서, \
  -r 옵션 은 하위디렉토리, 폴더 내 파일이 있을 경우, 같이 삭제합니다. \ 
  ( 옵션을 안 써준다면, 하위 디렉토리나 파일이 있으면 삭제가 되지 않습니다. ) \
  -f 옵션은 삭제 시 확인 과정을 거치지 않습니다.  \
  관리자 계정으로 로그인해서 rm -rf / 를 입력하는 실수는 하지 않도록 주의합니다.  \
  ( 이렇게 하면 모든 데이터가 삭제됩니다. 주의, 또 주의!! ) \
```bash
  rm -rf <삭제를 원하는 폴더, 또는 파일의 경로>
```

5. 이동, 파일 & 폴더명 변경


### centos






2. 유선 또는 무선 인터넷 사용
```
  ifconfig <interface명> up     // 인터페이스를 작동시킨다.
  또는,
  ip link set dev <interface명> up      // 인터페이스를 작동시킨다.  중간의 dev는 어떤 역할인지 확인 필요

  ifconfig <interface명>  // 해당 interface명에 대한 정보만 표시.
```

https://blog.work6.kr/60
https://withjeon.com/2018/03/09/wifi-in-linux/
https://wikidocs.net/3200

그냥 바로 무선 설정을 하고 싶지만, wpa_supplicant 명령어가 있어야 하는데, 
centos에 이것이 없다. 이를 설치를 해야 무선을 쓸 수 있고, 그러려면 일단 유선을... ;;;





[^1]: sample.