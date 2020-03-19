# Installation

DCF CLI는 아래 두 가지 방법으로 설치할 수 있다.

1. 바이너리 파일 다운로드
2. 소스코드 컴파일



## Donwload Binary

```bash
$ wget https://github.com/DigitalCompanion-KETI/DCFramework/releases/download/v1.0.0/dcf-cli
$ sudo chmod 777 dcf-cli
$ mv dcf-cli /usr/bin
```



## Compile from Source Code



### Clone pb from GitHub

먼저 Go 패키지 관련 디렉토리가 있는지 확인하고, 없다면 디렉토리를 생성한다.

```bash
$ mkdir -p $GOPATH/src/github.com/digitalcompanion-keti
```



Digital Companion Framework의 pb를 다운로드한다.

```bash
$ cd $GOPATH/src/github.com/digitalcompanion-keti
$ wget https://github.com/DigitalCompanion-KETI/DCFramework/releases/download/v1.0.0/pb-master.zip
$ unzip pb-master.zip
$ mv pb-master pb
$ sudo chown -R $USER pb
```



### Clone CLI from GitHub

Digital Companion Framework의 CLI를 다운로드한다.

```bash
$ cd $GOPATH/src/github.com/digitalcompanion-keti
$ wget https://github.com/DigitalCompanion-KETI/DCFramework/releases/download/v1.0.1/dcf-cli-master.zip
$ unzip dcf-cli-master.zip
$ mv dcf-cli-master dcf-cli
$ sudo chown -R $USER dcf-cli
```



### Build CLI

아래 명령어를 이용하여 CLI를 빌드한다.

```bash
$ cd dcf-cli
$ go build
$ go install
```



## Configure Private Docker Registry

Digital Companion Framework의 사설 도커 저장소에 로그인하기 위한 설정을 진행한다.

아래 예제에서는 전자부품연구원에서 제공하는 사설 도커 저장소의 IP를 입력한다. 만약 사용자의 컴퓨팅 환경에서 사설 도커 저장소를 구축하였다면 사용자의 사설 도커 저장소의 IP 정보를 입력한다.



### Insecure registry

Docker의 `daemon.json`파일을 아래와 같이 작성한다.

```bash
$ sudo vim /etc/docker/daemon.json
```

```json
{
    "insecure-registries": ["keti.asuscomm.com:5001"]
}
```



Docker를 재시작한다.

```bash
$ sudo service docker restart
```



Insecure registry 등록이 잘 되었는지 확인한다.

```bash
$ sudo docker info
>>
Insecure Registries:
 keti.asuscomm.com:5001
```



### Login Docker Registry

Digital Companion Framework의 사설 도커 저장소에 로그인한다.

```bash
$ sudo docker login keti.asuscomm.com:5001
>>>
Username: elwlxjfehdqkswk
Password: elwlxjfehdqkswk
```



