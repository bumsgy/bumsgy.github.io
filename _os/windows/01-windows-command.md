---
title: "windows 계열 명령어 정리"
permalink: /os/windows/command/
excerpt: "windows command list"
last_modified_at: 2019-04-18T15:53:52-04:00
redirect_from:
  - /theme-setup/
toc: true
---

### netstat
netstat은 현재 TCP/IP 네트워크 연결을 표시하는 명령어입니다. \\
옵션의 경우, 서로 연결해서 사용 할 수 있습니다. \\
a는 모든 연결과 수신 대기 포트를 표시합니다. \\
n은 주소와 포트 번호를 숫자 형식으로 표시합니다. \\
o는 각 연결의 소유자 프로세스 ID를 표시합니다. \\
이를 한 꺼번에 연결한다면 
```bash
  netstat -ano 
```
이렇게 사용할 수 있습니다.

모든 옵션을 알고 싶다면 \\
```bash
  netstat --help
```

### findstr 
findstr 명령어는 문자열을 찾는 명령어입니다. \\
linux의 grep과 비슷한 형식으로 사용이 가능한 듯합니다. \\

```bash 
  findstr /?
```
명령어로 모든  옵션을 볼 수 있습니다. \\
정규식도 사용이 가능하네요.ㄴ