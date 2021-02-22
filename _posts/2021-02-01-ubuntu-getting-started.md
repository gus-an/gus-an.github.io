---
title: Oracle VM VirtualBox로 Ubuntu 설치하기
author: gus
date: 2021-02-01 14:46:00 +0900 1
categories: [Linux, Ubuntu]
tags: [ubuntu, virtualbox]
toc: true
---

이번 포스팅에서는 **VirtualBox** 가상화 머신을 활용하여 윈도우 위에 격리된 운영체제 **Ubuntu**를 설치해보겠습니다.

우분투는 하나의 운영체제이기 때문에, 독립적인 컴퓨터(CPU, 메모리, 네트워크 등)가 필요합니다. 윈도우를 지우고 우분투 OS를 설치할 수도 있겠지만 컴퓨터가 한대 뿐이고 윈도우를 써야하기 때문에 **VirtualBox**의 도움을 받아야 합니다. 😂

<h2>Table of Contents</h2>

- [1. 설치 과정](#1-설치-과정)
  - [1.1. Oracle VirtualBox 설치](#11-oracle-virtualbox-설치)
  - [1.2. Ubuntu 이미지 다운로드](#12-ubuntu-이미지-다운로드)
  - [1.3. 가상 머신 만들기](#13-가상-머신-만들기)
  - [1.4. 가상 머신 설정](#14-가상-머신-설정)
    - [1.4.1. 시스템 > 프로세서 개수 설정](#141-시스템--프로세서-개수-설정)
    - [1.4.2. 디스플레이 > 비디오 메모리 설정](#142-디스플레이--비디오-메모리-설정)
    - [1.4.3. 저장소 > 디스크 이미지 삽입](#143-저장소--디스크-이미지-삽입)
    - [1.4.4. 공유폴더 생성](#144-공유폴더-생성)
  - [1.5. Ubuntu 운영체제 설치](#15-ubuntu-운영체제-설치)
- [2. 추가 작업](#2-추가-작업)
  - [2.1. 공유 폴더 생성](#21-공유-폴더-생성)
    - [2.1.1. 게스트 확장 이미지 삽입](#211-게스트-확장-이미지-삽입)
    - [2.1.2. 게스트 확장 이미지 실행](#212-게스트-확장-이미지-실행)
    - [2.1.3. 마운트 폴더 생성 및 마운트](#213-마운트-폴더-생성-및-마운트)
    - [2.1.4. 재부팅시 자동 마운트](#214-재부팅시-자동-마운트)
  - [2.2. 초기 상태 저장](#22-초기-상태-저장)
- [3. 이제 뭐하지?](#3-이제-뭐하지)

<br>

# 1. 설치 과정
Oracle에서 제공하는 가상머신 프로그램을 설치합니다. 

## 1.1. Oracle VirtualBox 설치

[Oracle VirtualBox 공식 사이트](https://www.virtualbox.org/)에서 설치 파일을 다운
받아 설치합니다.


![virtualblox_startup.png](/assets/img/screenshots/virtualblox_startup.png)

*설치 완료 후 정상적으로 실행된 화면*

## 1.2. Ubuntu 이미지 다운로드

우분투 18.04 LTS 버젼을 사용하겠습니다. [공식 사이트](https://releases.ubuntu.com/18.04/)에 방문하면 **Desktop image**와 **Server install image** 두 종류의 이미지가 제공됩니다. 둘의 차이점은 윈도우처럼 그래픽 UI가 제공되는지 여부에 있고, 당연히 데스크탑 버젼이 더 무겁습니다. 서버 프로그래밍으로는 그래픽 UI가 필요하지 않기 때문에, **64-bit PC (AMD64) server install image** 를 클릭하여 다운로드 합니다.

![ubuntu_website.png](/assets/img/screenshots/ubuntu_website.png)

## 1.3. 가상 머신 만들기

설치한 VirtualBox 에 우분투 이미지를 이용하여 가상 머신을 생성하겠습니다.

1. **새로 만들기** > 내용 입력 > **다음**
   - 이름: 자유롭게
   - 머신 폴더: 가상머신 이미지 저장 위치
   - 종류: Linux
   - 버젼: Ubuntu(64-bit)

![virtualbox_new.png](/assets/img/screenshots/virtualbox_new.png)

2. 메모리 크기: 1GB > **다음**
   - 메모리 크기는 구동하려는 서비스에 따라 더 많이 필요할 수 있습니다.
   - 가상머신 생성 후에 조정할 수 있습니다!
3. 하드 디스크: **지금 새 가상 하드 디스크 만들기** > **다음**
4. 하드 디스크 파일 종류: **VDI** > **다음**
5. 물리적 하드 드라이브: **고정 크기** > **만들기**
   - 고정 크기로 설정하면 설정 용량만큼 컴퓨터 자원을 미리 할당받습니다.
   - 동적 크기로 설정하면 필요한 만큼 자원을 확보하게 되지만 고정 크기보다 느립니다.
   - 하드 드라이브는 한번 생성하면 변경하기 복잡하기 때문에, 필요한 만큼 설정합니다.

## 1.4. 가상 머신 설정

이제 생성된 가상 머신을 실행하고, 우분투 운영체제를 설치하겠습니다. VirtualBox에서는 가상 컴퓨터의 여러가지 옵션을 수시로 바꿀 수 있는 기능을 지원합니다. 간단히 몇 가지만 설정하고 진행하겠습니다. 생성된 가상 머신을 우클릭 하여 설정으로 들어갑니다.

![virtualbox_setting.png](/assets/img/screenshots/virtualbox_setting.png)

### 1.4.1. 시스템 > 프로세서 개수 설정

기본적으로 1로 세팅되어 있는 프로세서 개수를 초록 범위에서 허락하는 개수로 변경합니다.

![virtualbox_processor.png](/assets/img/screenshots/virtualbox_processor.png)

### 1.4.2. 디스플레이 > 비디오 메모리 설정

비디오 메모리도 기본적으로 16MB로 잡혀있기 때문에, 최대한으로 설정합니다.

![virtualbox_video.png](/assets/img/screenshots/virtualbox_video.png)

### 1.4.3. 저장소 > 디스크 이미지 삽입

이전에 다운로드한 우분투 이미지 파일은 가상 머신의 디스크로 입력합니다. 컴퓨터에 운영체제 설치 CD를 삽입하는 것과 같다고 생각합니다. 😃 

**저장소** 탭으로 이동하여, 디스크모양에 초록색 플러스 표시(광학 드라이브 추가)를 클릭합니다.

![virtualbox_adddisk.png](/assets/img/screenshots/virtualbox_adddisk.png)

**추가** 버튼을 클릭하고, 다운받은 우분투 이미지 파일을 찾아 **열기** > **선택** 순으로 버튼을 클릭합니다.

![virtualbox_diskdone.png](/assets/img/screenshots/virtualbox_diskdone.png)

### 1.4.4. 공유폴더 생성

가상 머신이 구동되면 독립된 파일 시스템을 갖게 됩니다. 우분투 기반이겠죠? 그러면 사용의 편의상 호스트 컴퓨터(윈도우)와 공유되는 폴더가 지정되면 좋습니다. 저희가 생성한 우분투 서버는 GUI가 지원되지 않기 때문에, 예를들어 웹에서 이미지를 검색하여 사용하고 싶다면 호스트 컴퓨터에서 공유 폴더에 저장하고 게스트 컴퓨터(우분투)에서 참조하는 식으로 사용합니다. 물론 파일은 양방향으로 쓰기가 가능합니다. 😃

**공유 폴더** 탭으로 이동하여 초록색 플러스(새 공유폴더 추가)를 클릭합니다.
- 폴더 경로: 저는 ``VirtualBox_VMs/shared/{가상머신 이름}`` 으로 폴더를 생성했습니다.
- 자동 마운트: 마운트 지점을 설정하기 않을 것이기 때문에 무시
- 마운트 지점: 설정에서 입력하면 해당 폴더의 권한 그룹이 vboxsf로 지정되기 때문에 무시
- 항상 사용하기 ✔: 리부팅 되어도 사용하기 위함
  
![virtualbox_mount.png](/assets/img/screenshots/virtualbox_mount.png)


**확인** > **확인** 버튼을 클릭하여 모든 설정을 적용합니다!

## 1.5. Ubuntu 운영체제 설치

VirtualBox 홈 화면에서 실행할 가상머신을 선택하고 **시작** 버튼을 클릭합니다. 본격적으로 우분투 운영체제 설치를 시작하겠습니다. 설치 중에 설정은 거의 기본 상태로 진행합니다.

![ubuntu_setup_1.png](/assets/img/screenshots/ubuntu_setup_1.png)
![ubuntu_setup_2.png](/assets/img/screenshots/ubuntu_setup_2.png)
![ubuntu_setup_3.png](/assets/img/screenshots/ubuntu_setup_3.png)
![ubuntu_setup_4.png](/assets/img/screenshots/ubuntu_setup_4.png)
![ubuntu_setup_5.png](/assets/img/screenshots/ubuntu_setup_5.png)

**Mirror address**는 우분투 관련 패키지를 받아오는 소스입니다. 우분투에서 ``apt-get`` 명령어를 통해 패키지를 받아올때 이 주소를 바라봅니다. 기본 설정인 ``kr.archive.ubuntu.com`` 도 한국이기 때문에 사용하기에 지장이 없으나, 속도면에서 좀 더 믿음이 가는 카카오 미러 서버로 변경합니다. 운영체제 설치할 때도 이 주소에서 많은 다운로드가 이뤄지기 때문에 체감 속도차이는 상당히 큽니다!! 😨

![ubuntu_setup_6.png](/assets/img/screenshots/ubuntu_setup_6.png)
![ubuntu_setup_7.png](/assets/img/screenshots/ubuntu_setup_7.png)
![ubuntu_setup_8.png](/assets/img/screenshots/ubuntu_setup_8.png)
![ubuntu_setup_9.png](/assets/img/screenshots/ubuntu_setup_9.png)

이름 설정은 기억하기 쉽게 저는 비밀번호 포함 모두 ``{가상머신 이름}`` 으로 지정했습니다.

![ubuntu_setup_10.png](/assets/img/screenshots/ubuntu_setup_10.png)

**OpenSSH** 설정은 가상머신에 SSH 통신으로 접속하기 위해 설정해줍니다. **SSH identity** 는 설정은 지금은 넘어가겠습니다. [다음 포스팅](/posts/ssh)을 참고해주세요. 

<details>
   <summary>
      SSH 란❔
   </summary>
      SSH 는 서버 컴퓨터 원격제어를 위한 통신 프로토콜이라고 생각하면 됩니다. 보안상 안전한 방법으로 shell에 접근하여 컴퓨터를 조작하고, 파일을 전송할 수 있습니다.
</details>

<br>

![ubuntu_setup_11.png](/assets/img/screenshots/ubuntu_setup_11.png)

프로그램 설치 도구 **Snap** 이용한 설치는 아무것도 하지 않고 넘어갑니다.

![ubuntu_setup_12.png](/assets/img/screenshots/ubuntu_setup_12.png)

앞서 미러 사이트를 카카오로 설정해줬기 때문에, 빠른 시간 안에 설치가 끝나길 기대합니다!
> *downloading and installing security updates* 에서 시간이 많이 소모되네요..🤔
> <br>
> *Cancel update and reboot* 버튼으로 진행할 수 있지만 기다려줍시다..

설치가 끝났습니다! **Reboot**으로 재기동 합니다.

![ubuntu_setup_13.png](/assets/img/screenshots/ubuntu_setup_13.png)

재기동 하면, 다음과 같이 설치 이미지를 제거하고 진행하라는 메시지가 나옵니다. 

![ubuntu_setup_14.png](/assets/img/screenshots/ubuntu_setup_14.png)

설치 전에 했던 것처럼 VirtualBox 설정에서 **저장소** 를 확인해보면 이미 제거된 것을 알 수 있습니다. ``Enter``로 진행합니다.

![ubuntu_setup_15.png](/assets/img/screenshots/ubuntu_setup_15.png)

반응이 없는것 같을 떄 ``Enter``를 치면 로그인 아이디를 묻는 메시지가 나옵니다. 아이디 / 비밀번호를 입력하여 진행합니다.

![ubuntu_setup_16.png](/assets/img/screenshots/ubuntu_setup_16.png)

드디어 모든 설치가 끝났습니다!!

<br>

# 2. 추가 작업

## 2.1. 공유 폴더 생성

가상머신 설정에서 공유폴더를 생성했는데요, 아직 적용이 완료되지는 않았습니다. VirtualBox를 이용해서 우분투를 설치할 때, 우분투에서 VirtualBox와 상호작용하기 위해 디스크 이미지를 이용한 추가적인 설치를 합니다. 공유 폴더 또한 이 과정을 통해 작업할 수 있습니다.

### 2.1.1. 게스트 확장 이미지 삽입

![ubuntu_guest_image.png](/assets/img/screenshots/ubuntu_guest_image.png)

### 2.1.2. 게스트 확장 이미지 실행

1. 이미지를 실행하기 위한 패키지 설치
```console
$ sudo apt-get update
$ sudo apt-get install build-essential linux-headers-$(uname -r)
```
2. 삽입한 이미지의 위치 후보군 출력
```console
$ ls /dev/sr*
```
3. 삽입한 이미지를 마운트 위치에 마운트
```console
$ sudo mount /dev/sr1 /mnt/
```
4. 마운트한 이미지 실행 후 리부팅
```console
$ cd /mnt
$ ls # 마운트된 이미지들 확인
$ sudo ./VBoxLinuxAdditions.run
$ sudo reboot
```

### 2.1.3. 마운트 폴더 생성 및 마운트
1. 우분투 내 마운트 지점 생성
```console
$ sudo mkdir -p /path/to/directory/ # 저는 /usr/local/fabric 이라고 지었습니다.
```
2. 생성된 디렉토리의 소유자 변경
```console
$ sudo chown -R {소유자}:{그룹} /usr/local/fabric # 저의 경우 fabric:fabric 
```
3. 호스트 윈도우와 게스트 우분투 파일시스템 마운트
```console
$ sudo mount -t vboxsf {마운트이름} /usr/local/fabric # 저의 경우 fabric
```
- ``-t`` (type): 마운트할 type을 vboxsf 로 지정
- ``마운트이름``: 설정에서 입력한 **폴더 이름**과 동일하게 입력
1. 확인
```console
$ cd /usr/local/fabric
$ touch example.txt
```
- 윈도우의 마운트 폴더에 가보면 ``example.txt`` 가 생성된 것을 확인할 수 있습니다.

### 2.1.4. 재부팅시 자동 마운트

1. ``/etc/fstab`` 정보 변경
```console
$ sudo vi /etc/fstab
```
- ``fstab``: 리눅스 파일시스템 정보를 정적으로 저장하는 파일입니다.
2. 파일 가장 아래에 한줄 추가 후 저장
```
fabric /usr/local/fabric vboxsf defaults 0 0
```
3. 재부팅 후 마운트 잘 되어있는지 확인
```
$ sudo reboot
$ cd /usr/local/fabric
```

## 2.2. 초기 상태 저장

여기까지 오라클의 VirtualBox를 이용하여 우분투 OS 설치과정을 마쳤습니다! 이 쯤에서 VirtualBox에서 지원하는 **스냅샷**을 이용하면 좋은데요, 현 시점의 시스템 상태를 저장하고 언제든지 복구할 수 있게 됩니다! 

VirtualBox 홈 화면에서 **찍기** 버튼을 클릭합니다.
- 스냅샷 이름과 간단한 설명을 추가합니다.

![virtualbox_snapshot_take.png](/assets/img/screenshots/virtualbox_snapshot_take.png)

홈 화면에서 내가 찍은 스냅샷을 타임라인처럼 확인할 수 있습니다. 복잡한 시스템을 설치할수록 중간 중간 스냅샷을 찍는 것을 적극 권장합니다. 😃

![virtualbox_snapshot_list.png](/assets/img/screenshots/virtualbox_snapshot_list.png)

<br>

# 3. 이제 뭐하지?

- [서버용 컴퓨터가 처음이라면 명령어부터 알아보기](/posts/linux-commands)
- [원격 접속으로 서버 컴퓨터 편하게 사용하기](/posts/ssh)
<!-- - 뭐니뭐니해도 알고리즘부터! (Jupyter를 이용한 알고리즘 풀이 환경 완성) -->
- [기업용 블록체인 네트워크 구축해보기](/posts/fabric-getting-started)