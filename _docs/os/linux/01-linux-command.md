---
title: "linux 계열 명령어 정리"
permalink: /docs/linux-command/
excerpt: "linux command list"
last_modified_at: 2019-04-18T15:53:52-04:00
redirect_from:
  - /theme-setup/
toc: true
---

### 공통

1. 


### centos
1. 종료
```
  shutdown -h now     // 지금 바로 종료하고, 컴퓨터를 종료한다.
  shutdown -r now     // 지금 바로 종료 후 재부팅한다.
```
https://btyy.tistory.com/48

2. 유선 또는 무선 인터넷 사용.
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