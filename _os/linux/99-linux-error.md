---
title: "linux 환경변수 설정 방법"
permalink: /os/linux/linux-error/
excerpt: "linux error"
last_modified_at: 2019-09-03T16:00:00+09:00
redirect_from:
  - /theme-setup/
toc: true
---

##### 임시 환경 변수 설정

- ifconfig 커맨드가 안 될 경우.
 -> net-tools 설치
```bash
  # yum install net-tools
```















centos 설치
    - centos를 설치하여 hyperledger를 셋팅하기로 함
    - https://centos.org/  방문
      centos는 다운로드를 CentOS Linux DVD ISO 또는 CentOS Stream DVD ISO로 
      다운로드 할 수 있다고 함.


   - Linux와 Stream 2가지를 지원한다고 표시.
     두 가지의 차이는? 

1.1.1. 검색 결과- https://hamonikr.org/board_bFBk25/48349  - 정기적인 배포판이 존재하기 보다는, 언제든지 새로운 버전이 출시되면 업데이트가   가능한 배포판의 종류- 이번 설치에서는 일반적인 linux 버전으로 설치 결정- 해당 링크로 들어가 본 결과, mirror 리스트만 나오는데, 여기에는 centos 8 버전만 표시됨.




  - nodejs를 설치하기 위해서는 먼저 epel 저장소를 설치해야 한다고 함.
   - epel이 무엇인가 검색.
1) https://zetawiki.com/wiki/YUM_epel_%EC%A0%80%EC%9E%A5%EC%86%8C_%EC%B6%94%EA%B0%80
2) 확인 결과, Extra Packages for Enterprise Linux yum의 줄임말이며,엔터프라이즈 리눅스를 위한 추가 패키지라고 함
3) nodejs 설치를 위한 repo 설치





 개발된 내용에 대한 설명(예)

   - 3분의 1 미만의 컴퓨터가 악의적으로 행동해도 나머지 3분의 2 이상의 컴퓨터가 이에 대한 합의를 이루는 형태의 PBFT(Practical Byzantine Fault Tolerance) 합의 알고리즘 개발

   - 최초 설정된 리더노드가 클라이언트로부터 수집된 요청을 정렬하여 블록을 생성하는 모듈 개발

   - 리더노드에서 생성된 블록을 각 노드로 전송하는 모듈 개발

   - 각 노드별 검증을 통해 노드별로 원장에 기록하고 실행결과 다른 노드들에게 전송 기능 개발﻿

   ==> 이러한 형태로 개발된 내용에 대해 글짓기가 필요합니다.  



2. 도식

   - Flow-Chart 또는 프로세스 흐름도 또는 데이터 흐름도 형태



3. 소스

   - 현재 작성된 소스 이미지 2장 포함﻿




   가십 구성요소는 다음 2개의 모듈로 구성된다.

discovery 모듈 - 활성 상태 및 응답하지 않는 피어 세트 유지
통신 모듈 - 연결 유지 및 메시지 배포

discovery 모듈은 통신 모듈을 사용하여 자신의 정보를 보내고, 통신 모듈은 discovery 관련된 메세지를 discovery 모듈에 전달.

discovery 프로토콜 흐름 
- 각 피어는 생성시 부트스트랩 피어 세트 B를 받는다.
- 각 피어는 알려진 멤버 집합을 나타내는 <id, endpoint, logical_timeestamp> 의 세트 V와 응답하지 않는 피어의 유사한 세트 H를 가지고 있으며, V[id]는  ID 요소를 가진 피어의 마지막 활성 메시지를 나타낸다.
- 메시지는 네트워크 CA에서 발급한 Pub(Priv) 키를 이용하여 암호화된 방식으로 서명(검증)할 수 있다.

discovery 프로토콜 정의 

1) H=B, V={}
2) 시작할 때 H의 모든 p에 연결을 시도 하고 성공하면 p를 V에 추가하고 통신 모듈을 업데이트.
3) 각 p(i) 로 부터 그들이 가지고 있는 V 를 통합하여 자신의 V를 구성. 
4) V가 변경된 경우 통신 모듈을 업데이트하여 새로 추가된 모든 피어에 대한 연결을 생성하고 응답하지 않는 피어에 대한 연결을 제거
5) 매 T초 마다 피어 p는  통신 모듈에 자신의  alive 메시지를 전달한다. 메시지에는 다음이 포함된다.  
          P(id), P(endpoint), m (inc_number: P가 돌기 시작한 시간, seq# :각 alive메세지마다 하나씩 증가 )
6) discovery 세트 V의 각 p 멤버에 대해, Alive 메시지의 마지막 시간을 추적한다. 받은지 5초가지났다면 
         V세트로부터 그 p를 제거 / H세트에 그 p 추가 / 통신모듈에서 커넥션을 종료 
7) 어떤 피어 p에서 Alive 메시지 m 을 수신 시:
     - m.id이 V에 없으면 추가 
    - m.timestamp.inc_number > V[id].timestamp.inc_number , m.timestamp.seq# > V[id].timestamp.seq# 면 V에 추가 

통신  프로토콜 흐름 
- 원장에 전달되지 않은 메시지에 대한 내부 버퍼 유지
- 피어에서 수신한 각 메시지에는 확인할 수 있는 ID가 있는데 이 ID는 Alive 메시지인 경우 <m.timeestamp, p.id> 또는 원장 배포 블록인 경우 원장 블록의 해시입니다. 어느 쪽이든 프로토콜 통신 프로토콜은 메시지의 ID를 m.id로 참조합니다.
- 각 피어는 이전에 발생한 메시지 ID의 세트 R과 수신 시간을 주기적으로 보유하고 5 초 이상 전에 수신 된 오래된 메시지 아이디를 삭제합니다.


통신  프로토콜 정의 
1) 시간 t에 피어 p(i)로부터 메시지 m을 수신하면 -> 메시지가 Alive 메시지 인 경우 ->discovery 모듈로 전달
2) 이전에 m.id ID를 받은 경우 (R에서 발견할수있음) 메시지를 삭제
3) 그렇지 않다면  
   3-1)  <m.id, m.t> 를 R 로 추가 
   3-2)  V로 부터 k = log (| V |) + 3 개의 무작위 피어를 선택 
          - 각각의 p(i)가 응답하지 않으면 discovery 모듈을 업데이트하여 p(i) 를 H에 추가
          - m 을 무작위 피어 p(i) 들에 전파 (모아 보낼 수도)
4) T 초마다 한 번, V의 각 P에 대해 k = log (| V |) + 3 개의 무작위 응답 피어 세트를 선택하고 
   4-1) V 전체를 p(i) 들에게 보냄.
   4-2) V(p,i), H(p,i)를 얻어서 V(p,i) , H(p,i) 에 있는 각 p를 V,H에 추가한다. 
   4-3) Send to p your highest ledger originated message sequence number s for which you received before all m s.t ,i < s
   4-4) If you have before received a message m(j) that wasn't passed to the ledger yet such that j>s , send to p a bitmap b of all sequence numbers from s to j , And receive back from p(i) a set of messages M(i) and (optionally) its own s , b
   4-5) M의 메시지를 내부적으로 저장
   4-6) (Optionally) send back to p(i) a crafted M of your own according to p(i) 's s(i) , b(i) sent
5) 사용자 또는 discovery 모듈로부터 메시지 m을 수신하면 :
   5-1) <m.id, m.t>를 R에 추가하십시오.
   5-2) k = log (| V |) + 3 개의 랜덤 피어 집합을 선택하고 m을 그들에게 보낸다.



출처: https://hamait.tistory.com/988 [HAMA 블로그]
