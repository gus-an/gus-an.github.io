---
title: Jupyter Notebook + Ngrok으로 어디서든 서버 접근
author: gus
date: 2021-02-22 10:57:00 +0900 1
categories: [Python, Jupyter Notebook]
tags: [jupyter, ngrok]
toc: true
---

**Jupyter Notebook**은 파이썬 코딩을 위해 개발된 아주 편리한 웹 베이스 도구입니다. 코드를 줄 단위로 지정하여 실행해 볼 수 있습니다! 신박하죠?😃 나아가 Jupyter를 이용하면 어디에서든 서버 컴퓨터에 접근하여 코드작성 및 관리할 수 있는 환경을 구축할 수 있습니다. 저는 핸드폰으로도 제 서버에 접근하여 알고리즘 문제를 풀 수 있도록 이 기능을 선택했습니다.

- **이 가이드는 Ubuntu 운영체제 기반입니다**

<h2>Table of Contents</h2>

- [1. 설치](#1-설치)
  - [1.1. Python3](#11-python3)
  - [1.2. Python3-pip](#12-python3-pip)
  - [1.3. Jupyter Notebook](#13-jupyter-notebook)
- [2. Jupyter Notebook 설정](#2-jupyter-notebook-설정)
  - [2.1. 비밀번호 생성](#21-비밀번호-생성)
  - [2.2. HTTPS 통신 설정](#22-https-통신-설정)
  - [2.3. config파일 설정](#23-config파일-설정)
- [3. Jupyter 실행](#3-jupyter-실행)
- [4. Jupyter 사용](#4-jupyter-사용)
  - [4.1. 접속](#41-접속)
  - [4.2. 기능 1. Python 3](#42-기능-1-python-3)
  - [4.3. 기능 2. Terminal](#43-기능-2-terminal)
  - [4.4. 기능 3. 디렉토리 관리](#44-기능-3-디렉토리-관리)
- [5. 심화: ngrok 적용](#5-심화-ngrok-적용)
  - [5.1. ngrok 설치](#51-ngrok-설치)
  - [5.2. ngrok 설정](#52-ngrok-설정)
  - [5.3. ngrok 실행](#53-ngrok-실행)
  - [5.4. background 로 실행](#54-background-로-실행)

<br>

# 1. 설치

우선 Jupyter를 설치하기 위해 필요한 도구들을 설치합니다.

```console
$ sudo apt-get update
```

## 1.1. Python3

```console
$ sudo apt-get install python3
``` 

우분투 서버의 경우 python3는 기본적으로 설치가 되어있습니다. 버젼을 확인하세요.

```console
$ python3 --version
```

## 1.2. Python3-pip

파이썬 패키지 인스톨러인 pip 을 설치합니다.

```console
$ sudo apt-get install python3-pip
```

## 1.3. Jupyter Notebook

pip 을 이용해 Jupyter Notebook을 설치합니다.

```console
$ sudo pip3 install notebook
```

<br> 

# 2. Jupyter Notebook 설정

pip 을 통해 파이썬에서 Jupyter를 사용할 준비가 되었습니다. 파이썬을 실행하여 Jupyter에 대한 설정을 진행합니다.


## 2.1. 비밀번호 생성

파이썬을 켜고 Jupyter 노트북에서 사용할 비밀번호를 생성합니다.

```console
$ python3
```

```console
>>> from notebook.auth import passwd
>>> passwd()
>>> exit()
```

입력창을 통해 비밀번호를 입력하면, 비밀번호 해싱 알고리즘인 `argon2`(또는 `sha1`)에 대한 출력이 나옵니다. 이 해시 값을 기억할 수 있도록 저장해 둡니다.

## 2.2. HTTPS 통신 설정

구동중인 Jupyter는 브라우저로 접속하게 됩니다. HTTP 통신을하게 되면 패킷을 가로챌 경우 저희 서버에 대한 정보가 노출될 위험이 있습니다. 보안을 강화하기 위해 사설 인증서을 생성하여 HTTPS 설정에 입력해주도록 합니다.

```console
$ mkdir ssl && cd ssl
$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout "cert.key" -out "cert.pem" -batch
```
- `-days`: 유효기간을 1년으로 생성합니다

위 명령어를 실행하면, 해당 폴더에 인증 파일인 `cert.key`와 `cert.pem`이 생성된 것을 확인할 수 있습니다.

## 2.3. config파일 설정

다음으로 Jupyter 환경설정 파일을 생성합니다.

```console
$ jupyter notebook --generate-config
```

그러면 생성된 환경설정 파일에 대한 경로가 출력이 됩니다. 해당 파일에 아래 정보를 입력하고 저장합니다.

```
c = get_config()
c.NotebookApp.ip = '0.0.0.0'
c.NotebookApp.port = 8888
c.NotebookApp.password = <복사한 비밀번호 해시 값>
c.NotebookApp.open_browser = False
c.NotebookApp.certfile = <사설인증서 .pem 파일의 절대경로>
c.NotebookApp.keyfile = <사설인증서 .key 파일의 절대경로>
```

- `ip`: IP를 `0.0.0.0`으로 설정하면, 어떤 곳에서든 접근이 가능합니다.
- `password`: 이 위치에 위에서 생성한 `argon2` 해시값을 복사합니다.
- `open_browser`: Jupyter 실행 시 서버에서 브라우저 열기를 해제합니다. 다른 브라우저를 통해 구동된 Jupyter 노트북에 접근은 여전히 가능합니다.
- `certfile`, `keyfile`: `<절대 경로>`는 폴더에서 `pwd`명령어 입력 후 `/파일명`까지 붙이면 됩니다.

<br>

# 3. Jupyter 실행
```
$ cd ~
$ sudo jupyter-notebook --allow-root
```
- `cd ~`: Jupyter 노트북을 실행시킬 디렉토리로 이동합니다. 해당 디렉토리가 Jupyter 노트북에서 접근 가능한 최상위 디렉토리가 됩니다.
- `--allow-root`: `sudo` 권한으로 실행하기 위해 입력합니다.

위 명령어를 통해 Jupyter 로그가 출력되면, 쉘이 종료되어도 프로세스가 유지될 수 있게 다음과 같이 입력합니다.

``` 
[ctrl + z]
$ bg
$ disown -h
```

이제 실행중이던 쉘을 종료해도 Jupyter Notebook에 접근할 수 있습니다.

<br>

# 4. Jupyter 사용

## 4.1. 접속

VM위에 Ubuntu를 구동하고 있는 경우, 쥬피터 접근을 위한 `8888`번 포트를 포트포워딩 해줍니다. 

크롬 브라우저에 `https://localhost:8888`로 접속하여 비밀번호를 입력합니다.

![login.png](/assets/img/screenshots/jupyter/login.png)

제 Jupyter 노트북에 접근된 화면입니다. 키파일을 생성하기 위한 `ssl`폴더가 눈에 띕니다.

## 4.2. 기능 1. Python 3

![new.png](/assets/img/screenshots/jupyter/new.png)

`New` > `Python 3` 를 클릭하면 현재 위치한 폴더에 새로운 Jupyter Notebook이 생성됩니다. 이렇게 열린 새로운 창이 파이썬 코딩하기에 좋은 환경입니다. 자세한 사용법은 따로 소개하겠습니다.

![python3.png](/assets/img/screenshots/jupyter/python3.png)

새로운 노트북에 `print('hi jupyter!')`를 입력하고 `shift + enter`를 입력한 모습입니다.

## 4.3. 기능 2. Terminal

`New` > `Terminal`를 클릭하면 역시 새로운 창에서 서버에 접근 가능한 쉘 화면이 생성됩니다. 여기서 컴퓨터를 관리할 수 있습니다.

![terminal.png](/assets/img/screenshots/jupyter/terminal.png)

## 4.4. 기능 3. 디렉토리 관리

그 외에도 `New` > `Text File`이나 `New` > `Folder`를 선택하면 원하는 방식대로 편하게 웹 상에서 서버의 파일시스템을 관리할 수 있습니다.

그 외 파일에 대한 변경 하상은, 좌측 체크박스를 선택하면 선택지가 보입니다.

![rename.png](/assets/img/screenshots/jupyter/rename.png)

생성한 노트북에 대한 옵션은 아래와 같습니다.

![notebook_option.png](/assets/img/screenshots/jupyter/notebook_option.png)

<br> 

# 5. 심화: ngrok 적용

**Ngrok**은 로컬호스트에 터널링을 도와주는 프로그램입니다.

제가 Jupyter Notebook을 설치한 최종 목표는 **아무데서나, 심지어 모바일에서도 내 서버에 접속하기 위함**이었습니다. 저처럼 윈도우 > VM > 우분투 > Jupyter Notebook 과 같은 환경을 설정했다면, 저의 윈도우 IP로부터 접근해야 합니다. 가정집의 경우 공유기 세팅을 통해 포트포워딩 설정을 하면 이런 접근 권한을 설정할 수 있습니다. 윈도우의 외부 IP 가 궁금하다면 `powershell`을 열고 `curl ifconfig.me`를 입력하면 알 수 있습니다.

하지만 공유기 설정에 접근할 수 없는 경우라면 지금 소개할 **ngrok** 프로그램을 활용할 수 있습니다. (**ngrok**실행을 막아놓은 네트워크 망이 아니라면 말이죠)

## 5.1. ngrok 설치

[공식 사이트](https://dashboard.ngrok.com/get-started/setup)에서 로그인 한 후 [리눅스 ngrok 다운로드 링크](https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip)를 복사합니다.

우분투에서 다음 명령어를 실행합니다.

```console
$ sudo apt-get install unzip
$ wget https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
$ unzip ngrok-stable-linux-amd64.zip
```

## 5.2. ngrok 설정

ngrok 다운로드 사이트에 로그인 했을 때, **2. Connect your account** 라는 설정을 따라해주면 좋습니다

```console
$ ./ngrok authtoken <토큰 값>
```

## 5.3. ngrok 실행

`ngrok` 명령어는 원래 아래와 같이 적으면 충분합니다.

```console
$ ./ngrok http 8888
```

그러나 우리가 실행한 Jupyter에선 ssl 설정을 해줬기 때문에 정상적인 HTTP 통신이 이뤄지지 않는 것을 경험했습니다. [여기](https://github.com/inconshreveable/ngrok/issues/475)를 참고하여 다음과 같은 명령어로 입력하니 정상적으로 동작합니다.

```console
$ ./ngrok http https://localhost:8888 -host-header="localhost:8888"
```

혹은 [여기](https://github.com/assets/img/screenshots/jupyter/notebook/issues/507)에선 `openssl` 생성시 다음과 같이 `-keyout` 옵션에서 `.pem` 형식으로 입력하면 된다고도 합니다.

```console
$ $sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout mycert.pem -out mycert.pem
```

ngrok 이 실행중인 화면입니다.

![ngrok.png](/assets/img/screenshots/jupyter/ngrok.png)

**https**로 시작하는 주소를 복사하여 어디에서든 브라우저로 접근하면 우리의 Jupyter 노트북에 접근이 가능합니다.

![ngrok_browser.png](/assets/img/screenshots/jupyter/ngrok_browser.png)

## 5.4. background 로 실행

**ngrok**도 마찬가지로 쉘을 닫아도 실행하기 위해 [다음](https://stackoverflow.com/questions/27162552/ngrok-running-in-background)을 참고했습니다. 

```console
$ ./ngrok http https://localhost:8888 -host-header="localhost:8888" > /dev/null &
$ disown -h
```

그리고 api 주소를 확인하기 위해 다음을 입력합니다.

```console
$ curl http://127.0.0.1:4040/api/tunnels
```

**ngrok**이 활성화되어있는 api 주소 정보를 출력합니다. ``https``로 시작되는 주소를 찾아 활용하세요!






