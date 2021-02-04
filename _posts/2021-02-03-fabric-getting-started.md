---
title: Hyperledger Fabric 설치부터 test-network 구동까지
author: gus
date: 2021-02-03 09:16:00 +0900 1
categories: [Blockchain, Hyperledger Fabric]
tags: [hyperledger, fabric, blockchain, private-blockchain]
toc: true
---

# 1. 개요

블록체인 기술이 점점 무르익고 있습니다. 대중들에게 친숙한 블록체인은 대게 **Public Blockchain**인데요, 비트코인, 이더리움이 여기에 속합니다. 퍼블릭 블록체인의 모든 정보는 공개되어 있기 때문에 블록에 한 번 쌓인 어떤 정보도 조작할 수 없는 특징이 있죠!

**이더리움**은 퍼블릭 블록체인 기술에 보편적인 코딩이 가능하도록 만들어진 기술입니다. **DApp**이라는 블록체인 기반 어플리케이션이 만들어질 수 있는 이유가 여기에 있습니다. 하지만 퍼블릭 블록체인이 상용화되기에는 크게 2가지 문제가 있습니다. 첫 째, 모든 정보를 모두에게 공개할 수만은 없다는 점. 둘 째, 구조적으로 블록이 쌓이는 속도의 성능 문제. 

많은 사람들이 이를 해결하고자, **Private Blockchain** 기술을 개발했으며, 그 중에서 IBM 에서 이끌고 있는 Hyperledger 재단의 **Fabric** 이라고 불리는 블록체인 기술이 기업 및 국책 과제의 가장 많은 선택을 받고 있습니다. 프라이빗 블록체인의 가장 큰 특징은 허가된 조직들만이 블록체인 네트워크에 참여하고 블록 정보를 열람할 수 있다는 것입니다.

오늘은 **Hyperledger Fabric (HLF)** 의 [공식 문서](https://hyperledger-fabric.readthedocs.io/en/release-2.2/whatis.html)를 참고하여 2.2버젼을 기준으로 설치하고 프라이빗 블록체인 네트워크를 개인 컴퓨터에 구동해보겠습니다! 😃 

<h2>Table of Contents</h2>

- [1. 개요](#1-개요)
- [2. Prerequisites](#2-prerequisites)
  - [2.1. Git](#21-git)
  - [2.2. curl](#22-curl)
  - [2.3. Docker](#23-docker)
  - [2.4. Docker Compose](#24-docker-compose)
- [3. Install Samples, Binaries, and Docker Images](#3-install-samples-binaries-and-docker-images)
  - [3.1. Samples 확인](#31-samples-확인)
  - [3.2. Binaries 확인](#32-binaries-확인)
  - [3.3. Docker Images 확인](#33-docker-images-확인)
- [4. test network](#4-test-network)
  - [4.1. 네트워크 컴포넌트 생성](#41-네트워크-컴포넌트-생성)
  - [4.2. 채널 생성](#42-채널-생성)

# 2. Prerequisites

HLF 블록체인 네트워크를 구성하기 위해 다양한 도구가 요구됩니다. 저는 우분투 OS에서 설치를 진행하겠습니다. 다른 운영체제를 사용하시는 분들은 [여기](https://hyperledger-fabric.readthedocs.io/en/release-2.2/prereqs.html)를 참고해주세요.

## 2.1. Git

설치

```console
$ sudo add-apt-repository ppa:git-core/ppa # 이 PPA는 최신 Git 버젼을 제공합니다.
$ sudo apt update
$ sudo apt install git
```

버젼 확인

```console
$ git --version
```

## 2.2. curl

설치

```liquid
$ sudo apt install curl
```

버젼 확인

```console
$ curl --version
```

## 2.3. Docker

설치

```console
$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

``root``가 아닌 유저도 ``Docker``를 사용 가능하게 그룹 추가

```console
$ sudo usermod -aG docker <username>
```

버젼 확인

```console
$ docker --version
```

도커 명령어 실행

```console
$ docker ps
```

<br>

❌아래와 같은 에러가 발생하는 경우❌

>Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.24/containers/json: dial unix /var/run/docker.sock: connect: permission denied

<details>
    <summary>해결 방법</summary>

<br>

``ls`` 명령어로 ``/var/run/docker.sock`` 파일의 권한을 살펴보면 그룹에 대한 실행 권한이 빠져있습니다. 그룹 내 사용자의 접근 가능하게 변경합니다.

```console
$ sudo chmod 666 /var/run/docker.sock
```

에러 상황부터 ``docker ps`` 명령어가 잘 작동하기까지 확인해보세요.

![docker_usergroup.png](/assets/img/screenshots/docker_usergroup.png)

</details>

<br>

<details>
    <summary>[재미로] 도커 hello-world 실행</summary>

```console
$ docker run hello-world
```

![docker_helloworld.png](/assets/img/screenshots/docker_helloworld.png)

hello-world 실행시 사용된 컨테이너/이미지 확인 및 삭제

```console
$ docker ps -a
$ docker rm $(docker ps -aq)
$ docker images
$ docker rmi $(docker images -aq)
```
</details>

<br>

## 2.4. Docker Compose

설치

```console
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose # 바이너리에 실행 권한 추가
```

Docker-compose 버젼 확인

```console
$ docker-compose --version
```

<br>

# 3. Install Samples, Binaries, and Docker Images

[가이드](https://hyperledger-fabric.readthedocs.io/en/release-2.2/install.html)따라 패브릭 샘플코드, 패브릭 네트워크 조작을 위한 실행 바이너리, 네트워크 구성에 사용되는 도커 이미지를 다운받겠습니다. 

저는 홈 디렉토리를 이용했습니다.

```console
$ cd ~
$ curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.2 1.4.9
```
- 현재 패브릭 2.2 버젼에서 배포된 최신 버젼은 ``fabric_version=2.2.2``와 ``fabric-ca_version=1.4.9``입니다.
- 버젼 정보를 입력하지 않으면 latest production release 기준으로 받아지게 됩니다.

<br>

## 3.1. Samples 확인

```console
$ cd ~/fabric-samples
$ ls
```

받아온 소스코드는 ``fabric-samples`` 폴더 안에 위치합니다. 여기서 ``fabcar``, ``test-network``, ``high-throughput`` 등 여러가지 폴더들에 실행해볼 수 있는 패브릭 블록체인 예제가 들어있습니다. 

![fabric_samples.png](/assets/img/screenshots/fabric_samples.png)

<br>

## 3.2. Binaries 확인

```console
$ cd ~/fabric-samples/bin
$ ls
```

![fabric_binaries.png](/assets/img/screenshots/fabric_binaries.png)

``bin`` 폴더에는 패브릭 네트워크 조작을 위해 쓰이는 바이너리 실행 파일들이들어있습니다. 어떤 디렉토리에서든 이 바이너리들을 실행할 수 있도록 ``PATH``를 ``export``해주면 좋습니다.

```console
$ export PATH=~/fabric-samples/bin:$PATH
```

<br>

## 3.3. Docker Images 확인

```console
$ docker images
```

패브릭 네트워크는 도커 컨테이너로 구동이 됩니다. 컨테이너를 생성할 때 사용되는 이미지들을 미리 다운받은 상태입니다. **TAG**를 확인해보면 설치할 때 입력했던 ``2.2.2`` 버젼인 것을 확인할 수 있습니다.

![fabric_docker_images.png](/assets/img/screenshots/fabric_docker_images.png)

<br>

# 4. test network

제공되는 샘플 중에서 ``test-network`` 를 이용하여 첫 패브릭 네트워크를 구성해봅시다! 😍

저희가 주목해야할 파일은 ``fabric-samples/test-network/network.sh`` 입니다. 블록체인 네트워크를 올리기 위한 명령어들을 스크립트로 작성했습니다. 스크립트에서 제공되는 옵션으로 손쉽게 네트워크를 올리거나, 올렸던 네트워크를 모두 초기화할 수도 있습니다. ``network.sh`` 스크립트 파일을 분석할 수 있다면 패브릭 블록체인 조작법을 거의 익혔다고 할 수 있습니다.

``network.sh``의 사용 설명서

```console
$ cd ~/fabric-samples/test-network
$ ./network.sh -h
```

```console
Usage:
  network.sh <Mode> [Flags]
    Modes:
      up - Bring up Fabric orderer and peer nodes. No channel is created
      up createChannel - Bring up fabric network with one channel
      createChannel - Create and join a channel after the network is created
      deployCC - Deploy a chaincode to a channel (defaults to asset-transfer-basic)
      down - Bring down the network

    Flags:
    Used with network.sh up, network.sh createChannel:
    -ca <use CAs> -  Use Certificate Authorities to generate network crypto material
    -c <channel name> - Name of channel to create (defaults to "mychannel")
    -s <dbtype> - Peer state database to deploy: goleveldb (default) or couchdb
    -r <max retry> - CLI times out after certain number of attempts (defaults to 5)
    -d <delay> - CLI delays for a certain number of seconds (defaults to 3)
    -i <imagetag> - Docker image tag of Fabric to deploy (defaults to "latest")
    -cai <ca_imagetag> - Docker image tag of Fabric CA to deploy (defaults to "latest")
    -verbose - Verbose mode

    Used with network.sh deployCC
    -c <channel name> - Name of channel to deploy chaincode to
    -ccn <name> - Chaincode name.
    -ccl <language> - Programming language of the chaincode to deploy: go, java, javascript, typescript
    -ccv <version>  - Chaincode version. 1.0 (default), v2, version3.x, etc
    -ccs <sequence>  - Chaincode definition sequence. Must be an integer, 1 (default), 2, 3, etc
    -ccp <path>  - File path to the chaincode.
    -ccep <policy>  - (Optional) Chaincode endorsement policy using signature policy syntax. The default policy requires an endorsement from Org1 and Org2
    -cccg <collection-config>  - (Optional) File path to private data collections configuration file
    -cci <fcn name>  - (Optional) Name of chaincode initialization function. When a function is provided, the execution of init will be requested and the function will be invoked.

    -h - Print this message

 Possible Mode and flag combinations
   up -ca -r -d -s -i -cai -verbose
   up createChannel -ca -c -r -d -s -i -cai -verbose
   createChannel -c -r -d -verbose
   deployCC -ccn -ccl -ccv -ccs -ccp -cci -r -d -verbose

 Examples:
   network.sh up createChannel -ca -c mychannel -s couchdb -i 2.0.0
   network.sh createChannel -c channelName
   network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript/ -ccl javascript
   network.sh deployCC -ccn mychaincode -ccp ./user/mychaincode -ccv 1 -ccl javascript
```

<br>

## 4.1. 네트워크 컴포넌트 생성

- test-network 에서 구성한 네트워크에는 다음과 같습니다.
  - 2개의 Organization
  - 각 Org에 1개씩 속한 Peer 노드
  - Raft 기반 1개의 Orderer 노드 

```console
$ ./network.sh up
$ docker ps
```

도커 컨테이너 목록을 살펴보면 위에서 설명한 노드마다 한개의 컨테이너, 그리고 네트워크를 조작하기 위한 ``cli`` 컨테이너가 구동중인 것을 확인할 수 있습니다.

![testnetwork_dockerps.png](/assets/img/screenshots/testnetwork_dockerps.png)

다음은 Peer 노드의 역할입니다.
- 블록체인 원장을 저장
- 원장에 저장하기 전 트랜잭션들 검증
- 스마트 컨트랙트 구동

모든 Peer는 Organization에 속해있어야 합니다. Peer 컨테이너의 ``NAMES``를 살펴보면 ``peer0.org1.exmaple.com`` 처럼 피어가 속한 Org를 명시하는 것이 관습입니다.

모든 패브릭 블록체인 네트워크는 Ordering 서비스를 담당하는 Orderer 그룹이 있어야 합니다. 지리적으로 떨어져잇는 Peer 노드들이 생성한 트랜잭션을 모아 순서를 정하고(합의 알고리즘을 통해) 블록을 생성하는 역할을 합니다. Ordering 서비스에서 전체 네트워크 트랜잭션의 병목 현상이 발생할 수 있습니다.

상용 서비스에서는 Orderer 노드를 여러개 구성해 Orderer Organization 간의 합의를 이루어야 합니다.

<br>

## 4.2. 채널 생성

이제 스크립트 파일을 통해 생성한 노드들로 채널을 구성합니다. **채널**의 특성은 다음과 같습니다.
- 특정 멤버들 간 프라이빗한 통신 레이어를 제공합니다. 
- 채널에 초대된 Org만이 채널을 사용할 수 있습니다. 초대되지 않은 멤버들에게는 네트워크에서 보이지 않습니다.
- 각 채널은 독립된 블록체인 원장을 소지합니다.
- 채널에 초대된 Org는 해당 Org에 속한 Peer 노드들을 채널에 **join**하도록 합니다.

**Org1**과 **Org2** 사이에 채널(**mychannel**)을 생성하고, Peer 들을 참여시켜봅시다.

```console
$ ./network.sh createChannel -c mychannel
```

<br>

로그를 분석해 채널 생성 과정을 파헤쳐보겠습니다.💥


1. 

```console
configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/mychannel.tx -channelID mychannel
```

- ``configtxgen`` 실행파일을 이용해 채널생성을 위한 트랜잭션 파일(**mychannel.tx**)을 생성합니다.
- 채널명(**mychannel**)을 설정하고, ``test-network/configtx/configtx.yaml``에 정의된 **TwoOrgsChannel**의 세부구성을 참조합니다.

2. ```console
   peer channel create -o localhost:7050 -c mychannel --ordererTLSHostnameOverride orderer.example.com -f ./channel-artifacts/mychannel.tx --outputBlock ./channel-artifacts/mychannel.block --tls --cafile /home/fabric/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
   ```
- ``peer`` 실행파일을 이용해 채널을 생성합니다. 
- 앞서 생성한 트랜잭션 파일을 사용하고, 채널생성 블록파일(**mychannel.block**)을 ``/channel-artifacts/`` 폴더에 생성합니다.
- 이 과정에서 **tls**통신을 위해 ``test-network/organizations`` 폴더에 있는 인증서를 사용합니다.

✅ **채널이 생성되었습니다.**

3. ```console
   peer channel join -b ./channel-artifacts/mychannel.block
   ```
- 환경변수 활용해 **Org1**과 **Org2**의 Peer들을 각각 채널에 합류시킵니다.

4. ```console
   peer channel fetch config config_block.pb -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
   2021-02-03 07:41:28.310 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
   2021-02-03 07:41:28.312 UTC [cli.common] readBlock -> INFO 002 Received block: 0
   2021-02-03 07:41:28.313 UTC [channelCmd] fetch -> INFO 003 Retrieving last config block: 0
   2021-02-03 07:41:28.315 UTC [cli.common] readBlock -> INFO 004 Received block: 0
   Decoding config block to JSON and isolating config to Org1MSPconfig.json
   ```


   + configtxlator proto_decode --input config_block.pb --type common.Block
   + jq '.data.data[0].payload.data.config'
   Generating anchor peer update transaction for Org1 on channel mychannel
   + jq '.channel_group.groups.Application.groups.Org1MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org1.example.com","port": 7051}]},"version": "0"}}' Org1MSPconfig.json
   + configtxlator proto_encode --input Org1MSPconfig.json --type common.Config
   + configtxlator proto_encode --input Org1MSPmodified_config.json --type common.Config
   + configtxlator compute_update --channel_id mychannel --original original_config.pb --updated modified_config.pb
   + configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate
   + jq .
   ++ cat config_update.json
   + echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":{' '"channel_id":' '"mychannel",' '"isolated_data":' '{},' '"read_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org1MSP":' '{' '"groups":' '{},' '"mod_policy":' '"",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"1"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '},' '"write_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org1MSP":' '{' '"groups":' '{},' '"mod_policy":' '"Admins",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"AnchorPeers":' '{' '"mod_policy":' '"Admins",' '"value":' '{' '"anchor_peers":' '[' '{' '"host":' '"peer0.org1.example.com",' '"port":' 7051 '}' ']' '},' '"version":' '"0"' '},' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"1"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"1"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '}}}}'
   + configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope
   ```
- 채널에서 **protobuf 데이터** 형식인 config_block.pb`` protobuf 데이터를


채널 이름을 바꿔 여러개의 채널을 생성할 수 있습니다.