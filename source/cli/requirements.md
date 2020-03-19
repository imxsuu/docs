# Requirements

CLI를 설치하기 위해서는 아래와 같은 의존성 소프트웨어가 설치되어 있어야 한다.



- [Docker >= 19.03](https://docs.docker.com/v17.09/engine/installation/linux/docker-ce/ubuntu/)

- [Nvidia-Docker2](https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0))

  Nvidia-Docker2는 호스트 컴퓨터의 그래픽 드라이버 버전에 따라서 CUDA 버전의 제한이 있으니, 아래 링크를 참고하여 그래픽 드라이버를 업데이트한다. 호스트 컴퓨터의 그래픽 드라이버는 `nvidia-smi`명령어로 확인할 수 있다.

  [CUDA toolkit 을 사용하기 위해 요구되는 최소 Driver 버전과 GPU 아키텍쳐](https://github.com/NVIDIA/nvidia-docker/wiki/CUDA#requirements)

- Golang

  DCF CLI를 소스코드에서 컴파일하게 될 경우 Golang이 설치되어 있어야 한다.



## Install Golang

공식 Go 다운로드 홈페이지](https://golang.org/doc/install)에서 자신의 환경에 맞게 설치파일을 다운로드한다.

설치파일을 압축 해제하고, `go` 폴더를 `/usr/local`로 옮긴다.

```bash
$ sudo tar -xvf go1.12.5.linux-amd64.tar.gz
$ sudo mv go /usr/local
```



`~/.bashrc`파일을 수정하여 환경변수 설정을 진행 및 적용한다.

```bash
$ vim ~/.bashrc
>>
# add this lines
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH

$ source ~/.bashrc
```



환경 변수 설정을 완료했다면 Go 설치를 확인한다.

```bash
$ go version
$ go env
```
