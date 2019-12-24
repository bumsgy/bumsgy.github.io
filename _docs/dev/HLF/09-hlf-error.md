---
title: Docker 개념
permalink: /docs/dev/docker/docker-concept/
excerpt: "docker concept"
last_modified_at: 2019-08-30T17:00:00+09:00
redirect_from:
  - /theme-setup/
toc: true
---

### Docker란?
너무 많은 곳에서 설명하고 있으니, 그 분들 링크~


### 




### Ledger ID already exists






docker-compose 
expected <block end>, but found '?'

indent가 2space. 4space면 해당 에러.



### 




ca.crt가 아닌 ca.example.com-cert.pem으로 되어 있는데..
tlsCerts/orderer라는 경로가 없음. 다시 한 번 확인

일단 비교 해 보니 orderer에 
adminprivatekey, signcertpem도 추가되어 있음.
그에 따라, 이 항목 다시 추가 해 봄.

    adminPrivateKey:
      path: server/common/hlf/connection-profile/ldt-network/crypto-config/ordererOrganizations/nmplus.com/users/Admin@nmplus.com/msp/keystore/key.pem
    signedCert:
      path: server/common/hlf/connection-profile/ldt-network/crypto-config/ordererOrganizations/nmplus.com/users/Admin@nmplus.com/msp/signcerts/Admin@org2.nmplus.com-cert.pem



동일.

설정 명칭에 PEM이 적혀있으므로, 동일하게 해 봄.
adminPrivateKey => adminPrivateKeyPem


에러는 동일.

getPeer - name peer0.org1.example.com, channel_org: undefined 에 대해 검색 중 
https://jira.hyperledger.org/secure/attachment/15166/logs.txt
내용에서 'undefined' 가 되어도 정상적으로 기동되는 것으로 예상.


오류 메시지 
"hyperledger" routines:SSL3_GET_RECORD:wrong version number 에 대해 검색
https://github.com/IBM-Blockchain/ibm-blockchain-issues/issues/32
 : 예전 버전에 대한 내용. 
  
https://jira.hyperledger.org/browse/BE-521
등등 여러 곳을 본 결과, TLS가 제대로 셋팅이 안 된 것은 아닌가 의심됨.
TLS 셋팅을 확인 해 봐야 할 듯

docker-compose에서 각 peer에 대한 tls는 띄우지않았는데, 
이를 풀어볼까?

설정하고 모든 network 재기동.
api 테스트 결과 에러 동일

혹시나, 
peer의 log를 확인 결과, chaincode 오류 메세지가 표시

--------------------------------------------------------------------------------------
2019-09-18 08:20:37.345 UTC [gossip.service] func1 -> INFO 044 Elected as a leader, starting delivery service for channel mychannel
2019-09-18 08:20:58.787 UTC [endorser] callChaincode -> INFO 045 [mychannel][e8f16a33] Entry chaincode: name:"lscc" 
2019-09-18 08:20:58.787 UTC [lscc] executeDeployOrUpgrade -> ERRO 046 cannot get package for chaincode (balance:0.1)-err:open /var/hyperledger/production/chaincodes/balance.0.1: no such file or directory
2019-09-18 08:20:58.787 UTC [endorser] callChaincode -> INFO 047 [mychannel][e8f16a33] Exit chaincode: name:"lscc"  (0ms)
2019-09-18 08:20:58.787 UTC [endorser] ProcessProposal -> ERRO 048 [mychannel][e8f16a33] simulateProposal() resulted in chaincode name:"lscc"  response status 500 for txid: e8f16a335c3ef4a569980a1e0bba09278f92a8c0dc2c6844f25dd58467a27656
--------------------------------------------------------------------------------------

chaincode에 대한 설정이 잘못 된 것은 아닌가 확인 필요.

volumes 설정이 잘못 되어 있었음.
- ../../:/var/hyperledger/production  으로 변경. 해당 폴더 하위에 chaincode 가 있음

하지만 에러 동일.
balance와 버전을 명시하는데 폴더로 찾아 들어가는데... 이것이 맞는지 확인 필요.


hyperledger-study02 를 서버 구성하여 기동.
이 때, chaincode가 동일하게 에러 발생하는지 확인을 위함.

확인 후 다시 ldt 서버 구성 기동.
peer 쪽 로그를 확인 해 보려 하니, peer1.org1 을 제외한 3개의 peer가 오류로 서버 기동이 제대로 안 됨.
지난 수정 후 오류가 발생하고 있었던 것으로 판단됨.

다시 수정해보니 TLS 설정은 문제가 없음.

    - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/peerOrg2/tls/server.key
    - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/peerOrg2/tls/server.crt
    - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/peerOrg2/tls/ca.crt

볼륨 설정에서 문제.
그런데, chaincode 설정을 위해서는 경로를 잡아 줘야 하는데, 이를 어떻게 해야 하는지. 확인 해 봐야 함.
    - ../../:/var/hyperledger/production
일단은 주석 처리 후 혹시 다른 부분에서 오류가 있는 것인지 확인.

서버 기동 이후,
channer 생성부터 peer 로그 확인.
cli 에서 채널 생성 및 join 확인.
cli 로그 확인 결과,

--------------------------------------------------------------------------------------
DEBU 02b MSP configuration file not found at [/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.nmplus.com/users/Admin@org2.nmplus.com/msp/config.yaml]: [stat /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.nmplus.com/users/Admin@org2.nmplus.com/msp/config.yaml: no such file or directory]
--------------------------------------------------------------------------------------

등의 에러 메세지가 발생하는 것을 파악했으나, 실제로 없는 파일인 것으로 파악됨.
configtx에서 생성이 안 되는 것인데, 이를 만들어줘야 하는 것인지는 차후 다시 확인 필요.

cli에서 create chaincode는 에러가 발생 없으나, install에서 에러 확인

--------------------------------------------------------------------------------------
2019-09-19 03:24:21.042 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 043 Using default escc
2019-09-19 03:24:21.042 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 044 Using default vscc
Error: error getting chaincode code balance: path to chaincode does not exist: /opt/gopath/src/github.com/hyperledger/fabric/examples/chaincode/go/balance_cc
--------------------------------------------------------------------------------------
경로 확인 

==> 경로에 nmplus로 설정했으나, 로그 사에서는 examples로 표시되고 있음.

확인 결과,
cli 폴더의 setup-chaincode.sh
안에서 경로를 설정하게 되어 있는데, 여기서 경로를 example로 설정하고 있었음. 
이를 변경.

--------------------------------------------------------------------------------------
CHAINCODE_DIR=github.com/hyperledger/fabric/nmplus/chaincode/go/
--------------------------------------------------------------------------------------


다시 서버 기동 중.
log 를 보니, join channel에서 에러 발생.
일단은 그대로 진행.

log 확인 결과
체인 코드는 정상적으로 생성...
--------------------------------------------------------------------------------------
[container] WriteFileToPackage -> DEBU 04c Writing file to tarball: src/github.com/hyperledger/fabric/nmplus/chaincode/go/balance_cc/example_cc.go
2019-09-19 03:36:33.428 UTC [msp.identity] Sign -> DEBU 04d Sign: plaintext: 0A98070A5C08031A0C08C1EF8BEC0510...CF3BFE130000FFFF770284D200240000 
2019-09-19 03:36:33.428 UTC [msp.identity] Sign -> DEBU 04e Sign: digest: C36018F247A9F7A0B5C41C70CCA4984D7086504DB58F302542903C68A875C05A 
2019-09-19 03:36:33.435 UTC [chaincodeCmd] install -> INFO 04f Installed remotely response:<status:200 payload:"OK" > 
--------------------------------------------------------------------------------------

그러나, rpc error
--------------------------------------------------------------------------------------
Error: error endorsing chaincode: rpc error: code = Unknown desc = access denied: channel [mychannel] creator org [Org1MSP]
--------------------------------------------------------------------------------------
join channel 과 연관이 있을 것으로 예상.

왜 join에서 오류가 나는 것인지 먼저 확인 해야 할 듯.

각 서버들 제대로 기동되는지 다시금 확인.

------------------------------------------------------------
er with ID=[name:"peer0.org1.nmplus.com" ], network ID=[dev], address=[peer0.org1.nmplus.com:7051]
2019-09-19 04:05:13.885 UTC [kvledger] LoadPreResetHeight -> INFO 029 Loading prereset height from path [/var/hyperledger/production/ledgersData/chains]
2019-09-19 04:05:13.885 UTC [fsblkstorage] LoadPreResetHeight -> INFO 02a Loading Pre-reset heights
2019-09-19 04:05:13.885 UTC [fsblkstorage] preRestHtFiles -> INFO 02b Dir [/var/hyperledger/production/ledgersData/chains/chains] missing... exiting
2019-09-19 04:05:13.885 UTC [fsblkstorage] LoadPreResetHeight -> INFO 02c Pre-reset heights loaded
2019-09-19 04:05:13.959 UTC [comm.grpc.server] 1 -> INFO 02d unary call completed grpc.service=gossip.Gossip grpc.method=Ping grpc.request_
------------------------------------------------------------
chains 부분이 missing으로 나오는데.. 이게 문제 발생 여지가 있는가?

서버 셋팅을 다시 하고, 재기동을 해 본 결과,
이번에는 join channel 정상 작동. ( 기존 오류는 왜 발생했던 것인가? )

chaincode 쉘 실행.
실행 로그 확인 결과.
install 에서는 오류 미발생.
instantiate 에서 오류 발생.
------------------------------------------------------------
2019-09-19 04:21:04.917 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 049 Using default escc
2019-09-19 04:21:04.917 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 04a Using default vscc
2019-09-19 04:21:04.918 UTC [msp.identity] Sign -> DEBU 04b Sign: plaintext: 0AA3070A6708031A0C08B0848CEC0510...314D53500A04657363630A0476736363 
2019-09-19 04:21:04.918 UTC [msp.identity] Sign -> DEBU 04c Sign: digest: B5AA2D63D73F9252D43AF3DC94B355CD659D4BE62CC843E22E1878B4E9F06890 
Error: could not assemble transaction, err proposal response was not successful, error code 500, msg error starting container: error starting container: Post http://unix.sock/containers/create?name=dev-peer0.org1.nmplus.com-balance-0.1: dial unix /var/run/docker.sock: connect: no such file or directory
------------------------------------------------------------

peer에서도 확인 
peer는 peer0.org1에 요청하고 있으므로, 해당 peer 서버 로그를 확인
------------------------------------------------------------
[mychannel][24da8de4] Entry chaincode: name:"lscc" 
2019-09-19 04:21:04.927 UTC [dockercontroller] Start -> ERRO 04b create container failed: Post http://unix.sock/containers/create?name=dev-peer0.org1.nmplus.com-balance-0.1: dial unix /var/run/docker.sock: connect: no such file or directory imageName=dev-peer0.org1.nmplus.com-balance-0.1-141fbb64ecf988ff7d8c48309fe571483e64e47ce738900805b516dc5b046497 containerName=dev-peer0.org1.nmplus.com-balance-0.1
2019-09-19 04:21:04.928 UTC [endorser] callChaincode -> INFO 04c [mychannel][24da8de4] Exit chaincode: name:"lscc"  (8ms)
2019-09-19 04:21:04.928 UTC [endorser] SimulateProposal -> ERRO 04d [mychannel][24da8de4] failed to invoke chaincode name:"lscc" , error: Post http://unix.sock/containers/create?name=dev-peer0.org1.nmplus.com-balance-0.1: dial unix /var/run/docker.sock: connect: no such file or directory
error starting container
error starting container
2019-09-19 04:21:04.928 UTC [comm.grpc.server] 1 -> INFO 04e unary call completed grpc.service=protos.Endorser grpc.method=ProcessProposal grpc.peer_address=172.25.0.13:42098 grpc.code=OK grpc.call_duration=9.876384ms
------------------------------------------------------------


failed to invoke chaincode name:"lscc" 로 검색 중
https://miiingo.tistory.com/178 에서 chaincode 를 각 peer에 설치를 해 줘야 하는 것으로 파악.
setup-chaincode 쉘 수정.
------------------------------------------------------------
CORE_PEER_ADDRESS=peer1.org1.nmplus.com:7055 peer chaincode install -n $1 -v $2 -p $CHAINCODE_DIR/$1_cc
CORE_PEER_ADDRESS=peer0.org2.nmplus.com:8051 peer chaincode install -n $1 -v $2 -p $CHAINCODE_DIR/$1_cc
CORE_PEER_ADDRESS=peer1.org2.nmplus.com:8055 peer chaincode install -n $1 -v $2 -p $CHAINCODE_DIR/$1_cc
------------------------------------------------------------


join channel 시 에러가 계속 발생.
------------------------------------------------------------
2019-09-19 05:33:35.844 UTC [grpc] HandleSubConnStateChange -> DEBU 03d pickfirstBalancer: HandleSubConnStateChange: 0xc0005b81a0, TRANSIENT_FAILURE
Error: error getting endorser client for channel: endorser client failed to connect to peer1.org2.nmplus.com:7051: failed to create new connection: connection error: desc = "transport: error while dialing: dial tcp 172.25.0.11:7051: connect: connection refused"
------------------------------------------------------------
docker 이미지를 모두 삭제 후 다시 생성하면 오류가 발생하지 않는 것을 확인.
이미지를 삭제하지 않으면 에러가 발생 함.

이후,이미지 모두 삭제 후 다시 생성하였으나, 오류 발생.
.....

여러번 이미지 생성, HLF network 구성, channel 생성 및 join channel 테스트
결과, 오류가 생길 경우, 정상적으로 join이 될 경우도 존재.
dependency가 안 되어 있어서 그런 것이 아닌가 예상.

좀더, 테스트 결과.
오류가 났을 경우에는 컨테이너를 삭제하고, 다시 생성해도 에러가 나고,
정상일 경우에는 컨테이너를 삭제하고, 다시 생성해도 정상이다.
dependency는 컨테이너 실행 시 의존성이 주입되는 것으로,
그보다는 이미지 다운로드 등에서 에러가 발생하는 것은 아닌가 예측.

그것도 아닌 듯. 이후에 테스트 중. 정상으로 되었는데, 컨테이너만 삭제 후 다시 생성했는데 오류가 발생.




에러 메세지
"hyperledger" /var/run/docker.sock: connect: no such file or directory Done 
에 대해 검색 중 
https://stackoverflow.com/questions/55173477/hyperledger-fabric-dial-unix-host-var-run-docker-sock-connect-no-such-file-o
답변 중 
CORE_VM_ENDPOINT=http://172.17.0.1:2375  로 변경해 보란 이야기가 있어 
peer들 설정에 추가.
테스트 결과 
------------------------------------------------------------
Error: could not assemble transaction, err proposal response was not successful, error code 500, msg error starting container: error starting container: Post http://172.17.0.1:2375/containers/create?name=dev-peer0.org1.nmplus.com-balance-0.1: dial tcp 172.17.0.1:2375: connect: no route to host
------------------------------------------------------------
메시지 내용은 바뀌었으나, 원인은 비슷한 것으로 예상ㄷ됨.

api에서 handshake가 안 되는 것으로 보아.
TLS 설정이 있어야 하는 것은 아닌가 예상.
order의 TLS 설정을 주석 해제.

결과 
------------------------------------------------------------
2019-09-19 08:56:19.583 UTC [grpc] HandleSubConnStateChange -> DEBU 04e pickfirstBalancer: HandleSubConnStateChange: 0xc0003e2090, CONNECTING
2019-09-19 08:56:19.584 UTC [grpc] HandleSubConnStateChange -> DEBU 04f pickfirstBalancer: HandleSubConnStateChange: 0xc0003e2090, READY
Error: rpc error: code = Unavailable desc = transport is closing
------------------------------------------------------------
로 에러가 발생하며, join channel도 그 이유로 불가.

order log를 확인 해 본 결과
------------------------------------------------------------
2019-09-19 08:56:19.580 UTC [core.comm] ServerHandshake -> ERRO 009 TLS handshake failed with error tls: first record does not look like a TLS handshake server=Orderer remoteaddress=172.25.0.13:42174
2019-09-19 08:56:19.584 UTC [core.comm] ServerHandshake -> ERRO 00a TLS handshake failed with error tls: first record does not look like a TLS handshake server=Orderer remoteaddress=172.25.0.13:42176
------------------------------------------------------------
TLS 에러 확인.
TLS 설정에 대해 확인을 해 봐야 할 듯하다.


handshake를 위해서는 orderer, peer 모두 tls를 설정해야 하고,
peer 볼륨에는 자신이 속한 tls 설정만 있으면 되지 않을까 예상.

------------------------------------------------------------
      - ../crypto-artifacts/peerOrganizations/org1.nmplus.com/peers/peer0.org1.nmplus.com:/etc/hyperledger/peerOrg1
      - ../crypto-artifacts/peerOrganizations/org2.nmplus.com/peers/peer0.org2.nmplus.com:/etc/hyperledger/peerOrg2
------------------------------------------------------------
이 설정 부분에서 자신이 속한 org의 peer만 남긴 후 이를 이용하여 tls 설정



hyperledger/fabric-sample/first-network 분석.
확인 결과, cli, ca 등등에 모두 tls 셋팅이 필요.

ca는 각각의 org의 인증서 파일을 이용. ==> 확인 결과 기존에 셋팅.
cli는 peer0.org1에 대한 인증서.

보던 중, peer에 chaincodelistenaddress를 설정 해 주고 있다.
이는 필수인 것인가? 적용 후 테스트 필요.
일단 해당 설정 내용은 상기 내용을 먼저 해 보고난 후 적용.

또한, peer 쪽에 tls 사용을 enable 하는 설정 내용이 누락되어 있음.
이 또한, 영향을 끼칠 수 있는가? 차례로 테스트.

cli에 tls 설정 적용.
------------------------------------------------------------
2019-09-20 04:29:19.344 UTC [grpc] HandleSubConnStateChange -> DEBU 040 pickfirstBalancer: HandleSubConnStateChange: 0xc000484330, TRANSIENT_FAILURE
2019-09-20 04:29:20.867 UTC [grpc] HandleSubConnStateChange -> DEBU 041 pickfirstBalancer: HandleSubConnStateChange: 0xc000484330, CONNECTING
2019-09-20 04:29:20.869 UTC [grpc] createTransport -> DEBU 042 grpc: addrConn.createTransport failed to connect to {peer1.org2.nmplus.com:7051 0  <nil>}. Err :connection error: desc = "transport: authentication handshake failed: tls: first record does not look like a TLS handshake". Reconnecting...
2019-09-20 04:29:20.869 UTC [grpc] HandleSubConnStateChange -> DEBU 043 pickfirstBalancer: HandleSubConnStateChange: 0xc000484330, TRANSIENT_FAILURE
Error: error getting endorser client for channel: endorser client failed to connect to peer1.org2.nmplus.com:7051: failed to create new connection: context deadline exceeded
------------------------------------------------------------
오류 내용은 기존과 좀 다르나, 결론은 연결이 불가.

먼저, peer 의 tls 사용 설정/.
동시에 GOSSIP 쪽 설정이 제대로 되어 있지 않았기에 이 것도 같이 설정.
------------------------------------------------------------
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org1.nmplus.com:7055
      - CORE_PEER_GOSSIP_ENDPOINT=peer0.org1.nmplus.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.nmplus.com:7051
------------------------------------------------------------
CORE_PEER_GOSSIP_BOOTSTRAP은 자신이 아닌, 같은 org에 있는 다른 peer를 설정해 줘야 한다.
endpoint와 externalendpoint는 자기 자신.

chaincode의 설정에 port는 어디 설정 중에 존재하지 않는 port이다.. 이 포트는 어디에 설정되어 있는 port인가?
------------------------------------------------------------
    environment:
      - CORE_PEER_ID=peer1.org3.example.com
      - CORE_PEER_ADDRESS=peer1.org3.example.com:12051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:12051
      - CORE_PEER_CHAINCODEADDRESS=peer1.org3.example.com:12052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:12052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org3.example.com:11051
------------------------------------------------------------

사용 설정에 2개를 설정해 놓았었는데, 안 쓰던 port로 설정을 해 봄.
------------------------------------------------------------
    ports:
      - 7051:7051
      - 7053:7053
------------------------------------------------------------
이때... 내부로 해야 하는가... 외부로 해야 하는가...
일단 외부로...

기동 후 peer0 로그 확인.
------------------------------------------------------------
 grpc.peer_subject="CN=peer1.org1.nmplus.com,L=San Francisco,ST=California,C=US" grpc.code=OK grpc.call_duration=251.822쨉s
2019-09-20 05:39:39.326 UTC [gossip.discovery] func1 -> WARN 030 Could not connect to Endpoint: peer1.org1.nmplus.com:7055, InternalEndpoint: peer1.org1.nmplus.com:7055, PKI-ID: <nil>, Metadata:  : context deadline exceeded
2019-09-20 05:40:07.328 UTC [gossip.discovery] func1 -> WARN 031 Could not connect to Endpoint: peer1.org1.nmplus.com:7055, InternalEndpoint: peer1.org1.nmplus.com:7055, PKI-ID: <nil>, Metadata:  : context deadline exceeded
------------------------------------------------------------
gossip 설정이 잘못 되어 있는가? 발생.
connection이 이루어지지 않는데... 현재 docker외부 설정으로 port가 설정되어 있음.
이를 docker 내부로 변경해야 하는가?

일단 org1에 대해서만 내부 port로 변경 후 테스트
------------------------------------------------------------
2019-09-20 05:46:47.898 UTC [comm.grpc.server] 1 -> INFO 02d unary call completed grpc.service=gossip.Gossip grpc.method=Ping grpc.request_deadline=2019-09-20T05:46:49.898Z grpc.peer_address=172.25.0.11:53712 grpc.peer_subject="CN=peer1.org1.nmplus.com,L=San Francisco,ST=California,C=US" grpc.code=OK grpc.call_duration=147.031쨉s
2019-09-20 05:46:47.904 UTC [comm.grpc.server] 1 -> INFO 02e streaming call completed grpc.service=gossip.Gossip grpc.method=GossipStream grpc.request_deadline=2019-09-20T05:46:57.9Z grpc.peer_address=172.25.0.11:53712 grpc.peer_subject="CN=peer1.org1.nmplus.com,L=San Francisco,ST=California,C=US" error="rpc error: code = Canceled desc = context canceled" grpc.code=Canceled grpc.call_duration=4.62685ms
2019-09-20 05:46:47.911 UTC [comm.grpc.server] 1 -> INFO 02f unary call completed grpc.service=gossip.Gossip grpc.method=Ping grpc.request_deadline=2019-09-20T05:46:49.91Z grpc.peer_address=172.25.0.11:53714 grpc.peer_subject="CN=peer1.org1.nmplus.com,L=San Francisco,ST=California,C=US" grpc.code=OK grpc.call_duration=126.57쨉s
2019-09-20 05:46:48.394 UTC [comm.grpc.server] 1 -> INFO 030 streaming call completed grpc.service=gossip.Gossip grpc.method=GossipStream grpc.peer_address=172.25.0.11:53714 grpc.peer_subject="CN=peer1.org1.nmplus.com,L=San Francisco,ST=California,C=US" error="rpc error: code = Canceled desc = context canceled" grpc.code=Canceled grpc.call_duration=482.828391ms
------------------------------------------------------------
확인 결과 에러 메세지는 없어지는 것으로 파악.

다시금... 천천히 보니...
ca와 cli 만 tls 셋팅이 되어 있음...
다른 샘플들에서 orderer, peer에 셋팅 되어 있는 것과는 무엇이 다른 것인가?

일단 다른 소스들의 셋팅을 정리.
--- hyperledger-study02
   tool/network : 

--- hyperledger-study01  도 있어서 이 또한 git에서 내려 받아 확인
하다 보니. peer 컨테이너의 port가 3개 존재.
각각의 port가 무엇에 쓰이고 있는지 확인 


https://openblockchain.readthedocs.io/en/latest/Setup/Network-setup/

If you are using the native Docker for Mac or Windows, the value for CORE_VM_ENDPOINT should be set to unix:///var/run/docker.sock. [TODO] double check this. I believe that 127.0.0.1:2375 also works.
--> - CORE_VM_ENDPOINT=unix:///var/run/docker.sock  부분은 항상 써 주는 것이 좋을 듯.
    또는 - CORE_VM_ENDPOINT=http://172.17.0.1:2375

혹시, 이로 인하여 오류가 발생 할 수 있으므로,
하나씩 변경 후 각각 컨테이너 기동, 컨테이너 삭제, 수정, 컨테이너 기동 순으로 진행.

기동에 오류는 없음. 
일단 다시금 channel join 테스트

------------------------------------------------------------
2019-09-20 08:23:58.874 UTC [grpc] createTransport -> DEBU 042 grpc: addrConn.createTransport failed to connect to {peer1.org2.nmplus.com:7051 0  <nil>}. Err :connection error: desc = "transport: authentication handshake failed: x509: certificate signed by unknown authority". Reconnecting...
2019-09-20 08:23:58.874 UTC [grpc] HandleSubConnStateChange -> DEBU 043 pickfirstBalancer: HandleSubConnStateChange: 0xc000072190, TRANSIENT_FAILURE
2019-09-20 08:23:59.155 UTC [grpc] func1 -> DEBU 044 Failed to dial peer1.org2.nmplus.com:7051: context canceled; please retry.
Error: error getting endorser client for channel: endorser client failed to connect to peer1.org2.nmplus.com:7051: failed to create new connection: context deadline exceeded
------------------------------------------------------------
connection 에러는 동일


다시 기존 진행하는 것으로 돌아가서,
orderer는 TLS 설정 삭제, peer, cli에 추가

------------------------------------------------------------
e dialing: dial tcp 172.25.0.9:7051: connect: connection refused". Reconnecting...
2019-09-20 08:36:07.818 UTC [grpc] HandleSubConnStateChange -> DEBU 03d pickfirstBalancer: HandleSubConnStateChange: 0xc0002178f0, TRANSIENT_FAILURE
Error: error getting endorser client for channel: endorser client failed to connect to peer1.org2.nmplus.com:7051: failed to create new connection: connection error: desc = "transport: error while dialing: dial tcp 172.25.0.9:7051: connect: connection refused"
------------------------------------------------------------
결과, channel 생성은 되지만, join channel은 오류 발생.
생성은 orderer에서, join은 peer들.

좀더 상세히 확인 해 보니, orderer 서버 구동 시 에러는 미발생.
이후, 채널 생성 중 오류 발생.


https://lists.hyperledger.org/g/fabric/topic/31696729
등의 검색 내용을 보면 
cluster 관련 셋팅이 존재.
설정 추가 후 테스트.
------------------------------------------------------------
2019-09-23 02:58:10.789 UTC [core.comm] ServerHandshake -> ERRO 00a TLS handshake failed with error tls: first record does not look like a TLS handshake server=Orderer remoteaddress=172.25.0.13:39232
2019-09-23 02:58:11.787 UTC [core.comm] ServerHandshake -> ERRO 00b TLS handshake failed with error tls: first record does not look like a TLS handshake server=Orderer remoteaddress=172.25.0.13:39234
2019-09-23 02:58:11.793 UTC [core.comm] ServerHandshake -> ERRO 00c TLS handshake failed with error tls: first record does not look like a TLS handshake server=Orderer remoteaddress=172.25.0.13:39236
------------------------------------------------------------
TLS 셋팅에서 에러. 오류 동일.


### 문득!!
TLS 셋팅은 모두 삭제 후,
CA 셋팅으로만 HLF Network 기동 후, HFC에서 샘플 테스트.
만약 된다면, chaincode 등을 확인 해 봐야 하는 것으로 판단해야 하지 않을까?

------------------------------------------------------------
2019-09-23 03:51:29.792 UTC [endorser] callChaincode -> INFO 04a [mychannel][558fa39f] Entry chaincode: name:"lscc" 
2019-09-23 03:51:33.807 UTC [dockercontroller] Start -> ERRO 04b create container failed: Post http://172.17.0.1:2375/containers/create?name=dev-peer0.org1.nmplus.com-balance-0.1: dial tcp 172.17.0.1:2375: connect: no route to host imageName=dev-peer0.org1.nmplus.com-balance-0.1-141fbb64ecf988ff7d8c48309fe571483e64e47ce738900805b516dc5b046497 containerName=dev-peer0.org1.nmplus.com-balance-0.1
2019-09-23 03:51:36.815 UTC [endorser] callChaincode -> INFO 04c [mychannel][558fa39f] Exit chaincode: name:"lscc"  (7023ms)
2019-09-23 03:51:36.815 UTC [endorser] SimulateProposal -> ERRO 04d [mychannel][558fa39f] failed to invoke chaincode name:"lscc" , error: Post http://172.17.0.1:2375/containers/create?name=dev-peer0.org1.nmplus.com-balance-0.1: dial tcp 172.17.0.1:2375: connect: no route to host
error starting container
error starting container
------------------------------------------------------------
peer 쪽 log 확인 결과, chaincode에서 에러 발생.

어느 부분에서 에러가 발생하기 시작한 것인지 로그 파악 결과.
------------------------------------------------------------
2019-09-23 04:03:29.052 UTC [grpc] DialContext -> DEBU 037 scheme "" not registered, fallback to default scheme
2019-09-23 04:03:29.052 UTC [grpc] watcher -> DEBU 038 ccResolverWrapper: sending new addresses to cc: [{peer1.org1.nmplus.com:7055 0  <nil>}]
2019-09-23 04:03:29.052 UTC [grpc] switchBalancer -> DEBU 039 ClientConn switching balancer to "pick_first"
2019-09-23 04:03:29.052 UTC [grpc] HandleSubConnStateChange -> DEBU 03a pickfirstBalancer: HandleSubConnStateChange: 0xc000438570, CONNECTING
2019-09-23 04:03:29.054 UTC [grpc] createTransport -> DEBU 03b grpc: addrConn.createTransport failed to connect to {peer1.org1.nmplus.com:7055 0  <nil>}. Err :connection error: desc = "transport: error while dialing: dial tcp 172.25.0.12:7055: connect: connection refused". Reconnecting...
2019-09-23 04:03:29.054 UTC [grpc] HandleSubConnStateChange -> DEBU 03c pickfirstBalancer: HandleSubConnStateChange: 0xc000438570, TRANSIENT_FAILURE
Error: error getting endorser client for install: endorser client failed to connect to peer1.org1.nmplus.com:7055: failed to create new connection: connection error: desc = "transport: error while dialing: dial tcp 172.25.0.12:7055: connect: connection refused"
------------------------------------------------------------
peer1.org1에 install이 안 된다는 에러 발생. 
이 부분 확인. 
peer0.org2, peer1.org2도 동일.
peer01.org1만 설치 된 것으로 파악. install 시 설정값을 바꿔주어야 하는 것은 아닌지 확인.
------------------------------------------------------------
CORE_PEER_ADDRESS=peer0.org1.nmplus.com:7051 peer chaincode install -n $1 -v $2 -p $CHAINCODE_DIR/$1_cc
CORE_PEER_ADDRESS=peer1.org1.nmplus.com:7055 peer chaincode install -n $1 -v $2 -p $CHAINCODE_DIR/$1_cc
CORE_PEER_ADDRESS=peer0.org2.nmplus.com:8051 peer chaincode install -n $1 -v $2 -p $CHAINCODE_DIR/$1_cc
CORE_PEER_ADDRESS=peer1.org2.nmplus.com:8055 peer chaincode install -n $1 -v $2 -p $CHAINCODE_DIR/$1_cc
------------------------------------------------------------
설정에서 혹시 port가 외부가 아닌 내부인 것은 아닌가?  설정을 각각 변경해 주는 것은 이 부분밖에 없음.
모두 7051 내부 port로 변경.

컨테이너 삭제 후 다시 기동 
logs 확인 결과.
------------------------------------------------------------
yload -> DEBU 04b done
2019-09-23 04:08:45.751 UTC [container] WriteFileToPackage -> DEBU 04c Writing file to tarball: src/github.com/hyperledger/fabric/nmplus/chaincode/go/balance_cc/example_cc.go
2019-09-23 04:08:45.755 UTC [msp.identity] Sign -> DEBU 04d Sign: plaintext: 0A98070A5C08031A0C08CD8AA1EC0510...CF3BFE130000FFFF770284D200240000 
2019-09-23 04:08:45.755 UTC [msp.identity] Sign -> DEBU 04e Sign: digest: 0A751B7DB8D3353BE76745BB9B1D663B198DF18CD1D48D6BB2F37B8841A11A4E 
Error: Error endorsing chaincode: rpc error: code = Unknown desc = access denied: channel [] creator org [Org1MSP]
------------------------------------------------------------
org1 관련 되어서는 에러가 발생하지 않는 것으로 보아, org2로 변경해 줘야 할 듯.


CORE_PEER_LOCALMSPID=Org2MSP 추가 후 테스트
------------------------------------------------------------
 Writing file to tarball: src/github.com/hyperledger/fabric/nmplus/chaincode/go/balance_cc/example_cc.go
2019-09-23 04:18:41.046 UTC [msp.identity] Sign -> DEBU 04d Sign: plaintext: 0A97070A5B08031A0B08A18FA1EC0510...CF3BFE130000FFFF770284D200240000 
2019-09-23 04:18:41.046 UTC [msp.identity] Sign -> DEBU 04e Sign: digest: D0F7DDB2E2013C5ACE2548091654A092CB0FE1439A75618336CF6AD3D1D4D0F2 
Error: Error endorsing chaincode: rpc error: code = Unknown desc = access denied: channel [] creator org [Org2MSP]
------------------------------------------------------------
지속적으로 계속 에러. 설정의 다른 부분을 더 추가 해줘야 하는가....

이 부분도 추가 해 봄.
------------------------------------------------------------
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.nmplus.com/users/Admin@org2.nmplus.com/msp
------------------------------------------------------------
install 성공.

하지만, instantiate 에러
------------------------------------------------------------
Sign: plaintext: 0AA7070A6708031A0C088094A1EC0510...324D53500A04657363630A0476736363 
2019-09-23 04:28:48.469 UTC [msp.identity] Sign -> DEBU 04c Sign: digest: D290E90CEEFAE80CA7B5A30C331BD5D5AA0D5092F051682032EBF2D2895AEA5F 
Error: error endorsing chaincode: rpc error: code = Unknown desc = access denied: channel [mychannel] creator org [Org1MSP]
------------------------------------------------------------

동일하게 다시 설정을 바꿔 주면 되지 않을까 예상.
instantiate 실행 명령어 분석.
// TODO
그 중 -p 옵션은 무엇인가?

instantiate 에러 메세지 중. 
------------------------------------------------------------
2019-09-23 06:14:48.178 UTC [msp.identity] Sign -> DEBU 04b Sign: plaintext: 0AA2070A6608031A0B08D8C5A1EC0510...535010030A04657363630A0476736363 
2019-09-23 06:14:48.178 UTC [msp.identity] Sign -> DEBU 04c Sign: digest: ED6B010A2BEEB176482A24A4DF00508393FDCA4A10899D5E4614D76919536C98 
Error: could not assemble transaction, err proposal response was not successful, error code 500, msg error starting container: error starting container: Post http://172.17.0.1:2375/containers/create?name=dev-peer0.org1.nmplus.com-balance-0.1: dial tcp 172.17.0.1:2375: connect: no route to host
------------------------------------------------------------

Error: could not assemble transaction 검색

https://stackoverflow.com/questions/55289283/hyperledger-fabric-error-could-not-assemble-transaction-proposalresponsepaylo 
내용 중 peer chaincode invoke
이 있어서, peer 컨테이너로 들어가서 확인.

peer chaincode invoke 뒤에 -C 옵션으로 chaincode 명칭 입력 후 다시 확인
------------------------------------------------------------
Error: error getting channel (balance) orderer endpoint: error bad proposal response 500: access denied for [GetConfigBlock][balance]: Failed to get policy manager for channel [balance]
------------------------------------------------------------
내용을 보았을 때, policy 문제가 맞는 듯함.


org는 1개로 peer를 10개로 설정 수정.


1. crypto-config/crypto-config.yaml 수정.
   => template 10으로 변경
      org2는 삭제
2. ./cryptogen.sh 실행
3. configtx.yaml 파일 수정.
  OneOrgOrdererGenesis 추가
4. configtxgen.sh 실행.
5. ca2 설정 삭제
   : docker-config/docker-compose-ca.yaml
6. docker-cli는 그대로
7. docker-orderer는 그대로.
8. docker-peer 
    peer2~9까지 추가
9. 그에 따라 couchdb2~9까지 추가
10. 이 때, depends_on 에 의존성 추가 해 줘야 한다.

HLF Network 기동 테스트

join channel에 2~9까지 peer 추가
org2에 대한 내용은 삭제.

channel 생성 및 join 테스트

chaincode 2~9까지 추가.
instantiate도 동일. install은 정상.

다시 로그 확인 join 채널에서 에러 발생.
------------------------------------------------------------
 pickfirstBalancer: HandleSubConnStateChange: 0xc000338de0, TRANSIENT_FAILURE
Error: error getting endorser client for channel: endorser client failed to connect to peer0.org1.nmplus.com:7051: failed to create new connection: connection error: desc = "transport: error while dialing: dial tcp 172.25.0.14:7051: connect: connection refused"
------------------------------------------------------------

확인 결과
------------------------------------------------------------
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.nmplus.com:7051
      - CORE_PEER_GOSSIP_ENDPOINT=peer7.org1.nmplus.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer7.org1.nmplus.com:7051
------------------------------------------------------------
설정을 거꾸로 했음. ENDPOINT와 EXTERNALENDPOINT는 자기 자신.
BOOTSTRAP은 최초 기동되는 서버로 설정.

결과 join까지 정상 작동.

좀 더 상세히 로그 확인 결과 gossip 설정에서 에러가 발생하는 듯.
------------------------------------------------------------
019-09-23 14:30:29.244 UTC [gossip.gossip] JoinChan -> INFO 046 Joining gossip network of channel mychannel with 1 organizations
2019-09-23 14:30:29.244 UTC [gossip.gossip] learnAnchorPeers -> INFO 047 Learning about the configured anchor peers of Org1MSP for channel mychannel : [{peer0.org1.nmplus.com 0} { 7051}]
2019-09-23 14:30:29.244 UTC [gossip.gossip] learnAnchorPeers -> WARN 048 Got invalid port (0), skipping connecting to anchor peer {peer0.org1.nmplus.com 0}
2019-09-23 14:30:29.244 UTC [gossip.gossip] learnAnchorPeers -> WARN 049 Got empty hostname, skipping connecting to anchor peer { 7051}
2019-09-23 14:30:29.246 UTC [gossip.service] updateEndpoints -> WARN 04a Failed to update ordering service endpoints, due to Channel with mychannel id was not found
2019-09-23 14:30:29.249 UTC [committer.txvalidator] Validate -> INFO 04b [mychannel] Validated block [1] in 60ms
2019-09-23 14:30:29.283 UTC [gossip.channel] reportMembershipChanges -> INFO 04c Membership view has changed. peers went online:  [[peer2.org1.nmplus.com:7051] [peer7.org1.nmplus.com:7051] [peer8.org1.nmplus.com:7051] [peer1.org1.nmplus.com:7051] [peer4.org1.nmplus.com:7051] [peer6.org1.nmplus.com:7051] [peer5.org1.nmplus.com:7051] [peer0.org1.nmplus.com:7051]] , current view:  [[peer2.org1.nmplus.com:7051] [peer7.org1.nmplus.com:7051] [peer8.org1.nmplus.com:7051] [peer1.org1.nmplus.com:7051] [peer4.org1.nmplus.com:7051] [peer6.org1.nmplus.com:7051] [peer5.org1.nmplus.com:7051] [peer0.org1.nmplus.com:7051]]
------------------------------------------------------------
warning인데... 확인 한 번 해야 할듯.

GOSSIP 정리.
GOSSIP은 리더피어로부터 모든 피어 정보를 가져옴.
그러므로, bootstrap은 리더피어를 바라보도록 해야 하며,
ENDPOINT도 리더피어를 보도록 한다.
EXTERNALENDPOINT는 자신을 외부에서 인식시키기 위한 설정으로 자신으로 설정.
------------------------------------------------------------
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.nmplus.com:7051
      - CORE_PEER_GOSSIP_ENDPOINT=peer0.org1.nmplus.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer7.org1.nmplus.com:7051
------------------------------------------------------------

채널 join 이후 update 로 anchorpeer 부분은 삭제.
이 부분은 org가 1개이므로 필요없음.
이후 확인 결과 join까지 모두 완료.

chaincode instantiate에서 여전히 에러 발생.
peer 확인 시 channel이 없다고 로그.
------------------------------------------------------------
channel [mychannel]: MSP error: channel doesn't exist
2019-09-24 01:55:33.442 UTC [comm.grpc.server] 1 -> INFO 033 unary call completed grpc.service=protos.Endorser grpc.method=ProcessProposal grpc.peer_address=172.25.0.24:53944 error="access denied: channel [mychannel] creator org [Org1MSP]" grpc.code=Unknown grpc.call_duration=711.173쨉s
------------------------------------------------------------

서로 통신이 제대로 안 되는 것으로 예상.
------------------------------------------------------------
 - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
------------------------------------------------------------
설정 변경후 network 다시 기동.

instantiate는 여전히 에러가 나며,
peer 로그를 확인 해 본 결과.
------------------------------------------------------------
2019-09-24 02:34:09.340 UTC [endorser] callChaincode -> INFO 048 [mychannel][c69659eb] Exit chaincode: name:"lscc"  (44566ms)
2019-09-24 02:34:09.340 UTC [endorser] SimulateProposal -> ERRO 049 [mychannel][c69659eb] failed to invoke chaincode name:"lscc" , error: container exited with 0
github.com/hyperledger/fabric/core/chaincode.(*RuntimeLauncher).Launch.func1
        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/runtime_launcher.go:63
runtime.goexit
        /opt/go/src/runtime/asm_amd64.s:1333
chaincode registration failed
2019-09-24 02:34:09.340 UTC [comm.grpc.server] 1 -> INFO 04a unary call completed grpc.service=protos.Endorser grpc.method=ProcessProposal grpc.peer_address=172.25.0.24:55296 grpc.code=OK grpc.call_duration=44.568818312s
------------------------------------------------------------





peer chaincode instantiate -o orderer.nmplus.com:7050 --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/nmplus.com/orderers/orderer.nmplus.com/msp/cacerts/ca.nmplus.com-cert.pem -C mychannel -n $1 -v $2 -c '{"Args":["init","a","200","b","200"]}' -P "OR   ('Org1MSP.member')"

instantiate 시 policy가 설정되어 있지 않아,
발생하는 것으로 파악.
이는 configtx.yaml에 설정 되는 것으로 확인.
sample을 이용해 다시 설정.
이후 다시 cryptogen, configtxgen 모두 다시 실행.


order가 계속 exit으로 떨어짐. 
하위 설정이 잘못 되어 있다고 함.
------------------------------------------------------------
OrdererOrg cannot contain endpoints value until V1_4_2+ capabilities have been enabled
panic: Failed validating bootstrap block: initializing channelconfig failed: could not create channel Orderer sub-group config: Orderer Org OrdererOrg cannot contain endpoints value until V1_4_2+ capabilities have been enabled

goroutine 1 [running]:
github.com/hyperledger/fabric/vendor/go.uber.org/zap/zapcore.(*CheckedEntry).Write(0xc00032a000, 0x0, 0x0, 0x0)
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/go.uber.org/zap/zapcore/entry.go:229 +0x515
github.com/hyperledger/fabric/vendor/go.uber.org/zap.(*SugaredLogger).log(0xc0000ae1f8, 0xc000132d04, 0x103a88e, 0x25, 0xc000479d10, 0x1, 0x1, 0x0, 0x0, 0x0)
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/go.uber.org/zap/sugar.go:234 +0xf6
github.com/hyperledger/fabric/vendor/go.uber.org/zap.(*SugaredLogger).Panicf(0xc0000ae1f8, 0x103a88e, 0x25, 0xc000479d10, 0x1, 0x1)
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/go.uber.org/zap/sugar.go:159 +0x79
github.com/hyperledger/fabric/common/flogging.(*FabricLogger).Panicf(0xc0000ae200, 0x103a88e, 0x25, 0xc000479d10, 0x1, 0x1)
        /opt/gopath/src/github.com/hyperledger/fabric/common/flogging/zap.go:74 +0x60
github.com/hyperledger/fabric/orderer/common/server.Start(0x1018e03, 0x5, 0xc00028f200)
        /opt/gopath/src/github.com/hyperledger/fabric/orderer/common/server/main.go:98 +0xcd
github.com/hyperledger/fabric/orderer/common/server.Main()
        /opt/gopath/src/github.com/hyperledger/fabric/orderer/common/server/main.go:91 +0x1ce
main.main()
        /opt/gopath/src/github.com/hyperledger/fabric/orderer/main.go:15 +0x20
------------------------------------------------------------

configtx.yaml 외에는 크게 변경한 점이 없어서 해당 파일을 본 결과.
혹시 rule 안에 들어가는 명칭이 ID가 아닌가 예상. 변경.
------------------------------------------------------------
  - &OrdererOrg
      Name: OrdererOrg
      ID: OrdererMSP
      MSPDir: crypto-artifacts/ordererOrganizations/nmplus.com/msp
      Policies: &OrdererOrgPolicies
        Readers:
          Type: Signature
          Rule: "OR('OrdererMSP.member')"
        Writers:
          Type: Signature
          Rule: "OR('OrdererMSP.member')"
        Admins:
          Type: Signature
          Rule: "OR('OrdererMSP.admin')"
      OrdererEndpoints:
        - "127.0.0.1:7050"
      AnchorPeers:
        - Host: 127.0.0.1
          Port: 7051
------------------------------------------------------------

계속 에러.. 이 부분이 아닌 듯.

에러 메시지 중 endpoints ... 설정 중 . OrdererEndpoints 부분을 삭제.
해당 부분은 configtx.yaml이므로, configtxgen.sh 를 다시 실행 한 후
HLF network 을 재기동해야 한다.

이후 에러 메시지 변경
------------------------------------------------------------
019-09-24 09:19:09.018 UTC [orderer.common.broadcast] ProcessMessage -> WARN 008 [channel: mychannel] Rejecting broadcast of config message from 172.25.0.24:47224 because of error: config does not validly parse: initializing channelconfig failed: could not create channel Application sub-group config: ACLs may not be specified without the required capability
------------------------------------------------------------

capability를 무조건 삭제하면 안 되고, 설정 해 줘야 하는 듯함.


모든 capability 모두 주석 제거.
org2 항목 삭제.
다시 테스트 결과, 정상 작동

join channel 이후 chaincode.
------------------------------------------------------------
019-09-24 10:32:34.604 UTC [gossip.service] func1 -> INFO 040 Elected as a leader, starting delivery service for channel mychannel
2019-09-24 10:32:37.608 UTC [ConnProducer] NewConnection -> ERRO 041 Failed connecting to {127.0.0.1:7050 [OrdererMSP]} , error: context deadline exceeded
2019-09-24 10:32:37.608 UTC [ConnProducer] NewConnection -> ERRO 042 Could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}]
2019-09-24 10:32:37.608 UTC [deliveryClient] connect -> ERRO 043 Failed obtaining connection: could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}]
2019-09-24 10:32:37.609 UTC [deliveryClient] try -> WARN 044 Got error: could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}] , at 1 attempt. Retrying in 1s
2019-09-24 10:32:41.611 UTC [ConnProducer] NewConnection -> ERRO 045 Failed connecting to {127.0.0.1:7050 [OrdererMSP]} , error: context deadline exceeded
2019-09-24 10:32:41.611 UTC [ConnProducer] NewConnection -> ERRO 046 Could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}]
2019-09-24 10:32:41.611 UTC [deliveryClient] connect -> ERRO 047 Failed obtaining connection: could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}]
2019-09-24 10:32:41.612 UTC [deliveryClient] try -> WARN 048 Got error: could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}] , at 2 attempt. Retrying in 2s
2019-09-24 10:32:46.614 UTC [ConnProducer] NewConnection -> ERRO 049 Failed connecting to {127.0.0.1:7050 [OrdererMSP]} , error: context deadline exceeded
2019-09-24 10:32:46.615 UTC [ConnProducer] NewConnection -> ERRO 04a Could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}]
2019-09-24 10:32:46.615 UTC [deliveryClient] connect -> ERRO 04b Failed obtaining connection: could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}]
2019-09-24 10:32:46.615 UTC [deliveryClient] try -> WARN 04c Got error: could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}] , at 3 attempt. Retrying in 4s
2019-09-24 10:32:53.618 UTC [ConnProducer] NewConnection -> ERRO 04d Failed connecting to {127.0.0.1:7050 [OrdererMSP]} , error: context deadline exceeded
2019-09-24 10:32:53.618 UTC [ConnProducer] NewConnection -> ERRO 04e Could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}]
2019-09-24 10:32:53.618 UTC [deliveryClient] connect -> ERRO 04f Failed obtaining connection: could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}]
2019-09-24 10:32:53.618 UTC [deliveryClient] try -> WARN 050 Got error: could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}] , at 4 attempt. Retrying in 8s
2019-09-24 10:32:57.617 UTC [endorser] callChaincode -> INFO 051 [][18b6cdba] Entry chaincode: name:"lscc" 
2019-09-24 10:32:57.646 UTC [couchdb] CreateDatabaseIfNotExist -> INFO 052 Created state database mychannel_lscc
------------------------------------------------------------
connection이 제대로 안 되고 있음.

다시 확인 결과.
channel부터 오류 발생.
------------------------------------------------------------
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.nmplus.com:7051
      - CORE_PEER_GOSSIP_ENDPOINT=peer0.org1.nmplus.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer7.org1.nmplus.com:7051
------------------------------------------------------------
설정 오류로 다시 설정.


instantiate  중 에러..

------------------------------------------------------------
2019-09-24 13:05:40.375 UTC [comm.grpc.server] 1 -> INFO 031 unary call completed grpc.service=protos.Endorser grpc.method=ProcessProposal grpc.peer_address=172.25.0.24:42770 grpc.code=OK grpc.call_duration=5.503493ms
2019-09-24 13:05:41.728 UTC [protoutils] ValidateProposalMessage -> WARN 032 channel [mychannel]: MSP error: channel doesn't exist
2019-09-24 13:05:41.728 UTC [comm.grpc.server] 1 -> INFO 033 unary call completed grpc.service=protos.Endorser grpc.method=ProcessProposal grpc.peer_address=172.25.0.24:42786 error="access denied: channel [mychannel] creator org [Org1MSP]" grpc.code=Unknown grpc.call_duration=779.748쨉s
------------------------------------------------------------

확인을 위해 다시 재기동했더니,
위의 connection 에러가 재발생.

channel create는 되는데, join channel에서 에러 발생.
orderer에 접근을 못 한다고 함. 
여러 검색으로도 확인을 못 함.
그 중, https://stackoverflow.com/questions/47978986/hyperledger-fabric-peer-connection-error-with-orderer?rq=1 에서
내부 IP인데, 이에 접근하지 못 하는 것은 아닌가 예상하며,
기존 configtx.yaml 과 비교.

------------------------------------------------------------
2019-09-24 14:01:05.983 UTC [gossip.service] func1 -> INFO 040 Elected as a leader, starting delivery service for channel mychannel
2019-09-24 14:01:08.988 UTC [ConnProducer] NewConnection -> ERRO 041 Failed connecting to {127.0.0.1:7050 [OrdererMSP]} , error: context deadline exceeded
2019-09-24 14:01:08.988 UTC [ConnProducer] NewConnection -> ERRO 042 Could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}]
2019-09-24 14:01:08.989 UTC [deliveryClient] connect -> ERRO 043 Failed obtaining connection: could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}]
2019-09-24 14:01:08.989 UTC [deliveryClient] try -> WARN 044 Got error: could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}] , at 1 attempt. Retrying in 1s
2019-09-24 14:01:12.992 UTC [ConnProducer] NewConnection -> ERRO 045 Failed connecting to {127.0.0.1:7050 [OrdererMSP]} , error: context deadline exceeded
2019-09-24 14:01:12.992 UTC [ConnProducer] NewConnection -> ERRO 046 Could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}]
2019-09-24 14:01:12.992 UTC [deliveryClient] connect -> ERRO 047 Failed obtaining connection: could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}]
2019-09-24 14:01:12.992 UTC [deliveryClient] try -> WARN 048 Got error: could not connect to any of the endpoints: [{127.0.0.1:7050 [OrdererMSP]}] , at 2 attempt. Retrying in 2s
------------------------------------------------------------

configtx.yaml의 orderer 설정.
------------------------------------------------------------
Orderer: &OrdererDefaults

    # Orderer Type: The orderer implementation to start.
    # Available types are "solo" and "kafka".
    OrdererType: solo

    # Addresses used to be the list of orderer addresses that clients and peers
    # could connect to.  However, this does not allow clients to associate orderer
    # addresses and orderer organizations which can be useful for things such
    # as TLS validation.  The preferred way to specify orderer addresses is now
    # to include the OrdererEndpoints item in your org definition
    Addresses:
        # - 127.0.0.1:7050
      - orderer.nmplus.com:7050
------------------------------------------------------------
address 부분 추가.
이후 테스트
join channel 까지는 정상.


하지만, instantiate chaincode는 에러가 그대로 발생.
------------------------------------------------------------
2019-09-24 14:19:23.267 UTC [endorser] SimulateProposal -> ERRO 049 [mychannel][474751a9] failed to invoke chaincode name:"lscc" , error: container exited with 0
github.com/hyperledger/fabric/core/chaincode.(*RuntimeLauncher).Launch.func1
        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/runtime_launcher.go:63
runtime.goexit
        /opt/go/src/runtime/asm_amd64.s:1333
chaincode registration failed
------------------------------------------------------------

또한 못하는 것의 하나로 network 접근이 제대로 안 되는 듯하여 
peer의 environment에 network 설정추가 
------------------------------------------------------------
environment:
- CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=nmp-net
------------------------------------------------------------



왜, channel이 존재하지 않는다고 에러가 나는가?
join channel만 한 후, peer에 접속해서 channel이 있는지 확인 해 보자.
------------------------------------------------------------
2019-09-24 14:52:44.028 UTC [comm.grpc.server] 1 -> INFO 031 unary call completed grpc.service=protos.Endorser grpc.method=ProcessProposal grpc.peer_address=172.25.0.24:47804 grpc.code=OK grpc.call_duration=5.399524ms
2019-09-24 14:52:49.699 UTC [protoutils] ValidateProposalMessage -> WARN 032 channel [mychannel]: MSP error: channel doesn't exist
2019-09-24 14:52:49.699 UTC [comm.grpc.server] 1 -> INFO 033 unary call completed grpc.service=protos.Endorser grpc.method=ProcessProposal grpc.peer_address=172.25.0.24:47844 error="access denied: channel [mychannel] creator org [Org1MSP]" grpc.code=Unknown grpc.call_duration=682.382쨉s
------------------------------------------------------------

원인 불명. 하나하나 join chainnel이 제대로 되었는지 확인 해 보니,
된 것으로 나오며, 에러 제거.
이후 다시 나올 수 있음.

------------------------------------------------------------
2019-09-24 15:20:18.383 UTC [endorser] callChaincode -> INFO 049 [mychannel][3fe2b891] Entry chaincode: name:"lscc" 
2019-09-24 15:20:18.434 UTC [chaincode.platform.golang] GenerateDockerBuild -> INFO 04a building chaincode with ldflagsOpt: '-ldflags "-linkmode external -extldflags '-static'"'
2019-09-24 15:20:58.515 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-balance-0.1] func2 -> INFO 04b 2019-09-24 15:20:58.514 UTC [shim] userChaincodeStreamGetter -> ERRO 001 context deadline exceeded
2019-09-24 15:20:58.515 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-balance-0.1] func2 -> INFO 04c error trying to connect to local peer
2019-09-24 15:20:58.516 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-balance-0.1] func2 -> INFO 04d github.com/hyperledger/fabric/core/chaincode/shim.userChaincodeStreamGetter
2019-09-24 15:20:58.516 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-balance-0.1] func2 -> INFO 04e    /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/shim/chaincode.go:112
2019-09-24 15:20:58.516 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-balance-0.1] func2 -> INFO 04f github.com/hyperledger/fabric/core/chaincode/shim.Start
2019-09-24 15:20:58.516 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-balance-0.1] func2 -> INFO 050    /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/shim/chaincode.go:151
2019-09-24 15:20:58.516 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-balance-0.1] func2 -> INFO 051 main.main
2019-09-24 15:20:58.516 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-balance-0.1] func2 -> INFO 052    /chaincode/input/src/github.com/hyperledger/fabric/nmplus/chaincode/go/balance_cc/example_cc.go:290
2019-09-24 15:20:58.516 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-balance-0.1] func2 -> INFO 053 runtime.main
2019-09-24 15:20:58.516 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-balance-0.1] func2 -> INFO 054    /opt/go/src/runtime/proc.go:201
2019-09-24 15:20:58.516 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-balance-0.1] func2 -> INFO 055 runtime.goexit
2019-09-24 15:20:58.516 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-balance-0.1] func2 -> INFO 056    /opt/go/src/runtime/asm_amd64.s:1333
2019-09-24 15:20:58.516 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-balance-0.1] func2 -> INFO 057 2019-09-24 15:20:58.515 UTC [example_cc0] Errorf -> ERRO 002 Error starting Simple chaincode: error trying to connect to local peer: context deadline exceeded
2019-09-24 15:20:58.826 UTC [dockercontroller] func2 -> INFO 058 Container dev-peer0.org1.nmplus.com-balance-0.1 has closed its IO channel
2019-09-24 15:20:58.954 UTC [endorser] callChaincode -> INFO 059 [mychannel][3fe2b891] Exit chaincode: name:"lscc"  (40571ms)
2019-09-24 15:20:58.955 UTC [endorser] SimulateProposal -> ERRO 05a [mychannel][3fe2b891] failed to invoke chaincode name:"lscc" , error: container exited with 0
github.com/hyperledger/fabric/core/chaincode.(*RuntimeLauncher).Launch.func1
        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/runtime_launcher.go:63
runtime.goexit
        /opt/go/src/runtime/asm_amd64.s:1333
chaincode registration failed
------------------------------------------------------------

상기의 channel이 없다는 메세지가 다시 나와 로그를 확인 해 본 결과,
peer0만 join이 안 되었음.
create 한 후 바로 join해서 그런건 아닌지...
sleep으로 시간차를 좀 두고, 다시 테스트.

peer0, 1, 2가 join 안 됨.
이후 peer는 join.

3도 안 됨.
3이 됨.
sleep를 한 번 확 늘려보기로 함.
sleep 10으로 한 이후,
2 정상.
다시 2 에러.
sleep 20으로 변경. 

join 시 에러 발생 안 함. 
최종. 각 peer join에도 sleep 1씩. 에러 미발생

백그라운드로 실행시키지 않고, 서버 구동 후 블록 생성 로그 확인.
결과 블록이 생성되지 않음.
cli에서 직접 invoke 

------------------------------------------------------------
docker exec -it 8b7a52d6fd1a /bin/bash
peer chaincode invoke -o orderer.nmplus.com -C mychannel -n balance 
------------------------------------------------------------

------------------------------------------------------------
2019-09-24 18:51:33.742 UTC [grpc] createTransport -> DEBU 048 grpc: addrConn.createTransport failed to connect to {orderer.nmplus.com 0  <nil>}. Err :connection error: desc = "transport: error while dialing: dial tcp: address orderer.nmplus.com: missing port in address". Reconnecting...
2019-09-24 18:51:33.742 UTC [grpc] HandleSubConnStateChange -> DEBU 049 pickfirstBalancer: HandleSubConnStateChange: 0xc000518880, TRANSIENT_FAILURE
Error: error getting broadcast client: orderer client failed to connect to orderer.nmplus.com: failed to create new connection: connection error: desc = "transport: error while dialing: dial tcp: address orderer.nmplus.com: missing port in address"
------------------------------------------------------------
에러가 발생. 이 부분 때문에 orderer에서 블록을 생성하지 못 하고 있는 것으로 예상.


검색 결과. 
https://www.edureka.co/community/29586/hyperledger-orderer-client-connect-orderer-failed-connection
내용에서 
------------------------------------------------------------
      - ORDERER_GENERAL_LOGLEVEL=${DOCKER_DEBUG}
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      - ORDERER_GENERAL_GENESISMETHOD=file
------------------------------------------------------------
port 설정 확인. port 설정 추가 후 재 기동

오류 비교 결과 동일.
------------------------------------------------------------
2019-09-24 19:12:01.072 UTC [grpc] createTransport -> DEBU 048 grpc: addrConn.createTransport failed to connect to {orderer.nmplus.com 0  <nil>}. Err :connection error: desc = "transport: error while dialing: dial tcp: address orderer.nmplus.com: missing port in address". Reconnecting...
2019-09-24 19:12:01.073 UTC [grpc] HandleSubConnStateChange -> DEBU 049 pickfirstBalancer: HandleSubConnStateChange: 0xc000267e80, TRANSIENT_FAILURE
Error: error getting broadcast client: orderer client failed to connect to orderer.nmplus.com: failed to create new connection: connection error: desc = "transport: error while dialing: dial tcp: address orderer.nmplus.com: missing port in address"
------------------------------------------------------------

TLS 적용을 위해,
orderer, ca, peer 설정에 TLS 추가
HLF network  기동은 되지만, 
create channel에서 에러, 그렇기 때문에 join 또한 에러.

------------------------------------------------------------
2019-09-24 23:54:43.678 UTC [grpc] HandleSubConnStateChange -> DEBU 055 pickfirstBalancer: HandleSubConnStateChange: 0xc000310c30, READY
2019-09-24 23:54:43.679 UTC [grpc] HandleSubConnStateChange -> DEBU 056 pickfirstBalancer: HandleSubConnStateChange: 0xc000310c30, TRANSIENT_FAILURE
2019-09-24 23:54:43.679 UTC [grpc] infof -> DEBU 057 transport: loopyWriter.run returning. connection error: desc = "transport is closing"
2019-09-24 23:54:43.679 UTC [grpc] HandleSubConnStateChange -> DEBU 058 pickfirstBalancer: HandleSubConnStateChange: 0xc000310c30, CONNECTING
Error: rpc error: code = Unavailable desc = transport is closing
------------------------------------------------------------

create 및 join command를 확인 해 보는 게 좋을듯.

명령어 변경
------------------------------------------------------------
peer channel create -o orderer.nmplus.com:7050 -c mychannel -f ./channel-artifacts/channel.tx \
--tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/nmplus.com/orderers/orderer.nmplus.com/msp/tlscacerts/tlsca.nmplus.com-cert.pem
------------------------------------------------------------

오류 발생.
혹시 각각 인증서 선언을 잘못한 것은 아닌지 확인 필요
------------------------------------------------------------
2019-09-25 00:10:59.920 UTC [grpc] watcher -> DEBU 039 ccResolverWrapper: sending new addresses to cc: [{orderer.nmplus.com:7050 0  <nil>}]
2019-09-25 00:10:59.921 UTC [grpc] switchBalancer -> DEBU 03a ClientConn switching balancer to "pick_first"
2019-09-25 00:10:59.921 UTC [grpc] HandleSubConnStateChange -> DEBU 03b pickfirstBalancer: HandleSubConnStateChange: 0xc00024d6b0, CONNECTING
2019-09-25 00:10:59.927 UTC [grpc] createTransport -> DEBU 03c grpc: addrConn.createTransport failed to connect to {orderer.nmplus.com:7050 0  <nil>}. Err :connection error: desc = "transport: authentication handshake failed: remote error: tls: bad certificate". Reconnecting...
2019-09-25 00:10:59.927 UTC [grpc] HandleSubConnStateChange -> DEBU 03d pickfirstBalancer: HandleSubConnStateChange: 0xc00024d6b0, TRANSIENT_FAILURE
Error: failed to create deliver client: orderer client failed to connect to orderer.nmplus.com:7050: failed to create new connection: connection error: desc = "transport: authentication handshake failed: remote error: tls: bad certificate"
------------------------------------------------------------

관련 된 것을 해도 해결 불가.
hyperledger-study02 설정과 모두 비교 결과.

------------------------------------------------------------
      - CORE_PEER_GOSSIP_USELEADERELECTION=true
      - CORE_PEER_GOSSIP_ORGLEADER=false

      - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
------------------------------------------------------------
추가 후 에러 미발생.
SKIPHANDSHAKE 항목이 중요 설정인 것으로 예상 됨.


REST API에서 호출 시 응답 속도가 너무 느림.
orderer 설정을 solo에서 kafka를 이용하여, ordering service 하도록 결정.
configtx.yaml 변경.
------------------------------------------------------------
Orderer: &OrdererDefaults

    # Orderer Type: The orderer implementation to start.
    # Available types are "solo" and "kafka".
    # OrdererType: solo
    OrdererType: kafka

   ... 중략 ...

    Kafka:
      # Brokers: A list of Kafka brokers to which the orderer connects
      # NOTE: Use IP:port notation
      Brokers:
        - kafka0:9092
        - kafka1:9092
        - kafka2:9092
        - kafka3:9092
------------------------------------------------------------


kafka를 관리하기 위한 zookeeper가 필요하다. 
zookeeper 설정.
3개의 서버를 구성해 준다.
------------------------------------------------------------
  zk1:
    image: hyperledger/fabric-zookeeper
    environment:
      - ZOO_MY_ID=1
      - ZOO_SERVERS=server.1=zk1:2888:3888 server.2=zk2:2888:3888 server.3=zk3:2888:3888
      - ZOO_TICK_TIME=2000
      - ZOO_INIT_LIMIT=10
      - ZOO_SYNC_LIMIT=5
------------------------------------------------------------

kafka를 설정해준다.
kafka는 구성원 중 1개가 서버 다운되었을 때, 짝수로 구성되면 ordering이 제대로 되지 않으므로,
꼭 짝수개로 구성한다. 
4개를 설정해 준다.
------------------------------------------------------------
  kafka0:
    image: hyperledger/fabric-kafka
    environment:
      - KAFKA_BROKER_ID=0
      - KAFKA_MIN_INSYNC_REPLICAS=2
      - KAFKA_MESSAGE_MAX_BYTES=50000000
      - KAFKA_REPLICA_FETCH_MAX_BYTES=50000000
      - KAFKA_PRODUCER_MAX_REQUEST_SIZE=50000000
      - KAFKA_CONSUMER_MAX_PARTITION_FETCH_BYTES=50000000
      - CONNECT_PRODUCER_MAX_REQUEST_SIZE=50000000
      - CONNECT_CONSUMER_MAX_PARTITION_FETCH_BYTES=50000000
      - KAFKA_DEFAULT_REPLICATION_FACTOR=3
      - KAFKA_ZOOKEEPER_CONNECT=zk1:2181,zk2:2181,zk3:2181
------------------------------------------------------------

configtx.yaml을 변경했으므로, 
configtxgen.sh을 꼭 실행시켜야 한다.
































==========================================================================
node js

인증서를 재생성 했으므로, 다시금 복사 해 줘야 함.
organization이 1개인 것으로 변경되었으므로, 
org2는 삭제. 대신 peer를 9까지 생성.

network-config-ldt_network.yaml 수정

서버 실행 결과 정상 실행.


ldt saveUser api 테스트

service 단 수정 필요한 것으로 예상.
------------------------------------------------------------
[2019-09-25 01:50:37.051] [DEBUG] Transaction - request:{"targets":["peer0.org1.nmplus.com","peer1.org1.nmplus.com","peer0.org2.nmplus.com","peer1.org2.nmplus.com"],"chaincodeId":"balance","fcn":"invoke","args":["testuser","username"],"chainId":"mychannel","txId":{"_nonce":{"type":"Buffer","data":[48,158,240,214,64,146,163,191,155,4,116,224,246,29,65,247,90,67,248,110,245,80,15,13]},"_transaction_id":"747e5abb01cd5d50cbce01091b01fc90d677add91bce32e40b30f582bfe73939","_admin":false}}
[2019-09-25 01:50:37.142] [ERROR] Transaction - Error: Peer with name "peer0.org2.nmplus.com" not assigned to this channel
    at Channel._getTargets (/home/fabric/go/src/github.com/ldt-hf/ldt-client/node_modules/fabric-client/lib/Channel.js:3515:13)
    at Channel.sendTransactionProposal (/home/fabric/go/src/github.com/ldt-hf/ldt-client/node_modules/fabric-client/lib/Channel.js:2791:26)
    at TrxClient.sendTransactionProposal [as invokeChainCode] (/home/fabric/go/src/github.com/ldt-hf/ldt-client/build/webpack:/server/blockchain/transaction/transaction.js:76:37)
    at <anonymous>
[2019-09-25 01:50:37.142] [ERROR] Transaction - Failed to invoke chaincode. cause:Error: Peer with name "peer0.org2.nmplus.com" not assigned to this channel
(node:12340) UnhandledPromiseRejectionWarning: Error: Failed to invoke chaincode. cause:Error: Peer with name "peer0.org2.nmplus.com" not assigned to this channel
    at TrxClient.invokeChainCode (/home/fabric/go/src/github.com/ldt-hf/ldt-client/build/webpack:/server/blockchain/transaction/transaction.js:204:11)
    at <anonymous>
(node:12340) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). (rejection id: 3)
(node:12340) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
------------------------------------------------------------


수정 후 테스트

------------------------------------------------------------
E0925 01:54:14.217154437   12419 ssl_transport_security.cc:1238] Handshake failed with fatal error SSL_ERROR_SSL: error:1408F10B:SSL routines:SSL3_GET_RECORD:wrong version number.
2019-09-24T16:54:14.337Z - error: [Remote.js]: Error: Failed to connect before the deadline URL:grpcs://localhost:9051
2019-09-24T16:54:14.340Z - error: [Remote.js]: Error: Failed to connect before the deadline URL:grpcs://localhost:8051
2019-09-24T16:54:14.341Z - error: [Remote.js]: Error: Failed to connect before the deadline URL:grpcs://localhost:7051
2019-09-24T16:54:14.341Z - error: [Remote.js]: Error: Failed to connect before the deadline URL:grpcs://localhost:12051
2019-09-24T16:54:14.342Z - error: [Remote.js]: Error: Failed to connect before the deadline URL:grpcs://localhost:11051
2019-09-24T16:54:14.342Z - error: [Remote.js]: Error: Failed to connect before the deadline URL:grpcs://localhost:13051
2019-09-24T16:54:14.342Z - error: [Remote.js]: Error: Failed to connect before the deadline URL:grpcs://localhost:16051
2019-09-24T16:54:14.343Z - error: [Remote.js]: Error: Failed to connect before the deadline URL:grpcs://localhost:15051
2019-09-24T16:54:14.343Z - error: [Remote.js]: Error: Failed to connect before the deadline URL:grpcs://localhost:14051
2019-09-24T16:54:14.343Z - error: [Remote.js]: Error: Failed to connect before the deadline URL:grpcs://localhost:10051
[2019-09-25 01:54:14.455] [ERROR] Transaction - TypeError: Cannot read property 'status' of undefined
    at TrxClient.status [as invokeChainCode] (/home/fabric/go/src/github.com/ldt-hf/ldt-client/build/webpack:/server/blockchain/transaction/transaction.js:91:41)
    at <anonymous>
    at process._tickCallback (internal/process/next_tick.js:189:7)
[2019-09-25 01:54:14.455] [ERROR] Transaction - Failed to invoke chaincode. cause:TypeError: Cannot read property 'status' of undefined
(node:12419) UnhandledPromiseRejectionWarning: Error: Failed to invoke chaincode. cause:TypeError: Cannot read property 'status' of undefined
    at TrxClient.invokeChainCode (/home/fabric/go/src/github.com/ldt-hf/ldt-client/build/webpack:/server/blockchain/transaction/transaction.js:204:11)
    at <anonymous>
    at process._tickCallback (internal/process/next_tick.js:189:7)
(node:12419) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). (rejection id: 1)
(node:12419) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
E0925 01:54:16.134850349   12419 ssl_transport_security.cc:1238] Handshake failed with fatal error SSL_ERROR_SSL: error:1408F10B:SSL routines:SSL3_GET_RECORD:wrong version number.
------------------------------------------------------------
오류 발생.

그럼 기존 sample api들은 되는 것인가? 
테스트.
sample도 에러 발생.
TLS 셋팅이 꼭 필요한 것은 아닌가 확인 필요.

 
------------------------------------------------------------
E0925 01:54:16.134850349   12419 ssl_transport_security.cc:1238] Handshake failed with fatal error SSL_ERROR_SSL: error:1408F10B:SSL routines:SSL3_GET_RECORD:wrong version number.
------------------------------------------------------------
에 대해 고민 해 본 결과,
version이 chaincode 등의 버젼이 아닌, TLS의 버젼을 뜻하는 것이 아닌가 예상.
HLF network에 TLS 적용.


api 호출 시 에러 발생.
에러 확인 결과 api에서 호출 시 인증서 정보가 맞지 않음.
------------------------------------------------------------
2019-09-25 15:19:19.724] [DEBUG] HLF - [NetworkConfig101.js]: _addPeersToChannel - peer9.org1.nmplus.com - grpcs://localhost:16051
[2019-09-25 15:19:19.725] [DEBUG] HLF - [NetworkConfig101.js]: getOrderer - name orderer.nmplus.com
[2019-09-25 15:19:19.731] [DEBUG] Transaction - request:{"targets":null,"chaincodeId":"balance","fcn":"saveUser","args":["123","testuser","20","male"],"chainId":"mychannel","txId":{"_nonce":{"type":"Buffer","data":[20,138,252,193,230,240,94,131,195,239,43,195,0,237,83,185,21,60,123,95,161,17,131,137]},"_transaction_id":"2cb42bd71c37647672e8d3245fff5603db23de4af750a272e84cab9213981dfa","_admin":false}}
[2019-09-25 15:19:19.758] [DEBUG] HLF - [crypto_ecdsa_aes]: ecdsa signature:  Signature {
  r: <BN: 9140cdee89ae1a23c47637ce6b14979226f015e259ab82f0e7168c3593651218>,
  s: <BN: 16fcdf0dc806422fcc5e4450db834e82f9fa948b9e669a46b3c16a7fc5b8a95>,
  recoveryParam: 1 }
[2019-09-25 15:19:19.862] [ERROR] Transaction - TypeError: Cannot read property 'status' of undefined
    at TrxClient.status [as invokeChainCode] (/home/fabric/go/src/github.com/ldt-hf/ldt-client/build/webpack:/server/blockchain/transaction/transaction.js:91:41)
(node:22396) UnhandledPromiseRejectionWarning: Error: Failed to invoke chaincode. cause:TypeError: Cannot read property 'status' of undefined
    at TrxClient.invokeChainCode (/home/fabric/go/src/github.com/ldt-hf/ldt-client/build/webpack:/server/blockchain/transaction/transaction.js:204:11)
    at <anonymous>
(node:22396) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). (rejection id: 1)
(node:22396) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
    at <anonymous>
[2019-09-25 15:19:19.862] [ERROR] Transaction - Failed to invoke chaincode. cause:TypeError: Cannot read property 'status' of undefined
------------------------------------------------------------
검색으로 확인 불가.
server 기동 시 kv 폴더에 인증서 정보를 복사 하는 듯.
인증서가 존재하면, 인증서를 복사하지 않는 것으로 예상. 
기존 폴더를 삭제 후 서버 기동.  인증서가 제대로 등록되는 것으로 파악 됨.


controller saveTrade수정.
------------------------------------------------------------
  saveTrade(req, res) {
    LdtService
      .saveTrade(req, res)
      .then(r => res
         .status(201)
         .location(`/api/v1/ldt/saveTrade`)
         .json(r));
  }
------------------------------------------------------------

routes에 saveTrade 추가

ldt.service.js에 saveTrade 추가
------------------------------------------------------------
  saveTrade(req, res) {
    l.info(`${req.body.id}`);
    l.info(`${req.body.name}`);

    let args = [];
    let peers = [];
    args.push(req.body.id);
    args.push(req.body.date);
    args.push(req.body.data);

    peers.push('peer0.org1.nmplus.com');
    peers.push('peer1.org1.nmplus.com');
    peers.push('peer2.org1.nmplus.com');
    peers.push('peer3.org1.nmplus.com');
    peers.push('peer4.org1.nmplus.com');
    peers.push('peer5.org1.nmplus.com');
    peers.push('peer6.org1.nmplus.com');
    peers.push('peer7.org1.nmplus.com');
    peers.push('peer8.org1.nmplus.com');
    peers.push('peer9.org1.nmplus.com');

//    return db.createUser(args);
    return Promise.resolve(transaction.invokeChainCode(peers, 'mychannel', 'ldt', 'saveTrade', args, 'admin', 'org1'));
  }
------------------------------------------------------------

swagger/Api.yaml에 해당 URL 추가
------------------------------------------------------------
  /ldt/saveTrade:
    post:
      tags:
        - ldt
      description: ldt save trade info
      parameters:
        - name: id
          in: body
          required: true
          schema:
            $ref: "#/ldtDefs/TradeBody"
      response:
        200:
          description: return thhh...
------------------------------------------------------------



테스트 결과 
swagger에서 호출하게 되면 오류 발생.
------------------------------------------------------------
Could not render this component
------------------------------------------------------------

검색으로도 원인 및 해결이 나오지 않음.
예상으로, example의 문자열 설정에서 에러가 발생하는 것은 아닌가 하여,
해당 설정을 단순 문자들만 입력.
------------------------------------------------------------
ldtDefs:
  ExampleBody:
    type: object
    title: example
    required:
      - id
      - name
    properties:
      id:
        type: string
        example: testuser
      name:
        type: string
        example: username
  TradeBody:
    type: object
    title: saveTrade
    required:
      - id
      - date
      - data
    properties:
      id:
        type: string
        example: testuser
      date:
        type: string
        example: "20001231175011"
      data:
        type: string
        # example: "{\"productSeq\":\"2\", \"count\": \"2\"}"
        example: eveveevev
------------------------------------------------------------
결과, 에러가 발생하지 않음.


log 확인 결과, 문자 깨짐이 확인 됨.
어디에서 문자 깨짐이 생기는 것인지 확인 필요.
------------------------------------------------------------
[2019-09-27 13:11:58.901] [DEBUG] Transaction - request:{"targets":["peer0.org1.nmplus.com","peer1.org1.nmplus.com","peer2.org1.nmplus.com","peer3.org1.nmplus.com","peer4.org1.nmplus.com","peer5.org1.nmplus.com","peer6.org1.nmplus.com","peer7.org1.nmplus.com","peer8.org1.nmplus.com","peer9.org1.nmplus.com"],"chaincodeId":"ldt","fcn":"saveTrade","args":["testuser","20001231175011","eveveevev"],"chainId":"mychannel","txId":{"_nonce":{"type":"Buffer","data":[56,19,170,240,34,232,64,57,178,226,73,152,57,220,229,217,58,161,111,128,144,15,250,87]},"_transaction_id":"e9ce38b1b2591e675ad174d37516e12a611dee71d63e52e0667189f81b9c12f0","_admin":false}}
[2019-09-27 13:11:58.938] [DEBUG] HLF - [crypto_ecdsa_aes]: ecdsa signature:  Signature {
  r: <BN: f2b89b9c459f0174b75168984b66f4598601ee849f780435fd57900fcf449a80>,
  s: <BN: 5038eb192d1aa26ae43c118b89c0af79fef187d45e3319c0622ea0e063766556>,
  recoveryParam: 0 }
[2019-09-27 13:11:59.133] [DEBUG] Transaction - B.S.SEO : 0E!占퐕6占占쏙옙13占퐄A占占占쏙옙W9占쏙옙r占占쏙옙98 RX占#占쏙옙~占쏙옙P占폥占쏙옙R占占占쏙옙占쏙옙(U占bF
[2019-09-27 13:11:59.133] [INFO] Transaction - AAAA sent Proposal and received ProposalResponse: Status - 200, message - "", metadata - "add succeed!!!", endorsement signature: 0E!占퐕6占占쏙옙13占퐄A占占占쏙옙W9占쏙옙r占占쏙옙98 RX占#占쏙옙~占쏙옙P占폥占쏙옙R占占占쏙옙占쏙옙(U占bF
[2019-09-27 13:11:59.133] [INFO] Transaction - invoke chaincode proposal was good
[2019-09-27 13:11:59.133] [INFO] Transaction - AAAA sent Proposal and received ProposalResponse: Status - 200, message - "", metadata - "add succeed!!!", endorsement signature: 0E!占퐕6占占쏙옙13占퐄A占占占쏙옙W9占쏙옙r占占쏙옙98 RX占#占쏙옙~占쏙옙P占폥占쏙옙R占占占쏙옙占쏙옙(U占bF
[2019-09-27 13:11:59.133] [INFO] Transaction - invoke chaincode proposal was good
[2019-09-27 13:11:59.133] [INFO] Transaction - AAAA sent Proposal and received ProposalResponse: Status - 200, message - "", metadata - "add succeed!!!", endorsement signature: 0E!占퐕6占占쏙옙13占퐄A占占占쏙옙W9占쏙옙r占占쏙옙98 RX占#占쏙옙~占쏙옙P占폥占쏙옙R占占占쏙옙占쏙옙(U占bF
[2019-09-27 13:11:59.134] [INFO] Transaction - invoke chaincode proposal was good
------------------------------------------------------------

확인 결과, 
transaction 이후, 결과 값 log에서 발생.



invoke 시 에러.
여러번 테스트 한 결과, 
매번 발생하는 것이 아닌, 서버를 새로 시작하고,
instantiate 되었을 때, 최초 발생하는 것으로 예상.
------------------------------------------------------------
[2019-09-29 14:58:00.102] [DEBUG] HLF - [NetworkConfig101.js]: getOrderer - name orderer.nmplus.com
[2019-09-29 14:58:00.108] [DEBUG] Transaction - request:{"targets":["peer0.org1.nmplus.com","peer1.org1.nmplus.com","peer2.org1.nmplus.com","peer3.org1.nmplus.com","peer4.org1.nmplus.com","peer5.org1.nmplus.com","peer6.org1.nmplus.com","peer7.org1.nmplus.com","peer8.org1.nmplus.com","peer9.org1.nmplus.com"],"chaincodeId":"ldt","fcn":"saveTrade","args":["testuser","20001231175011","eveveevev"],"chainId":"mychannel","txId":{"_nonce":{"type":"Buffer","data":[182,201,149,251,196,116,76,51,196,244,194,17,192,37,200,192,111,152,114,211,113,8,66,83]},"_transaction_id":"358a21e9a32d7c2818f89b88df454605727c852841ebcc8853bc45ea5c622684","_admin":false}}
[2019-09-29 14:58:00.141] [DEBUG] HLF - [crypto_ecdsa_aes]: ecdsa signature:  Signature {
  r: <BN: f325e35e6390183a05bf881c0d42c39a8819af5440183b642d605f19ff5da934>,
  s: <BN: 4e6055dfc51afc36af4f0dc47da81a1c4dbc91a8b81b0140dbc50e4d812a51d4>,
  recoveryParam: 0 }
2019-09-29T05:58:45.275Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-09-29T05:58:45.279Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-09-29T05:58:45.279Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-09-29T05:58:45.280Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-09-29T05:58:45.280Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-09-29T05:58:45.281Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-09-29T05:58:45.283Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-09-29T05:58:45.283Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-09-29T05:58:45.286Z - error: [Peer.js]: sendProposal - timed out after:45000
[2019-09-29 14:58:45.286] [INFO] Transaction - invoke chaincode proposal was good
[2019-09-29 14:58:45.287] [ERROR] Transaction - [{"version":1,"timestamp":null,"response":{"status":200,"message":"","payload":{"type":"Buffer","data":[97,100,100,32,115,117,99,99,101,101,100,33,33,33]}},"payload":{"type":"Buffer","data":[10,32,16,146,102,242,223,214,227,202,72,36,112,61,41,94,231,172,102,2,240,30,62,28,153,123,184,143,88,48,22,201,197,115,18,98,10,63,18,40,10,3,108,100,116,18,33,26,31,10,8,116,101,115,116,117,115,101,114,26,19,123,34,100,111,99,84,121,112,101,34,58,34,84,114,97,100,101,34,125,18,19,10,4,108,115,99,99,18,11,10,9,10,3,108,100,116,18,2,8,1,26,19,8,200,1,26,14,97,100,100,32,115,117,99,99,101,101,100,33,33,33,34,10,18,3,108,100,116,26,3,48,46,49]},"endorsement":{"endorser":{"type":"Buffer","data":[10,7,79,114,103,49,77,83,80,18,146,6,45,45,45,45,45,66,69,71,73,78,32,67,69,82,84,73,70,73,67,65,84,69,45,45,45,45,45,10,77,73,73,67,70,122,67,67,65,98,50,103,65,119,73,66,65,103,73,82,65,75,113,71,87,50,69,55,111,106,72,104,66,88,74,79,72,66,70,102,85,100,69,119,67,103,89,73,75,111,90,73,122,106,48,69,65,119,73,119,99,84,69,76,10,77,65,107,71,65,49,85,69,66,104,77,67,86,86,77,120,69,122,65,82,66,103,78,86,66,65,103,84,67,107,78,104,98,71,108,109,98,51,74,117,97,87,69,120,70,106,65,85,66,103,78,86,66,65,99,84,68,86,78,104,98,105,66,71,10,99,109,70,117,89,50,108,122,89,50,56,120,71,68,65,87,66,103,78,86,66,65,111,84,68,50,57,121,90,122,69,117,98,109,49,119,98,72,86,122,76,109,78,118,98,84,69,98,77,66,107,71,65,49,85,69,65,120,77,83,89,50,69,117,10,98,51,74,110,77,83,53,117,98,88,66,115,100,88,77,117,89,50,57,116,77,66,52,88,68,84,69,53,77,68,107,121,78,84,65,119,77,68,73,119,77,70,111,88,68,84,73,53,77,68,107,121,77,106,65,119,77,68,73,119,77,70,111,119,10,87,106,69,76,77,65,107,71,65,49,85,69,66,104,77,67,86,86,77,120,69,122,65,82,66,103,78,86,66,65,103,84,67,107,78,104,98,71,108,109,98,51,74,117,97,87,69,120,70,106,65,85,66,103,78,86,66,65,99,84,68,86,78,104,10,98,105,66,71,99,109,70,117,89,50,108,122,89,50,56,120,72,106,65,99,66,103,78,86,66,65,77,84,70,88,66,108,90,88,73,119,76,109,57,121,90,122,69,117,98,109,49,119,98,72,86,122,76,109,78,118,98,84,66,90,77,66,77,71,10,66,121,113,71,83,77,52,57,65,103,69,71,67,67,113,71,83,77,52,57,65,119,69,72,65,48,73,65,66,74,81,115,50,119,79,84,88,119,116,52,117,47,122,70,72,102,88,102,79,81,88,79,85,67,108,48,49,119,52,72,72,74,86,107,10,75,82,56,74,107,69,90,90,49,49,89,111,66,114,76,117,80,121,82,52,52,88,73,118,100,110,72,56,74,69,99,83,101,55,100,83,114,121,54,56,100,76,122,48,72,119,88,74,98,114,87,106,84,84,66,76,77,65,52,71,65,49,85,100,10,68,119,69,66,47,119,81,69,65,119,73,72,103,68,65,77,66,103,78,86,72,82,77,66,65,102,56,69,65,106,65,65,77,67,115,71,65,49,85,100,73,119,81,107,77,67,75,65,73,80,47,104,110,122,65,54,69,74,99,85,65,67,111,107,10,117,116,110,110,102,50,121,99,67,78,85,75,74,117,76,53,107,83,76,88,89,43,71,69,100,110,117,67,77,65,111,71,67,67,113,71,83,77,52,57,66,65,77,67,65,48,103,65,77,69,85,67,73,81,68,119,75,56,98,122,90,81,78,72,10,50,117,83,48,73,78,54,78,82,98,70,49,73,106,54,112,49,121,122,56,76,80,80,73,77,72,108,74,54,90,115,99,49,103,73,103,83,47,106,74,67,78,81,74,111,118,108,68,48,81,104,90,105,110,116,112,83,74,113,105,116,78,98,112,10,116,72,55,120,114,54,68,84,104,105,116,90,103,112,77,61,10,45,45,45,45,45,69,78,68,32,67,69,82,84,73,70,73,67,65,84,69,45,45,45,45,45,10]},"signature":{"type":"Buffer","data":[48,69,2,33,0,222,19,253,159,205,191,206,153,11,194,127,107,106,81,85,254,91,71,0,146,173,30,24,163,73,89,235,56,245,62,245,25,2,32,121,61,239,56,75,71,95,213,159,133,249,137,250,41,122,177,3,129,222,216,220,224,5,186,73,168,113,29,70,170,189,86]}},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}}},{},{},{},{},{},{},{},{},{}]
[2019-09-29 14:58:45.287] [ERROR] Transaction - invoke chaincode proposal was bad
[2019-09-29 14:58:45.288] [ERROR] Transaction - [{"version":1,"timestamp":null,"response":{"status":200,"message":"","payload":{"type":"Buffer","data":[97,100,100,32,115,117,99,99,101,101,100,33,33,33]}},"payload":{"type":"Buffer","data":[10,32,16,146,102,242,223,214,227,202,72,36,112,61,41,94,231,172,102,2,240,30,62,28,153,123,184,143,88,48,22,201,197,115,18,98,10,63,18,40,10,3,108,100,116,18,33,26,31,10,8,116,101,115,116,117,115,101,114,26,19,123,34,100,111,99,84,121,112,101,34,58,34,84,114,97,100,101,34,125,18,19,10,4,108,115,99,99,18,11,10,9,10,3,108,100,116,18,2,8,1,26,19,8,200,1,26,14,97,100,100,32,115,117,99,99,101,101,100,33,33,33,34,10,18,3,108,100,116,26,3,48,46,49]},"endorsement":{"endorser":{"type":"Buffer","data":[10,7,79,114,103,49,77,83,80,18,146,6,45,45,45,45,45,66,69,71,73,78,32,67,69,82,84,73,70,73,67,65,84,69,45,45,45,45,45,10,77,73,73,67,70,122,67,67,65,98,50,103,65,119,73,66,65,103,73,82,65,75,113,71,87,50,69,55,111,106,72,104,66,88,74,79,72,66,70,102,85,100,69,119,67,103,89,73,75,111,90,73,122,106,48,69,65,119,73,119,99,84,69,76,10,77,65,107,71,65,49,85,69,66,104,77,67,86,86,77,120,69,122,65,82,66,103,78,86,66,65,103,84,67,107,78,104,98,71,108,109,98,51,74,117,97,87,69,120,70,106,65,85,66,103,78,86,66,65,99,84,68,86,78,104,98,105,66,71,10,99,109,70,117,89,50,108,122,89,50,56,120,71,68,65,87,66,103,78,86,66,65,111,84,68,50,57,121,90,122,69,117,98,109,49,119,98,72,86,122,76,109,78,118,98,84,69,98,77,66,107,71,65,49,85,69,65,120,77,83,89,50,69,117,10,98,51,74,110,77,83,53,117,98,88,66,115,100,88,77,117,89,50,57,116,77,66,52,88,68,84,69,53,77,68,107,121,78,84,65,119,77,68,73,119,77,70,111,88,68,84,73,53,77,68,107,121,77,106,65,119,77,68,73,119,77,70,111,119,10,87,106,69,76,77,65,107,71,65,49,85,69,66,104,77,67,86,86,77,120,69,122,65,82,66,103,78,86,66,65,103,84,67,107,78,104,98,71,108,109,98,51,74,117,97,87,69,120,70,106,65,85,66,103,78,86,66,65,99,84,68,86,78,104,10,98,105,66,71,99,109,70,117,89,50,108,122,89,50,56,120,72,106,65,99,66,103,78,86,66,65,77,84,70,88,66,108,90,88,73,119,76,109,57,121,90,122,69,117,98,109,49,119,98,72,86,122,76,109,78,118,98,84,66,90,77,66,77,71,10,66,121,113,71,83,77,52,57,65,103,69,71,67,67,113,71,83,77,52,57,65,119,69,72,65,48,73,65,66,74,81,115,50,119,79,84,88,119,116,52,117,47,122,70,72,102,88,102,79,81,88,79,85,67,108,48,49,119,52,72,72,74,86,107,10,75,82,56,74,107,69,90,90,49,49,89,111,66,114,76,117,80,121,82,52,52,88,73,118,100,110,72,56,74,69,99,83,101,55,100,83,114,121,54,56,100,76,122,48,72,119,88,74,98,114,87,106,84,84,66,76,77,65,52,71,65,49,85,100,10,68,119,69,66,47,119,81,69,65,119,73,72,103,68,65,77,66,103,78,86,72,82,77,66,65,102,56,69,65,106,65,65,77,67,115,71,65,49,85,100,73,119,81,107,77,67,75,65,73,80,47,104,110,122,65,54,69,74,99,85,65,67,111,107,10,117,116,110,110,102,50,121,99,67,78,85,75,74,117,76,53,107,83,76,88,89,43,71,69,100,110,117,67,77,65,111,71,67,67,113,71,83,77,52,57,66,65,77,67,65,48,103,65,77,69,85,67,73,81,68,119,75,56,98,122,90,81,78,72,10,50,117,83,48,73,78,54,78,82,98,70,49,73,106,54,112,49,121,122,56,76,80,80,73,77,72,108,74,54,90,115,99,49,103,73,103,83,47,106,74,67,78,81,74,111,118,108,68,48,81,104,90,105,110,116,112,83,74,113,105,116,78,98,112,10,116,72,55,120,114,54,68,84,104,105,116,90,103,112,77,61,10,45,45,45,45,45,69,78,68,32,67,69,82,84,73,70,73,67,65,84,69,45,45,45,45,45,10]},"signature":{"type":"Buffer","data":[48,69,2,33,0,222,19,253,159,205,191,206,153,11,194,127,107,106,81,85,254,91,71,0,146,173,30,24,163,73,89,235,56,245,62,245,25,2,32,121,61,239,56,75,71,95,213,159,133,249,137,250,41,122,177,3,129,222,216,220,224,5,186,73,168,113,29,70,170,189,86]}},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}}},{},{},{},{},{},{},{},{},{}]
[2019-09-29 14:58:45.288] [ERROR] Transaction - invoke chaincode proposal was bad
------------------------------------------------------------
init 메소드에서 발생하는 것으로 예상.

init을 주석 처리한 결과, 에러는 그대로 발생. 
그 문제는 아닌 것으로 판단 됨.

// TODO
init은 최초 API 호출시 실행되는 것이 아닌, 채널 생성 시 에러가 발생.
init 보다는 이후 해당 펑션에 대해, 초기 값이 없는 등에 의해 에러가 발생하는 것이 아닌가 예상.


호출 시 로그를 확인 해 본 결과, 
호출 시 응답 시간이 약 3초 걸림.
응답이 너무 느리므로, 그에 대한 조치가 필요할 것으로 판단됨.


데이터 조회를 위한 query api 추가.
호출 url 설정.
------------------------------------------------------------
fabric@localhost:~/go/src/github.com/ldt-hf/ldt-client/server/api/controllers/ldt: 
$ vi router.js 
import * as express from 'express';
import controller from './controller';

export default express
  .Router()
  .get('/', controller.all)
  .get('/', controller.byId)
  .post('/user', controller.createUser)
  .post('/saveTrade', controller.saveTrade);
------------------------------------------------------------

controller api 호출 추가
------------------------------------------------------------
fabric@localhost:~/go/src/github.com/ldt-hf/ldt-client/server/api/controllers/ldt: 
$ vi controller.js 
import LdtService from '../../services/ldt/ldt.service';

export class Controller {
  createUser(req, res) {
    LdtService
      .createUser(req, res)
      .then(r => res
        .status(201)
        .location(`/api/v1/ldt/user`)
        .json(r));
  }

  all(req, res) {
    LdtService
      .all()
      .then(r => res.json(r));
  }

  byId(req, res) {
    LdtService
      .byId(req.params.id)
      .then(r => {
              if (r) res.json(r);
              else res.status(404).end();
            }
       );
  }

   ... 중략 ...
------------------------------------------------------------

그에 따라, service 단 로직 추가.
------------------------------------------------------------
fabric@localhost:~/go/src/github.com/ldt-hf/ldt-client/server/api/services/ldt: 
$ vi ldt.service.js 
import l from '../../../common/logger';
// import db from './ldt.db.service';

const transaction = require('../../../blockchain/transaction/transaction');

class LdtService {

  ... 중략 ...

  all() {
    const args = [];

    // args.push(id);
    return Promise.resolve(transaction.queryChainCode(null, 'mychannel', 'ldt',
      null, 'query', 'admin', 'org1'));
  }

  byId(id) {
    const args = [];

    args.push(id);
    return Promise.resolve(transaction.queryChainCode(null, 'mychannel', 'ldt',
      args, 'query', 'admin', 'org1'));
  }

  ... 중략 ...

------------------------------------------------------------


테스트 결과, URL 호출이 안 되고 있음.
서버 컴파일 및 재기동 이후 정상 작동 확인

최초 HLF network 호출 시 에러 발생.
------------------------------------------------------------
2019-10-03T06:11:45.485Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-03T06:11:45.485Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-03T06:11:45.485Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-03T06:11:45.486Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-03T06:11:45.486Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-03T06:11:45.488Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-03T06:11:45.490Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-03T06:11:45.490Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-03T06:11:45.491Z - error: [Peer.js]: sendProposal - timed out after:45000
[2019-10-03 15:11:45.492] [INFO] Transaction - invoke chaincode proposal was good
[2019-10-03 15:11:45.493] [ERROR] Transaction - [{"version":1,"timestamp":null,"response":{"status":200,"message":"","payload":{"type":"Buffer","data":[97,100,100,32,115,117,99,99,101,101,100,33,33,33]}},"payload":{"type":"Buffer","data":[10,32,229,219,217,87,175,71,76,231,196,154,103,38,146,241,170,132,187,222,139,100,32,230,219,34,79,60,15,27,37,183,224,23,18,154,1,10,119,18,96,10,3,108,100,116,18,89,10,14,10,8,116,101,115,116,117,115,101,114,18,2,8,1,26,71,10,8,116,101,115,116,117,115,101,114,26,59,91,123,34,100,97,116,101,34,58,34,50,48,48,48,49,50,51,49,49,55,53,48,49,52,34,44,34,112,114,111,100,117,99,116,83,101,113,34,58,34,49,50,34,44,34,97,109,111,117,110,116,34,58,34,50,48,34,125,93,18,19,10,4,108,115,99,99,18,11,10,9,10,3,108,100,116,18,2,8,1,26,19,8,200,1,26,14,97,100,100,32,115,117,99,99,101,101,100,33,33,33,34,10,18,3,108,100,116,26,3,48,46,49]},"endorsement":{"endorser":{"type":"Buffer","data":[10,7,79,114,103,49,77,83,80,18,146,6,45,45,45,45,45,66,69,71,73,78,32,67,69,82,84,73,70,73,67,65,84,69,45,45,45,45,45,10,77,73,73,67,70,122,67,67,65,98,50,103,65,119,73,66,65,103,73,82,65,75,113,71,87,50,69,55,111,106,72,104,66,88,74,79,72,66,70,102,85,100,69,119,67,103,89,73,75,111,90,73,122,106,48,69,65,119,73,119,99,84,69,76,10,77,65,107,71,65,49,85,69,66,104,77,67,86,86,77,120,69,122,65,82,66,103,78,86,66,65,103,84,67,107,78,104,98,71,108,109,98,51,74,117,97,87,69,120,70,106,65,85,66,103,78,86,66,65,99,84,68,86,78,104,98,105,66,71,10,99,109,70,117,89,50,108,122,89,50,56,120,71,68,65,87,66,103,78,86,66,65,111,84,68,50,57,121,90,122,69,117,98,109,49,119,98,72,86,122,76,109,78,118,98,84,69,98,77,66,107,71,65,49,85,69,65,120,77,83,89,50,69,117,10,98,51,74,110,77,83,53,117,98,88,66,115,100,88,77,117,89,50,57,116,77,66,52,88,68,84,69,53,77,68,107,121,78,84,65,119,77,68,73,119,77,70,111,88,68,84,73,53,77,68,107,121,77,106,65,119,77,68,73,119,77,70,111,119,10,87,106,69,76,77,65,107,71,65,49,85,69,66,104,77,67,86,86,77,120,69,122,65,82,66,103,78,86,66,65,103,84,67,107,78,104,98,71,108,109,98,51,74,117,97,87,69,120,70,106,65,85,66,103,78,86,66,65,99,84,68,86,78,104,10,98,105,66,71,99,109,70,117,89,50,108,122,89,50,56,120,72,106,65,99,66,103,78,86,66,65,77,84,70,88,66,108,90,88,73,119,76,109,57,121,90,122,69,117,98,109,49,119,98,72,86,122,76,109,78,118,98,84,66,90,77,66,77,71,10,66,121,113,71,83,77,52,57,65,103,69,71,67,67,113,71,83,77,52,57,65,119,69,72,65,48,73,65,66,74,81,115,50,119,79,84,88,119,116,52,117,47,122,70,72,102,88,102,79,81,88,79,85,67,108,48,49,119,52,72,72,74,86,107,10,75,82,56,74,107,69,90,90,49,49,89,111,66,114,76,117,80,121,82,52,52,88,73,118,100,110,72,56,74,69,99,83,101,55,100,83,114,121,54,56,100,76,122,48,72,119,88,74,98,114,87,106,84,84,66,76,77,65,52,71,65,49,85,100,10,68,119,69,66,47,119,81,69,65,119,73,72,103,68,65,77,66,103,78,86,72,82,77,66,65,102,56,69,65,106,65,65,77,67,115,71,65,49,85,100,73,119,81,107,77,67,75,65,73,80,47,104,110,122,65,54,69,74,99,85,65,67,111,107,10,117,116,110,110,102,50,121,99,67,78,85,75,74,117,76,53,107,83,76,88,89,43,71,69,100,110,117,67,77,65,111,71,67,67,113,71,83,77,52,57,66,65,77,67,65,48,103,65,77,69,85,67,73,81,68,119,75,56,98,122,90,81,78,72,10,50,117,83,48,73,78,54,78,82,98,70,49,73,106,54,112,49,121,122,56,76,80,80,73,77,72,108,74,54,90,115,99,49,103,73,103,83,47,106,74,67,78,81,74,111,118,108,68,48,81,104,90,105,110,116,112,83,74,113,105,116,78,98,112,10,116,72,55,120,114,54,68,84,104,105,116,90,103,112,77,61,10,45,45,45,45,45,69,78,68,32,67,69,82,84,73,70,73,67,65,84,69,45,45,45,45,45,10]},"signature":{"type":"Buffer","data":[48,68,2,32,45,146,156,249,116,51,246,158,178,158,2,3,130,25,170,251,189,51,239,128,157,65,234,144,222,30,82,234,99,192,142,213,2,32,25,62,255,5,156,137,103,202,142,1,61,72,90,186,44,197,179,111,190,193,186,132,32,248,107,36,211,130,148,174,173,218]}},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}}},{},{},{},{},{},{},{},{},{}]
[2019-10-03 15:11:45.493] [ERROR] Transaction - invoke chaincode proposal was bad'
------------------------------------------------------------

여러번의 테스트 결과, 최초 1,2회 정도가 에러가 발생하며, 
이후부터는 정상 작동하는 것으로 파악.










































=======================================================================================
=== chaincode
=======================================================================================
chaincode는 key, value로만 저장 가능.
key가 2개인 구조로는 불가.

id는 무조건 key.
여기에 날짜를 검색 조건으로 하여 넣을 수가 없는가? 
key, value 구조이므로, 불가능 할 것으로 예상.

기존 sample인 balance.go 를 이용하여, 구현하도록 함.

go
strconv.Atoi : 패키지 중 메소드로 문자열을 숫자로 변경해 준다.
각 거래 데이터를 배열로 저장해야 할 것으로 예상.
또한, 중복처리를 위해, 저장 시에는 가장 최근 것에 대해,
데이터가 존재하는지 확인 한 후 저장하는 로직이 필요할 것으로 예상.

struct는 user는 삭제 후,
trade 추가.
user 정보는 필요가 없을 것으로 예상됨.
해당 ID에 대한 거래 정보만 가지고 있으면 되지 않을까 생각
------------------------------------------------------------
type Trade struct {
        ObjectType string `json:"docType"`
        id         string `json:"id"`
        date       string `json:"date"`
        data       string `json:"data"`
}
-----------------------------------------------------------

invoke 메소드에 
saveTrade라면 명칭으로 추가. 거래 정보를 저장하는 메소드로,
이에 대한 func 또한 구현해야 함.
-----------------------------------------------------------
func (t *SimpleChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
  --- 생략 ---

        if function == "test" {
                // Add an entity to its state
                return t.test(stub, args)
        }
  --- 생략 ---
        logger.Errorf("Unknown action, check the first argument, must be one of 'delete', 'query', or 'move'. But got: %v", args[0])
        return shim.Error(fmt.Sprintf("Unknown action, check the first argument, must be one of 'delete', 'query', or 'move'. But got: %v", args[0]))
}
-----------------------------------------------------------

saveTrade 저장 메소드 구현.
-----------------------------------------------------------
func (t *SimpleChaincode) saveTrade(stub shim.ChaincodeStubInterface, args []string) pb.Response {
        // must be an invoke
        var id   string         // User id
        var date string         // trade date : yyyymmdd 24hhmiss
        var data string         // trade data
        var err error

        if len(args) != 2 {
                return shim.Error("Incorrect number of arguments. Expecting 2, function followed by 1 name and 1 value")
        }

        id = args[0]
        date = strconv.Atoi(args[1])
        data = args[2]

        Trade := &Trade{id, date, data}
        TradeJSONasBytes, err := json.Marshal(Trade)

        // Write the state to the ledger
        err = stub.PutState(id, TradeJSONasBytes)
        if err != nil {
                return shim.Error(err.Error())
        }

    return shim.Success([]byte("add succeed!!!"))
}
-----------------------------------------------------------


instantiate chaincode에서 에러.
-----------------------------------------------------------
2019-09-26 07:33:45.458 UTC [msp.identity] Sign -> DEBU 04c Sign: digest: 6AC2328B8A7816E4DB018AD36426DEC8DECEBDF743B7247188309B69FFE0040B 
Error: could not assemble transaction, err proposal response was not successful, error code 500, msg error starting container: error starting container: Failed to generate platform-specific docker build: Error returned from build: 2 "# github.com/hyperledger/fabric/nmplus/chaincode/go/ldt_cc
chaincode/input/src/github.com/hyperledger/fabric/nmplus/chaincode/go/ldt_cc/example_cc.go:237:10: cannot assign int to date (type string) in multiple assignment
chaincode/input/src/github.com/hyperledger/fabric/nmplus/chaincode/go/ldt_cc/example_cc.go:240:28: too few values in &Trade literal
-----------------------------------------------------------

chaincode/input/src/github.com/hyperledger/fabric/nmplus/chaincode/go/ldt_cc/example_cc.go:237:10: cannot assign int to date (type string) in multiple assignment
내용은 strconv에서 에러가 나는 듯. 날짜를 숫자로 넘기게 되면 이를 문자열로 변경해야 한다.
일단은 해당 펑션 제거.


chaincode/input/src/github.com/hyperledger/fabric/nmplus/chaincode/go/ldt_cc/example_cc.go:240:28: too few values in &Trade literal
값이 많다고 하는 듯.
-----------------------------------------------------------
       Trade := &Trade{"trade", date, data}
        TradeJSONasBytes, err := json.Marshal(Trade)
-----------------------------------------------------------
struct에 id는 넣지 않는 듯.

-----------------------------------------------------------
        err = stub.PutState(id, TradeJSONasBytes)
-----------------------------------------------------------
id는 이 부분에서 값으로 지정한다. ( key )


하지만, RDB처럼 저장이 불가.
Key, Date를 key로 하여, 데이터를 저장해야 하나,
Key, value로만 저장이 가능하기에, id만 key를 넣고,
나머지는 모두 배열로 저장하는 것을 좋을 듯.

테스트 중 에러.
-----------------------------------------------------------
2019-10-01 03:48:07.641 UTC [msp.identity] Sign -> DEBU 04b Sign: plaintext: 0AA7070A6708031A0C08F798CBEC0510...314D53500A04657363630A0476736363 
2019-10-01 03:48:07.641 UTC [msp.identity] Sign -> DEBU 04c Sign: digest: 15D39B49E0FB800C6B4CF8A40632E041EFC118524DB41C4D8F7921BA175BEFF3 
Error: could not assemble transaction, err proposal response was not successful, error code 500, msg error starting container: error starting container: Failed to generate platform-specific docker build: Error returned from build: 2 "# github.com/hyperledger/fabric/nmplus/chaincode/go/ldt_cc
chaincode/input/src/github.com/hyperledger/fabric/nmplus/chaincode/go/ldt_cc/example_cc.go:230:13: data declared and not used
-----------------------------------------------------------
data 변수명을 선언은 했으나, 사용하지 않아서 오류가 발생한 것으로 판단됨.
현재 단계적으로 구축을 하기 위해, 테스트 중이므로, 
해당 부분은 일단 주석 처리 후 재테스트
이 때, HLF Network는 container만 중지했다가 실행할 경우, 제대로 install이 되지 않는다.
이미지를 모두 삭제한 후 재다운로드 받아야 한다.


배열에 담아서 처리하는 것으로 테스트
-----------------------------------------------------------
func (t *SimpleChaincode) saveTrade(stub shim.ChaincodeStubInterface, args []string) pb.Response { 
        // must be an invoke 
        var id   string         // User id 
        // var date string      // trade date : yyyymmdd 24hhmiss 
        var data string         // trade data 
        var err error 
        // var tmpArr []InnerData 
        // var insArr [...]byte 
        var tt []string 
 
        logger.Infof("arg", args[0]) 
        logger.Infof("arg", args[1]) 
        logger.Infof("arg", args[2]) 
 
        if len(args) != 3 { 
                return shim.Error("Incorrect number of arguments. Expecting 2, function followed by 1 name and 1 value") 
        } 
 
        id = args[0] 
        // date, err = strconv.Atoi(args[1]) 
        // date = args[1] 
        data = args[2] 
 
        logger.Infof("data length : %s", data) 
        // olddata, err := stub.GetState(id) 
 
        trade := Trade{tt} 
        // json.Unmarshal(olddata, &trade) 
 
        logger.Infof("oldData length : %d", len(trade.data)) 
 
        tradeJSONasBytes, err := json.Marshal(trade) 
        err = stub.PutState(id, tradeJSONasBytes) 
        if err != nil { 
                return shim.Error(err.Error()) 
        } 
 
    return shim.Success([]byte("add succeed!!!")) 
} 
-----------------------------------------------------------

배열에 담긴 것은 byte로 되어 있어야 한다고 에러.
string은 안 된다고 에러 메세지.
-----------------------------------------------------------
019-10-01 04:15:37.011 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 04a Using default vscc
2019-10-01 04:15:37.012 UTC [msp.identity] Sign -> DEBU 04b Sign: plaintext: 0AA6070A6608031A0B08E9A5CBEC0510...314D53500A04657363630A0476736363 
2019-10-01 04:15:37.012 UTC [msp.identity] Sign -> DEBU 04c Sign: digest: 388A46D876A053668B4C7BF65C44D9B0ADBFB1881E90D615CED5B20817CBDBBA 
Error: could not assemble transaction, err proposal response was not successful, error code 500, msg error starting container: error starting container: Failed to generate platform-specific docker build: Error returned from build: 2 "# github.com/hyperledger/fabric/nmplus/chaincode/go/ldt_cc
chaincode/input/src/github.com/hyperledger/fabric/nmplus/chaincode/go/ldt_cc/example_cc.go:255:16: cannot use trade.data (type string) as type []byte in assignment
-----------------------------------------------------------

수정 후 다른 에러.
문법적 오류인 것으로 판단
-----------------------------------------------------------
2019-10-01 04:37:56.774 UTC [msp.identity] Sign -> DEBU 04b Sign: plaintext: 0AA7070A6708031A0C08A4B0CBEC0510...314D53500A04657363630A0476736363 
2019-10-01 04:37:56.774 UTC [msp.identity] Sign -> DEBU 04c Sign: digest: C9930EDF632013FF9A80C3EE9147FF39C6CE3ED3C619D39CEB1FE934EA7AEF36 
Error: could not assemble transaction, err proposal response was not successful, error code 500, msg error starting container: error starting container: Failed to generate platform-specific docker build: Error returned from build: 2 "# github.com/hyperledger/fabric/nmplus/chaincode/go/ldt_cc
chaincode/input/src/github.com/hyperledger/fabric/nmplus/chaincode/go/ldt_cc/example_cc.go:232:15: syntax error: unexpected newline, expecting type
"
-----------------------------------------------------------

= 과 :=의 차이는 무엇인가?
= 의 경우는 변수가 선언되어 있어야 함. 선언되어 있는 변수에 값을 설정하는 것.
:=의 경우, 변수가 선언되어 있지 않았을 때,
바로 선언하며, 해당 변수에 값을 설정하는 것.
서로 반대로 사용할 경우, 문법에러 발생.

http://golang.site/go/article/16-Go-%EA%B5%AC%EC%A1%B0%EC%B2%B4-struct

https://medium.com/@sagehoon/disco-very-go-%EB%B0%B0%EC%97%B4%EA%B3%BC-%EC%8A%AC%EB%9D%BC%EC%9D%B4%EC%8A%A4-9e8fb5853ede
[]로 선언하는 것은 slice며,
[]뒤에 type을 명시하여, 해당 slice 안에 어떠한 type의 데이터가 들어가게 되는지 선언하게 된다.

json 형식에 대한 처리.
https://bada-ie.com/board/view/?page=1&uid=1700&category_code=91&code=all&key=&keyfield=
검색 결과., json 형식일 경우, encoding/json을 이용하여,
unmashal, 또는 mashal을 사용해야 한다.


chaincode 변경 시 매번 이미지 삭제 후 이미지 다운로드
컨테이너 생성, 서버 기동, chaincode 설치 테스트. 진행.
최소 10분 정도의 시간이 걸림. 이를 줄일 수는 없는가?

검색을 통해, 해당 go 파일이 있는 곳에서 
-----------------------------------------------------------
go test
-----------------------------------------------------------
실행을 통해, 적어도 문법적 오류는 체크 할 수 있음을 파악.




배열은 어떻게 선언해야 하는가?


최초 시 에러 메세지 변경됨.
어디에서 발생하는 에러인가?
// TODO
-----------------------------------------------------------
2019-10-01T10:26:41.218Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-01T10:26:41.222Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-01T10:26:41.223Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-01T10:26:41.223Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-01T10:26:41.223Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-01T10:26:41.223Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-01T10:26:41.223Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-01T10:26:41.223Z - error: [Peer.js]: sendProposal - timed out after:45000
2019-10-01T10:26:41.224Z - error: [Peer.js]: sendProposal - timed out after:45000
[2019-10-01 19:26:41.224] [ERROR] Transaction - [{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}},"isProposalResponse":true},{},{},{},{},{},{},{},{},{}]
[2019-10-01 19:26:41.225] [ERROR] Transaction - invoke chaincode proposal was bad
[2019-10-01 19:26:41.226] [ERROR] Transaction - [{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}},"isProposalResponse":true},{},{},{},{},{},{},{},{},{}]
[2019-10-01 19:26:41.226] [ERROR] Transaction - invoke chaincode proposal was bad
[2019-10-01 19:26:41.227] [ERROR] Transaction - [{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}},"isProposalResponse":true},{},{},{},{},{},{},{},{},{}]
[2019-10-01 19:26:41.227] [ERROR] Transaction - invoke chaincode proposal was bad
[2019-10-01 19:26:41.227] [ERROR] Transaction - [{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}},"isProposalResponse":true},{},{},{},{},{},{},{},{},{}]
[2019-10-01 19:26:41.228] [ERROR] Transaction - invoke chaincode proposal was bad
[2019-10-01 19:26:41.228] [ERROR] Transaction - [{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}},"isProposalResponse":true},{},{},{},{},{},{},{},{},{}]
[2019-10-01 19:26:41.228] [ERROR] Transaction - invoke chaincode proposal was bad
[2019-10-01 19:26:41.229] [ERROR] Transaction - [{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}},"isProposalResponse":true},{},{},{},{},{},{},{},{},{}]
[2019-10-01 19:26:41.229] [ERROR] Transaction - invoke chaincode proposal was bad
[2019-10-01 19:26:41.229] [ERROR] Transaction - [{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}},"isProposalResponse":true},{},{},{},{},{},{},{},{},{}]
[2019-10-01 19:26:41.230] [ERROR] Transaction - invoke chaincode proposal was bad
[2019-10-01 19:26:41.232] [ERROR] Transaction - [{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}},"isProposalResponse":true},{},{},{},{},{},{},{},{},{}]
[2019-10-01 19:26:41.233] [ERROR] Transaction - invoke chaincode proposal was bad
[2019-10-01 19:26:41.233] [ERROR] Transaction - [{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}},"isProposalResponse":true},{},{},{},{},{},{},{},{},{}]
[2019-10-01 19:26:41.234] [ERROR] Transaction - invoke chaincode proposal was bad
[2019-10-01 19:26:41.234] [ERROR] Transaction - [{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}},"isProposalResponse":true},{},{},{},{},{},{},{},{},{}]
[2019-10-01 19:26:41.235] [ERROR] Transaction - invoke chaincode proposal was bad
[2019-10-01 19:26:41.235] [DEBUG] Transaction - Failed to send Proposal and receive all good ProposalResponse
[2019-10-01 19:26:41.236] [ERROR] Transaction - Failed to invoke chaincode. cause:Failed to send Proposal and receive all good ProposalResponse
(node:16299) UnhandledPromiseRejectionWarning: Error: Failed to invoke chaincode. cause:Failed to send Proposal and receive all good ProposalResponse
    at TrxClient.invokeChainCode (/home/fabric/go/src/github.com/ldt-hf/ldt-client/build/webpack:/server/blockchain/transaction/transaction.js:204:11)
    at <anonymous>
    at process._tickCallback (internal/process/next_tick.js:189:7)
(node:16299) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). (rejection id: 1)
-----------------------------------------------------------

에러 상태에서 계속 실행 해 보면, 메세지가  더 길어지는 것이 확인 됨.
-----------------------------------------------------------
[2019-10-02 11:53:46.189] [ERROR] Transaction - invoke chaincode proposal was bad
[2019-10-02 11:53:46.190] [ERROR] Transaction - [{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:7051","name":"peer0.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer0.org1.nmplus.com","grpc.ssl_target_name_override":"peer0.org1.nmplus.com","grpc.default_authority":"peer0.org1.nmplus.com"}},"isProposalResponse":true},{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:8051","name":"peer1.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer1.org1.nmplus.com","grpc.ssl_target_name_override":"peer1.org1.nmplus.com","grpc.default_authority":"peer1.org1.nmplus.com"}},"isProposalResponse":true},{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:9051","name":"peer2.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer2.org1.nmplus.com","grpc.ssl_target_name_override":"peer2.org1.nmplus.com","grpc.default_authority":"peer2.org1.nmplus.com"}},"isProposalResponse":true},{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:10051","name":"peer3.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer3.org1.nmplus.com","grpc.ssl_target_name_override":"peer3.org1.nmplus.com","grpc.default_authority":"peer3.org1.nmplus.com"}},"isProposalResponse":true},{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:11051","name":"peer4.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer4.org1.nmplus.com","grpc.ssl_target_name_override":"peer4.org1.nmplus.com","grpc.default_authority":"peer4.org1.nmplus.com"}},"isProposalResponse":true},{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:12051","name":"peer5.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer5.org1.nmplus.com","grpc.ssl_target_name_override":"peer5.org1.nmplus.com","grpc.default_authority":"peer5.org1.nmplus.com"}},"isProposalResponse":true},{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:13051","name":"peer6.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer6.org1.nmplus.com","grpc.ssl_target_name_override":"peer6.org1.nmplus.com","grpc.default_authority":"peer6.org1.nmplus.com"}},"isProposalResponse":true},{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:14051","name":"peer7.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer7.org1.nmplus.com","grpc.ssl_target_name_override":"peer7.org1.nmplus.com","grpc.default_authority":"peer7.org1.nmplus.com"}},"isProposalResponse":true},{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:15051","name":"peer8.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer8.org1.nmplus.com","grpc.ssl_target_name_override":"peer8.org1.nmplus.com","grpc.default_authority":"peer8.org1.nmplus.com"}},"isProposalResponse":true},{"status":500,"payload":{"type":"Buffer","data":[]},"peer":{"url":"grpcs://localhost:16051","name":"peer9.org1.nmplus.com","options":{"grpc.max_receive_message_length":-1,"grpc.max_send_message_length":-1,"grpc.keepalive_time_ms":120000,"grpc.http2.min_time_between_pings_ms":120000,"grpc.keepalive_timeout_ms":20000,"grpc.http2.max_pings_without_data":0,"grpc.keepalive_permit_without_calls":1,"name":"peer9.org1.nmplus.com","grpc.ssl_target_name_override":"peer9.org1.nmplus.com","grpc.default_authority":"peer9.org1.nmplus.com"}},"isProposalResponse":true}]
[2019-10-02 11:53:46.190] [ERROR] Transaction - invoke chaincode proposal was bad
-----------------------------------------------------------

확인 결과, 위에 에러에서는 peer0에 대한 로그만 있는 것으로 파악됨.
아직, gossip으로 전파가 되지 않은 상태로 여겨짐.



peer0의 로그를 보면, chaincode에 대한 로그를 볼 수 있을 것이라 판단.
확인.
-----------------------------------------------------------
2019-10-02 04:31:34.967 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 071 2019-10-02 04:31:34.965 UTC [ldt_cc0] Infof -> INFO 002 arg%!(EXTRA string=testuser)
2019-10-02 04:31:34.967 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 072 2019-10-02 04:31:34.965 UTC [ldt_cc0] Infof -> INFO 003 arg%!(EXTRA string=20001231175011)
2019-10-02 04:31:34.967 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 073 2019-10-02 04:31:34.965 UTC [ldt_cc0] Infof -> INFO 004 arg%!(EXTRA string=eveveevev)
2019-10-02 04:31:34.968 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 074 panic: runtime error: index out of range
2019-10-02 04:31:34.969 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 075        panic: runtime error: invalid memory address or nil pointer dereference
2019-10-02 04:31:34.970 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 076 [signal SIGSEGV: segmentation violation code=0x1 addr=0x28 pc=0x957456]
2019-10-02 04:31:34.971 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 077 
2019-10-02 04:31:34.971 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 078 goroutine 29 [running]:
2019-10-02 04:31:34.971 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 079 github.com/hyperledger/fabric/core/chaincode/shim.(*Handler).triggerNextState(0xc00038a480, 0x0, 0xc00038c960)
2019-10-02 04:31:34.971 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 07a        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/shim/handler.go:35 +0x26
2019-10-02 04:31:34.971 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 07b github.com/hyperledger/fabric/core/chaincode/shim.(*Handler).handleTransaction.func1.1(0xc00038a480, 0xc000379eb0, 0xc00038c960)
2019-10-02 04:31:34.971 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 07c        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/shim/handler.go:247 +0x42
2019-10-02 04:31:34.971 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 07d panic(0xae7c40, 0x130dc80)
2019-10-02 04:31:34.973 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 07e        /opt/go/src/runtime/panic.go:513 +0x1b9
2019-10-02 04:31:34.973 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 07f main.(*SimpleChaincode).saveTrade(0x137f440, 0xc66e40, 0xc0003d22c0, 0xc0003cc490, 0x3, 0x3, 0x0, 0x0, 0x0, 0x0, ...)
2019-10-02 04:31:34.973 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 080        /chaincode/input/src/github.com/hyperledger/fabric/nmplus/chaincode/go/ldt_cc/example_cc.go:265 +0x2cc
2019-10-02 04:31:34.973 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 081 main.(*SimpleChaincode).Invoke(0x137f440, 0xc66e40, 0xc0003d22c0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, ...)
2019-10-02 04:31:34.973 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 082        /chaincode/input/src/github.com/hyperledger/fabric/nmplus/chaincode/go/ldt_cc/example_cc.go:135 +0x321
2019-10-02 04:31:34.973 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 083 github.com/hyperledger/fabric/core/chaincode/shim.(*Handler).handleTransaction.func1(0xc00038a480, 0xc00038c960, 0xc000396300)
2019-10-02 04:31:34.973 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 084        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/shim/handler.go:273 +0x4eb
2019-10-02 04:31:34.973 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 085 created by github.com/hyperledger/fabric/core/chaincode/shim.(*Handler).handleTransaction
2019-10-02 04:31:34.973 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 086        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/shim/handler.go:242 +0x53
2019-10-02 04:31:34.995 UTC [chaincode] ProcessStream -> ERRO 087 handling chaincode support stream: rpc error: code = Canceled desc = context canceled
receive failed
github.com/hyperledger/fabric/core/chaincode.(*Handler).ProcessStream
        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/handler.go:427
github.com/hyperledger/fabric/core/chaincode.(*ChaincodeSupport).HandleChaincodeStream
        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/chaincode_support.go:191
github.com/hyperledger/fabric/core/chaincode.(*ChaincodeSupport).Register
        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/chaincode_support.go:196
github.com/hyperledger/fabric/core/chaincode/accesscontrol.(*interceptor).Register
        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/accesscontrol/interceptor.go:57
github.com/hyperledger/fabric/protos/peer._ChaincodeSupport_Register_Handler
        /opt/gopath/src/github.com/hyperledger/fabric/protos/peer/chaincode_shim.pb.go:1069
github.com/hyperledger/fabric/vendor/google.golang.org/grpc.(*Server).processStreamingRPC
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/google.golang.org/grpc/server.go:1124
github.com/hyperledger/fabric/vendor/google.golang.org/grpc.(*Server).handleStream
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/google.golang.org/grpc/server.go:1212
github.com/hyperledger/fabric/vendor/google.golang.org/grpc.(*Server).serveStreams.func1.1
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/google.golang.org/grpc/server.go:686
runtime.goexit
        /opt/go/src/runtime/asm_amd64.s:1333
2019-10-02 04:31:35.637 UTC [dockercontroller] func2 -> INFO 088 Container dev-peer0.org1.nmplus.com-ldt-0.1 has closed its IO channel
2019-10-02 04:32:04.964 UTC [endorser] callChaincode -> INFO 089 [mychannel][63a14a26] Exit chaincode: name:"ldt"  (34977ms)
2019-10-02 04:32:04.967 UTC [endorser] SimulateProposal -> ERRO 08a [mychannel][63a14a26] failed to invoke chaincode name:"ldt" , error: timeout expired while executing transaction
github.com/hyperledger/fabric/core/chaincode.(*Handler).Execute
        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/handler.go:1250
github.com/hyperledger/fabric/core/chaincode.(*ChaincodeSupport).execute
        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/chaincode_support.go:313
github.com/hyperledger/fabric/core/chaincode.(*ChaincodeSupport).Invoke
        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/chaincode_support.go:302
github.com/hyperledger/fabric/core/chaincode.(*ChaincodeSupport).Execute
        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/chaincode_support.go:239
github.com/hyperledger/fabric/core/endorser.(*SupportImpl).Execute
        /opt/gopath/src/github.com/hyperledger/fabric/core/endorser/support.go:147
github.com/hyperledger/fabric/core/endorser.(*Endorser).callChaincode
        /opt/gopath/src/github.com/hyperledger/fabric/core/endorser/endorser.go:146
github.com/hyperledger/fabric/core/endorser.(*Endorser).SimulateProposal
        /opt/gopath/src/github.com/hyperledger/fabric/core/endorser/endorser.go:247
github.com/hyperledger/fabric/core/endorser.(*Endorser).ProcessProposal
        /opt/gopath/src/github.com/hyperledger/fabric/core/endorser/endorser.go:500
github.com/hyperledger/fabric/core/handlers/auth/filter.(*expirationCheckFilter).ProcessProposal
        /opt/gopath/src/github.com/hyperledger/fabric/core/handlers/auth/filter/expiration.go:61
github.com/hyperledger/fabric/core/handlers/auth/filter.(*filter).ProcessProposal
        /opt/gopath/src/github.com/hyperledger/fabric/core/handlers/auth/filter/filter.go:32
github.com/hyperledger/fabric/protos/peer._Endorser_ProcessProposal_Handler.func1
        /opt/gopath/src/github.com/hyperledger/fabric/protos/peer/peer.pb.go:169
github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware.ChainUnaryServer.func1.1
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware/chain.go:31
github.com/hyperledger/fabric/core/comm.(*Throttle).UnaryServerIntercptor
        /opt/gopath/src/github.com/hyperledger/fabric/core/comm/throttle.go:54
github.com/hyperledger/fabric/core/comm.(*Throttle).UnaryServerIntercptor-fm
        /opt/gopath/src/github.com/hyperledger/fabric/peer/node/start.go:228
github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware.ChainUnaryServer.func1.1
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware/chain.go:34
github.com/hyperledger/fabric/common/grpclogging.UnaryServerInterceptor.func1
        /opt/gopath/src/github.com/hyperledger/fabric/common/grpclogging/server.go:91
github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware.ChainUnaryServer.func1.1
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware/chain.go:34
github.com/hyperledger/fabric/common/grpcmetrics.UnaryServerInterceptor.func1
        /opt/gopath/src/github.com/hyperledger/fabric/common/grpcmetrics/interceptor.go:30
github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware.ChainUnaryServer.func1
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware/chain.go:39
github.com/hyperledger/fabric/protos/peer._Endorser_ProcessProposal_Handler
        /opt/gopath/src/github.com/hyperledger/fabric/protos/peer/peer.pb.go:171
github.com/hyperledger/fabric/vendor/google.golang.org/grpc.(*Server).processUnaryRPC
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/google.golang.org/grpc/server.go:982
github.com/hyperledger/fabric/vendor/google.golang.org/grpc.(*Server).handleStream
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/google.golang.org/grpc/server.go:1208
github.com/hyperledger/fabric/vendor/google.golang.org/grpc.(*Server).serveStreams.func1.1
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/google.golang.org/grpc/server.go:686
runtime.goexit
        /opt/go/src/runtime/asm_amd64.s:1333
error sending
failed to execute transaction 63a14a26d1feb2702ec852526bf22a52f4b917feb12efc40fd789af1c3c79149
github.com/hyperledger/fabric/core/chaincode.processChaincodeExecutionResult
        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/chaincode_support.go:245
github.com/hyperledger/fabric/core/chaincode.(*ChaincodeSupport).Execute
        /opt/gopath/src/github.com/hyperledger/fabric/core/chaincode/chaincode_support.go:240
github.com/hyperledger/fabric/core/endorser.(*SupportImpl).Execute
        /opt/gopath/src/github.com/hyperledger/fabric/core/endorser/support.go:147
github.com/hyperledger/fabric/core/endorser.(*Endorser).callChaincode
        /opt/gopath/src/github.com/hyperledger/fabric/core/endorser/endorser.go:146
github.com/hyperledger/fabric/core/endorser.(*Endorser).SimulateProposal
        /opt/gopath/src/github.com/hyperledger/fabric/core/endorser/endorser.go:247
github.com/hyperledger/fabric/core/endorser.(*Endorser).ProcessProposal
        /opt/gopath/src/github.com/hyperledger/fabric/core/endorser/endorser.go:500
github.com/hyperledger/fabric/core/handlers/auth/filter.(*expirationCheckFilter).ProcessProposal
        /opt/gopath/src/github.com/hyperledger/fabric/core/handlers/auth/filter/expiration.go:61
github.com/hyperledger/fabric/core/handlers/auth/filter.(*filter).ProcessProposal
        /opt/gopath/src/github.com/hyperledger/fabric/core/handlers/auth/filter/filter.go:32
github.com/hyperledger/fabric/protos/peer._Endorser_ProcessProposal_Handler.func1
        /opt/gopath/src/github.com/hyperledger/fabric/protos/peer/peer.pb.go:169
github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware.ChainUnaryServer.func1.1
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware/chain.go:31
github.com/hyperledger/fabric/core/comm.(*Throttle).UnaryServerIntercptor
        /opt/gopath/src/github.com/hyperledger/fabric/core/comm/throttle.go:54
github.com/hyperledger/fabric/core/comm.(*Throttle).UnaryServerIntercptor-fm
        /opt/gopath/src/github.com/hyperledger/fabric/peer/node/start.go:228
github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware.ChainUnaryServer.func1.1
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware/chain.go:34
github.com/hyperledger/fabric/common/grpclogging.UnaryServerInterceptor.func1
        /opt/gopath/src/github.com/hyperledger/fabric/common/grpclogging/server.go:91
github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware.ChainUnaryServer.func1.1
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware/chain.go:34
github.com/hyperledger/fabric/common/grpcmetrics.UnaryServerInterceptor.func1
        /opt/gopath/src/github.com/hyperledger/fabric/common/grpcmetrics/interceptor.go:30
github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware.ChainUnaryServer.func1
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/github.com/grpc-ecosystem/go-grpc-middleware/chain.go:39
github.com/hyperledger/fabric/protos/peer._Endorser_ProcessProposal_Handler
        /opt/gopath/src/github.com/hyperledger/fabric/protos/peer/peer.pb.go:171
github.com/hyperledger/fabric/vendor/google.golang.org/grpc.(*Server).processUnaryRPC
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/google.golang.org/grpc/server.go:982
github.com/hyperledger/fabric/vendor/google.golang.org/grpc.(*Server).handleStream
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/google.golang.org/grpc/server.go:1208
github.com/hyperledger/fabric/vendor/google.golang.org/grpc.(*Server).serveStreams.func1.1
        /opt/gopath/src/github.com/hyperledger/fabric/vendor/google.golang.org/grpc/server.go:686
runtime.goexit
        /opt/go/src/runtime/asm_amd64.s:1333
2019-10-02 04:32:04.968 UTC [comm.grpc.server] 1 -> INFO 08b unary call completed grpc.service=protos.Endorser grpc.method=ProcessProposal grpc.peer_address=172.25.0.1:49186 grpc.code=OK grpc.call_duration=34.990871029s
-----------------------------------------------------------
밑에 부분의 에러만 보았는데, 
info 부분에 panic: runtime error: index out of range 라는 오류가 확인 됨.

배열 부분에서 오류가 나는 것이 아닌가 예상.
일단 어느 부분인지 확인이 제대로 보이지 않으므로, 로그를 찍어 확인.
-----------------------------------------------------------
       id = args[0]
        logger.Infof("11111111")
        // date, err = strconv.Atoi(args[1])
        // date = args[1]
        // data = args[2]

        // trade := make([]Trade, 1)            // contruct

        innerData1 := InnerData{"20190101", "2", "3"}
        logger.Infof("222222222")
        innerData2 := InnerData{"20190102", "2", "3"}
        innerData3 := InnerData{"20190103", "2", "3"}

        var data = []InnerData{}
        logger.Infof("333333333333")

        // trade[0].datas = innerData1
        // trade[1].datas = innerData2
        // trade[2].datas = innerData3

        data[0] = innerData1
        logger.Infof("444444444")
        data[1] = innerData2
        data[2] = innerData3

        // tradeJSONasBytes, err := json.Marshal(trade)
        tradeJSONasBytes, err := json.Marshal(data)
-----------------------------------------------------------
테스트 결과 444 부분이 찍히지 않는 것으로 보아, data[0] 부분에서 에러가 나며,
배열 사이즈가 없어, 메모리에 생성되지 않은 것은 아닌가 예상 됨.


배열을 먼저 선언한 후 로직을 처리하는 것으로 변경.
-----------------------------------------------------------
        innerData3 := InnerData{"20190103", "2", "3"}

        var data = make([]InnerData, 1)
        logger.Infof("333333333333")

        // trade[0].datas = innerData1
        // trade[1].datas = innerData2
        // trade[2].datas = innerData3

        data[0] = innerData1
        logger.Infof("444444444")
        data = append(data, innerData2)
        data = append(data, innerData3)
        // data[1] = innerData2
        // data[2] = innerData3
-----------------------------------------------------------

테스트 결과
-----------------------------------------------------------
[{}, {}, {} ]
-----------------------------------------------------------
값이 아무 것도 나오지 않음.

query쪽에서 잘못 가져오는 것인지,
데이터가 저장이 안 된 것인지 확인이 필요.
-----------------------------------------------------------

query에서 배열로 가져오도록 수정.
-----------------------------------------------------------
// Query callback representing the query of a chaincode
func (t *SimpleChaincode) query(stub shim.ChaincodeStubInterface, args []string) pb.Response {
        var id string

        if len(args) != 1 {
                return shim.Error("Incorrect number of arguments. Expection name of the ID to query")
        }

        id = args[0]

        logger.Infof("####### query args #### : %s", id)

        marshalData, err := stub.GetState(id)

        if err != nil {
                jsonResp := "{\"Error\":\"Failed to get state for " + id + "\"}"
                return shim.Error(jsonResp)
        }

        // trade := Trade{}
        var data []Trade

        json.Unmarshal(marshalData, data)
        // logger.Infof("Query Response:%s\n", trade)
        // return shim.Success(json.Marshal(trade))
        logger.Infof("=-----------------")
        return shim.Success(data)
}
-----------------------------------------------------------

테스트 결과
-----------------------------------------------------------
# github.com/ldt-hf/tools/chaincodes/ldt_cc
./example_cc.go:334:21: cannot use data (type []Trade) as type []byte in argument to shim.Success
fabric@localhost:~/go/src/github.com/ldt-hf/tools/chaincodes/ldt_cc: 
-----------------------------------------------------------
Trade structs를 byte로 변환이 안 된다고 에러 발생.

좀 더 golang에 대해 공부를 해 본 결과
-----------------------------------------------------------
func (t *SimpleChaincode) query(stub shim.ChaincodeStubInterface, args []string) pb.Response {
-----------------------------------------------------------
여기에서 pb는 peer를 뜻하는 것이며,  pb.Response는 byte로 return 값을 받는다.
mashal을 쓰는 이유는 json 형식을 byte형태로 변경하기 위함이며,
String 형을 return 할 때는 []byte를 사용하여, 스트링을 byte형으로 변경해 준다.
-----------------------------------------------------------
        jsonResp := "{\"Hello ChainCode !!!!!!!!!!!!!\"}"
        logger.Infof("Query Response:%s\n", jsonResp)
        return shim.Success([]byte(jsonResp))
-----------------------------------------------------------

-----------------------------------------------------------
        // Get the state from the ledger
        Avalbytes, err := stub.GetState(A)
        if err != nil {
                jsonResp := "{\"Error\":\"Failed to get state for " + A + "\"}"
                return shim.Error(jsonResp)
        }

        if Avalbytes == nil {
                jsonResp := "{\"Error\":\"Nil amount for " + A + "\"}"
                return shim.Error(jsonResp)
        }

        jsonResp := "{\"Name\":\"" + A + "\",\"Amount\":\"" + string(Avalbytes) + "\"}"
        logger.Infof("Query Response:%s\n", jsonResp)
        return shim.Success(Avalbytes)
-----------------------------------------------------------
또한, sample 소스를 보아, stub.GetState는 byte로 return된다.


logger.Infof는 string 형으로만 되는 듯하다.
일단 byte 형은 안 되는 듯하며, string으로 변경해서 확인 가능하다.

query에서 직접 데이터 조회 후 logger를 찍어 보도록 소스 수정.
-----------------------------------------------------------
func (t *SimpleChaincode) query(stub shim.ChaincodeStubInterface, args []string) pb.Response {
        var id string

        if len(args) != 1 {
                return shim.Error("Incorrect number of arguments. Expection name of the ID to query")
        }

        id = args[0]

        logger.Infof("####### query args #### : %s", id)

        marshalData, err := stub.GetState(id)
        logger.Infof(string(marshalData))

        if err != nil {
                jsonResp := "{\"Error\":\"Failed to get state for " + id + "\"}"
                return shim.Error(jsonResp)
        }

        // trade := Trade{}
        // var data []InnerData

        var data = make([]InnerData, 1)
        json.Unmarshal(marshalData, &data)
        // logger.Infof("Query Response:%s\n", trade)
        // return shim.Success(json.Marshal(trade))
        logger.Infof("=-----------------")
        // return shim.Success([]byte(data))
        return shim.Success(marshalData)
}
-----------------------------------------------------------

확인 결과,
데이터 자체가 조회 되지 않음.
혹여, 데이터가 저장 자체가 되지 않은 것이 아닌지 확인 필요.
-----------------------------------------------------------
2019-10-02 08:12:39.254 UTC [endorser] callChaincode -> INFO 0aa [mychannel][18dd9a19] Entry chaincode: name:"ldt" 
2019-10-02 08:12:39.257 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 0ab 2019-10-02 08:12:39.255 UTC [ldt_cc0] Info -> INFO 037 ########### example_cc0 Invoke ###########
2019-10-02 08:12:39.257 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 0ac 2019-10-02 08:12:39.256 UTC [ldt_cc0] Infof -> INFO 038 ####### query args #### : testuser
2019-10-02 08:12:39.317 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 0ad 2019-10-02 08:12:39.316 UTC [ldt_cc0] Infof -> INFO 039 [{},{},{}]
2019-10-02 08:12:39.317 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 0ae 2019-10-02 08:12:39.316 UTC [ldt_cc0] Infof -> INFO 03a =-----------------
-----------------------------------------------------------

struct는 로그를 어떻게 찍을 수 있는가?
검색 결과, 
https://www.golangprograms.com/how-to-print-struct-variables-data.html
%v 옵션을 이용하여 가능.
소스 수정 후 테스트.
-----------------------------------------------------------
        innerData1 := InnerData{"20190101", "2", "3"}
        logger.Infof("222222222")
        innerData2 := InnerData{"20190102", "2", "3"}
        innerData3 := InnerData{"20190103", "2", "3"}
        logger.Infof("%+v\n", innerData1)
        logger.Infof("%+v\n", innerData2)
        logger.Infof("%+v\n", innerData3)

        var data = make([]InnerData, 1)
        logger.Infof("444444444")

        data = append(data, innerData1)
        logger.Infof("%+v\n", data)
        data = append(data, innerData2)
        logger.Infof("%+v\n", data)
        data = append(data, innerData3)
        logger.Infof("%+v\n", data)
-----------------------------------------------------------
로그 확인 결과, 배열 데이터로는 들어간 것은 확인 됨.
그러나, 저장은 제대로 안 되는 것으로 예상됨.
-----------------------------------------------------------
2019-10-02 08:42:49.828 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 05b 2019-10-02 08:42:49.828 UTC [ldt_cc0] Infof -> INFO 00b {date:20190103 productSeq:2 amount:3}
2019-10-02 08:42:49.828 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 05c 2019-10-02 08:42:49.828 UTC [ldt_cc0] Infof -> INFO 00c 333333333333
2019-10-02 08:42:49.828 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 05d 2019-10-02 08:42:49.828 UTC [ldt_cc0] Infof -> INFO 00d 444444444
2019-10-02 08:42:49.829 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 05e 2019-10-02 08:42:49.828 UTC [ldt_cc0] Infof -> INFO 00e [{date: productSeq: amount:} {date:20190101 productSeq:2 amount:3}]
2019-10-02 08:42:49.829 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 05f 2019-10-02 08:42:49.829 UTC [ldt_cc0] Infof -> INFO 00f [{date: productSeq: amount:} {date:20190101 productSeq:2 amount:3} {date:20190102 productSeq:2 amount:3}]
2019-10-02 08:42:49.831 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 060 2019-10-02 08:42:49.830 UTC [ldt_cc0] Infof -> INFO 010 [{date: productSeq: amount:} {date:20190101 productSeq:2 amount:3} {date:20190102 productSeq:2 amount:3} {date:20190103 productSeq:2 amount:3}]
2019-10-02 08:42:49.835 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 061 2019-10-02 08:42:49.831 UTC [ldt_cc0] Infof -> INFO 011 save trade ==========================
2019-10-02 08:42:49.835 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 062 2019-10-02 08:42:49.831 UTC [ldt_cc0] Infof -> INFO 012 [{},{},{},{}]
2019-10-02 08:42:49.837 UTC [endorser] callChaincode -> INFO 063 [mychannel][4eb0d886] Exit chaincode: name:"ldt"  (19ms)
2019-10-02 08:42:49.838 UTC [comm.grpc.server] 1 -> INFO 064 unary call completed grpc.service=protos.Endorser grpc.method=ProcessProposal grpc.peer_address=172.25.0.1:56190 grpc.code=OK grpc.call_duration=22.427356ms
-----------------------------------------------------------

data는 string이 아닌데,
그대로 바로 저장 되지는 않은가?
무조건 byte로 해서 저장해야 하는가?
-----------------------------------------------------------
        if err != nil {
                logger.Infof("marshal error ")
                return shim.Error(err.Error())
        }
        err = stub.PutState(id, data)
-----------------------------------------------------------

go test 결과.
-----------------------------------------------------------
# github.com/ldt-hf/tools/chaincodes/ldt_cc
./example_cc.go:288:22: undefined: tradeJSONasBytes
./example_cc.go:294:21: cannot use data (type []InnerData) as type []byte in argument to stub.PutState
fabric@localhost:~/go/src/github.com/ldt-hf/tools/chaincodes/ldt_cc: 
-----------------------------------------------------------
안 됨.
json.mashal은 json 형태로 되어 있어야 하는데,
그렇지 않기 때문에 그런 것으로 예상.
상위 struct를 만들고, 그것은 json 형식.
이를 mashal하도록 해야 할 듯.


배열에 넣는 것까진 진행이 되나, mashal에서 데이터가 사라짐.
샘플로 제공받은 chaincode 로 테스트.
-----------------------------------------------------------
-ldt-0.1] func2 -> INFO 062 2019-10-02 10:26:34.981 UTC [ldt_cc0] Infof -> INFO 012 save trade ==========================
2019-10-02 10:26:34.983 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 063 2019-10-02 10:26:34.981 UTC [ldt_cc0] Infof -> INFO 013 testuser
2019-10-02 10:26:34.983 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 064 2019-10-02 10:26:34.981 UTC [ldt_cc0] Infof -> INFO 014 {}
2019-10-02 10:26:34.983 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 065 2019-10-02 10:26:34.981 UTC [ldt_cc0] Infof -> INFO 015 {"docType":"User","name":"efef","age":"12","sex":"male"}
2019-10-02 10:26:34.983 UTC [endorser] callChaincode -> INFO 066 [mychannel][53cf293c] Exit chaincode: name:"ldt"  (4ms)
-----------------------------------------------------------
-----------------------------------------------------------
unc2 -> INFO 05f 2019-10-02 10:35:51.690 UTC [ldt_cc0] Infof -> INFO 00f [{date: productSeq: amount:} {date:20190101 productSeq:2 amount:3} {date:20190102 productSeq:2 amount:3}]
2019-10-02 10:35:51.691 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 060 2019-10-02 10:35:51.690 UTC [ldt_cc0] Infof -> INFO 010 [{date: productSeq: amount:} {date:20190101 productSeq:2 amount:3} {date:20190102 productSeq:2 amount:3} {date:20190103 productSeq:2 amount:3}]
2019-10-02 10:35:51.691 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 061 2019-10-02 10:35:51.690 UTC [ldt_cc0] Infof -> INFO 011 &{ObjectType:trade datas:[{date: productSeq: amount:} {date:20190101 productSeq:2 amount:3} {date:20190102 productSeq:2 amount:3} {date:20190103 productSeq:2 amount:3}]}
2019-10-02 10:35:51.691 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 062 2019-10-02 10:35:51.690 UTC [ldt_cc0] Infof -> INFO 012 save tr
-----------------------------------------------------------
데이터 비교 결과, struct에 들어가는 것이 다름.
여기서 에러가 발생하는 것으로 예상.

배열 일 경우, 데이터가 삭제되는 것으로 예상.
안 되는 것인지 검색.
"golang" "json.marshal" splice
https://www.dotnetperls.com/json-go
https://blog.golang.org/json-and-go
https://yourbasic.org/golang/json-example/#encode-marshal-struct-to-json
검색 페이지를 보던 중.

-----------------------------------------------------------
type FruitBasket struct {
    Name    string
    Fruit   []string
    Id      int64  `json:"ref"`
    private string // An unexported field is not encoded.
    Created time.Time
}
-----------------------------------------------------------
private 변수만 encoded 란 것을 보고, 혹시 소문자라 그런 것은 아닌지 의심.
golang은 대문자는 public, 소문자 private임.
소스 변경
-----------------------------------------------------------
type Trade struct {
        ObjectType string       `json:"docType"`
        Datas      []InnerData  `json:"datas"`
}

type InnerData struct {
        Date            string  `json:"date"`
        ProductSeq      string  `json:"productSeq"`
        Amount          string  `json:"amount"`
}
-----------------------------------------------------------

정상적으로 encoding 되는 것으로 파악.


데이터 확인 중. 배열의 첫 값이 아무런 값이 없이 저장되고 있음.
최초 초기화 때 잘못 되고 있는 건 아닌지 확인.
-----------------------------------------------------------
2019-10-02 08:42:49.828 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 05b 2019-10-02 08:42:49.828 UTC [ldt_cc0] Infof -> INFO 00b {date:20190103 productSeq:2 amount:3}
2019-10-02 08:42:49.828 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 05c 2019-10-02 08:42:49.828 UTC [ldt_cc0] Infof -> INFO 00c 333333333333
2019-10-02 08:42:49.828 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 05d 2019-10-02 08:42:49.828 UTC [ldt_cc0] Infof -> INFO 00d 444444444
2019-10-02 08:42:49.829 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 05e 2019-10-02 08:42:49.828 UTC [ldt_cc0] Infof -> INFO 00e [{date: productSeq: amount:} {date:20190101 productSeq:2 amount:3}]
2019-10-02 08:42:49.829 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 05f 2019-10-02 08:42:49.829 UTC [ldt_cc0] Infof -> INFO 00f [{date: productSeq: amount:} {date:20190101 productSeq:2 amount:3} {date:20190102 productSeq:2 amount:3}]
2019-10-02 08:42:49.831 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 060 2019-10-02 08:42:49.830 UTC [ldt_cc0] Infof -> INFO 010 [{date: productSeq: amount:} {date:20190101 productSeq:2 amount:3} {date:20190102 productSeq:2 amount:3} {date:20190103 productSeq:2 amount:3}]
2019-10-02 08:42:49.835 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 061 2019-10-02 08:42:49.831 UTC [ldt_cc0] Infof -> INFO 011 save trade ==========================
2019-10-02 08:42:49.835 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 062 2019-10-02 08:42:49.831 UTC [ldt_cc0] Infof -> INFO 012 [{},{},{},{}]
2019-10-02 08:42:49.837 UTC [endorser] callChaincode -> INFO 063 [mychannel][4eb0d886] Exit chaincode: name:"ldt"  (19ms)
2019-10-02 08:42:49.838 UTC [comm.grpc.server] 1 -> INFO 064 unary call completed grpc.service=protos.Endorser grpc.method=ProcessProposal grpc.peer_address=172.25.0.1:56190 grpc.code=OK grpc.call_duration=22.427356ms
-----------------------------------------------------------

소스 중 초기화 부분의 1을 0으로 배열 크기를 변경.
-----------------------------------------------------------
        var data = make([]InnerData, 0)
-----------------------------------------------------------
정상 확인.

거래 저장 시 배열에 계속 쌓도록 먼저 데이터를 조회한 후,
해당 배열에 추가하도록 소스 변경
-----------------------------------------------------------
        var data = make([]InnerData, 0)

        oldData, err := stub.GetState(id)
        json.Unmarshal(oldData, &data)

        innerData := InnerData{date, productseq, amount}

        logger.Infof("333333333333")

        data = append(data, innerData)
-----------------------------------------------------------

테스트 결과, 정상적으로 쌓이는 것을 확인.
중복 확인 로직 추가만 하면 될 것으로 예상
-----------------------------------------------------------
  recoveryParam: 1 }
[2019-10-03 12:16:20.643] [INFO] Transaction - testuser now has [{"date":"","productSeq":"","amount":""},{"date":"20001231175011","productSeq":"12","amount":"20"},{"date":"20001231175011","productSeq":"12","amount":"20"},{"date":"20001231175011","productSeq":"12","amount":"20"},{"date":"20001231175011","productSeq":"12","amount":"20"},{"date":"20001231175011","productSeq":"12","amount":"20"},{"date":"20001231175011","productSeq":"12","amount":"20"}] after the move
-----------------------------------------------------------

추가하도록 한 후 로그 확인. 결과
현재대로라면, 최초 빈 값이 한 번 들어가게 된다.  수정 필요.
임시 배열에 먼저 넣은 후, length 체크를 해야 함.
-----------------------------------------------------------
     var data = make([]InnerData, 0)

        oldData, err := stub.GetState(id)
        logger.Infof(string(oldData))
        logger.Infof("len : %d", len(oldData))
        json.Unmarshal(oldData, &data)

        for index, val := range data {
                logger.Infof("index : %d", index)
                logger.Infof("for data : %s", val.Date)
        }
-----------------------------------------------------------
아무것도 값이 들어가지 않은 것으로 len 체크를 해 본 결과,
key들로 인하여 길이 length는 0이 아님.
key만 있을 때 length가 41이므로 그 길이를 초과할 경우에만 기존 데이터를 넣는 것으로 로직 변경.
-----------------------------------------------------------
        oldData, err := stub.GetState(id)
        if len(oldData) > 41 {
                json.Unmarshal(oldData, &data)
        }
-----------------------------------------------------------

확인 결과, 원하는 대로 저장.
-----------------------------------------------------------
fo -> INFO 079 ########### example_cc0 Invoke ###########
2019-10-03 05:33:35.232 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 0f4 2019-10-03 05:33:35.231 UTC [ldt_cc0] Infof -> INFO 07a ####### query args #### : testuser
2019-10-03 05:33:35.266 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 0f5 2019-10-03 05:33:35.266 UTC [ldt_cc0] Infof -> INFO 07b [{"date":"20001231175011","productSeq":"12","amount":"20"},{"date":"20001231175011","productSeq":"12","amount":"20"},{"date":"20001231175011","productSeq":"12","amount":"20"},{"date":"20001231175011","productSeq":"12","amount":"20"},{"date":"20001231175011","productSeq":"12","amount":"20"},{"date":"20001231175011","productSeq":"12","amount":"20"}]
2019-10-03 05:33:35.266 UTC [peer.chaincode.dev-peer0.org1.nmplus.com-ldt-0.1] func2 -> INFO 0f6 2019-10-03 05:33:35.266 UTC [ldt_cc0] Infof -> INFO 07c =-----------------
2019-10-03 05:33:35.266 UTC [endorser] callChaincode -> INFO 0f7 [mychannel][f015e6aa] Exit chaincode: name:"ldt"  (36ms)
2019-10-03 05:33:35.268 UTC [comm.grpc.server] 1 -> INFO 0f8 unary call completed grpc.service=protos.Endorser grpc.method=ProcessProposal grpc.peer_address=172.25.0.1:50358 grpc.code=OK grpc.call_duration=39.476955ms
-----------------------------------------------------------

date는 년월일시분초까지이며,
중복일 경우, 데이터가 추가되지 않도록 처리해야 함.
dup 체크 변수와 for문을 이용하여, dup일 경우에는 
저장 처리 하지 않도록 변경

-----------------------------------------------------------
        var err error
        var dupCheck bool = false

        if len(args) != 4 {
                return shim.Error("Incorrect number of arguments. Expecting 2, function followed by 1 name and 1 value")
        }

        id = args[0]
        date := args[1]
        productseq := args[2]
        amount := args[3]
        var data = make([]InnerData, 0)

        oldData, err := stub.GetState(id)
        if len(oldData) > 41 {
                json.Unmarshal(oldData, &data)
        }

        for index, val := range data {
                if date == val.Date {
                        dupCheck = true
                        break
                }
        }

        if dupCheck {
                return shim.Success([]byte("data duplication!!!"))
        }
-----------------------------------------------------------
테스트 결과 정상적으로 진행됨.
중복 될 경우에는 저장하지 않음.


























