# Function Component

DCF는 마이크로 아키텍처 서비스 기반 프레임워크이며 인공지능 모델 서빙(Serving)을 지원하는 목적으로 만들어졌다. 인공지능 모델 서빙(Serving) 시 기본 배포 단위는 함수(Function)이다. 함수(Function)는 두 가지 컴포넌트로 구성된다.

1. DCF와 데이터 및 함수 컨테이너의 정보 수집 역할을 하는 와쳐(Watcher)
2. 사용자가 정의한 인공지능 모델 함수(Funtion)



DCF는 기본적으로 함수(Funtion) 컨테이너에 데이터를 전송하고 정보를 수집하는 와쳐(Watcher)를 제공한다. 따라서 사용자는 오로지 **인공지능 모델 함수**만 작성하면 된다.



본 챕터에서는 **인공지능 모델 함수**작성을 위한 함수 생성 방법과 구조에 대해서 설명한다.



## Create Function

사용자가 함수를 만들기 위해서는 아래와 같은 과정을 거쳐야 한다.

1. DCF에서 제공하는 기본 함수 컴포넌트 템플릿 생성
2. 기본 함수 컴포넌트 내에서 사용자가 정의한 인공지능 모델 작성
3. 작성된 함수 빌드
4. 빌드된 함수 작동 테스트
5. DCF에 함수 배포



사용자가 인공지능 모델 함수 컴포넌트를 작성하려면 먼저 DCF에서 제공하는 함수 템플릿을 받아와야 한다. 함수 템플릿은 CLI를 통해서만 생성할 수 있다. 



아래 명령어는 함수 템플릿을 생성하는 CLI 명령어이다.

```bash
$ dcf-cli function init [function name] --runtime [runtime] -f [name of configuration yaml file. default name is config.yaml] --gateway [dcf gateway address]
>>> dcf-cli function init echo --runtime python
Directory: echo is created.
Function handler created in directory: echo/src
Rewrite the function handler code in echo/src directory
Config file written: config.yaml
```

`init`함수의 옵션과 순서는 아래와 같다.

1. `<Function Name>` : 함수 이름이다. 반드시 첫글자는 영문 소문자로 구성되어야 한다.
2. `--runtime; -r` : 함수의 런타임 종류 (Python 또는 Golang)
3. `--config; -f`: 함수 설정 파일의 이름. 인자 이 없다면 `config.yaml`파일로 자동 생성된다.
4. `--entrypoint; -e`: 호출할 함수 이름. 인자 값이 없다면 `handler`가 기본 값이다.
5. `--dir; -d`: 실질적으로 함수를 포함하는 `handler.py`의 위치 경로
6. `--gateway; -g`: DCF의 gateway 주소. 함수를 호출할 때 사용하는 DCF의 주소이다. 기본적으로는 IP 값이며 도메인 설정이 되어있다면 URL로도 사용이 가능하다.



## Function Template

함수를 생성하면 기본적으로 함수 이름과 같은 폴더가 생성된다. 폴더 내부에는 아래와 같은 파일들이 생성된다.

```bash
.
├── config.yaml
├── Dockerfile
├── requirements.txt
└── src
    └── handler.py

```

- `config.yaml` : DCF에 대한 정보, 배포되는 함수 컨테이너에 대한 설정 값
- `Dockerfile` : 함수 컨테이너의 베이스 도커 이미지 파일
- `requirements.txt` : 함수에 필요한 의존성 패키지 파일 리스트
- `src/handler.py` : 작성해야 할 함수 파일



### config.yaml

`config.yaml`파일은 DCF에 대한 정보와 함수 컨테이너에 대한 설정 값을 포함하고 있다.

```
functions:
  echo: 									
    runtime: python 						
    desc: "" 								
    maintainer: ""							
    handler:								
      dir: ./src							
      file: ""								
      name: Handler							
    docker_registry: keti.asuscomm.com:5001 
    image: keti.asuscomm.com:5001/echo		
    limits:									
      memory: ""							
      cpu: ""								
      gpu: ""								
    build_args:								
    - CUDA_VERSION=9.0						
    - CUDNN_VERSION=7						
    - UBUNTU_VERSION=16.04					
dcf:										
  gateway: keti.asuscomm.com:32222			
```

`config.yaml`파일 구성은 위와 같으며 각 필드에 대한 상세 설명은 다음과 같다.

- `functions` : 함수 컨테이너에 대해서 기술한다.
  - `echo` : 함수 이름이다. `dcf-cli function init`함수에서 인자 값으로 준 함수의 이름이 된다.
    - `runtime` : 함수 런타임 종류. 현재 DCF는 Python 3.x와 Golang을 런타임으로 제공한다.
    - `desc` : 함수에 대한 상세 설명. 함수 담당자가 함수에 대한 설명을 기술할 수 있다.
    - `maintainer` : 함수 작성자 혹은 유지보수 담당자에 대한 정보를 기록한다. 이메일 혹은 담당자 이름, 닉네임 등 자유롭게 서술 가능하다.
    - `handler` : 함수의 엔트리포인트이다. `dcf-cli function init`시 `-e`옵션으로 변경할 수 있지만, 변경하지 않을 것을 권장한다.
      - `dir` : 함수파일의 디렉토리 위치를 의미한다. 이 또한 `dcf-cli function init`시  `-d`옵션으로 변경할 수 있지만, 변경하지 않을 것을 권장한다.
      - `name` : 함수 파일 내부에 작성되어 있는 함수 이름을 의미한다. Python 런타임에서는 `Handler`라는  Callable Class로 구성되어 있다.
    - `docker_registry`: DCF 사설 도커 레지스트리의 주소 값이다.
    - `image` : DCF 사설 도커 레지스트리에 보내질 도커 이미지의 이름 값이다.
    - `limits`: 함수 컨테이너의 컴퓨팅 리소스 값 설정에 대해서 기술한다.
      - `memory`: 함수 컨테이너가 가질 수 있는 메모리 값을 기술한다.
      - `cpu` : 함수 컨테이너가 가질 수 있는 CPU 코어 개수 값을 기술한다.
      - `gpu` : 함수 컨테이너가 사용할 GPU 개수를 적는다. 개수 값이 빈 문자열이면 CPU 함수로 작동한다.
    - `build_args`: 추가 옵션에 대해서 기술한다.
      - `CUDA_VERSION = 9.0` : GPU 사용 시 Nvidia-Docker의 CUDA 버전을 결정한다. 기본 값은 9.0이다.
      - `CUDNN_VERSION = 7`: GPU 사용 시 Nvidia-Docker의 CuDNN 버전을 결정한다. 기본 값은 7이다.
      - `UBUNTU_VERSION = 16.04`: Nvidia-Docker의 Ubuntu 버전을 결정한다. 기본 값은 16.04이다.
- `dcf` : DCF에 대한 정보를 기술한다.
  - `gateway` : 함수 호출 시 DCF Gateway를 통해서 함수를 호출하게 되는데, 그 DCF의 gateway 주소 값이다.



### requirements.txt

Python 런타임에서 필요한 의존성패키지 리스트를 적는다. 아래는 requirements.txt의 예시를 보여준다.

```
numpy==1.16.4
matplotlib
scipy==1.0.0
pillow
wavio
```



### Dockerfile

함수 컨테이너의 베이스 도커 이미지다. 함수를 빌드하게 되면 해당 Dockerfile을 기반으로 함수 컨테이너를 빌드하게 된다.



### src/handler.py

Python 런타임에서 실제 함수 내용이 작성되는 파일이다. 기본적으로 제공되는 함수 내용은 아래와 같다.

```python
class Handler:
    def __init__(self):
        pass

    def __call__(self, req):
        return req.input
```



Python 런타임의 경우 함수 모듈을 제공하지 않으며 Callable Class를 제공한다. Callable Class는 인공지능 함수를 배포할 때, 초기화 함수 `__init__`을 통해 인공지능 모델의 가중치를 불러올 수 있으며 `__call__`함수를 통해 일반 함수를 호출하듯이 사용할 수 있는 메커니즘을 제공한다.



따라서 인공지능 모델 개발자들은 기존에 학습과 추론이 통합되어 있던 코드에서 추론부만 분리하여 `Handler` Class 구조에 맞춰 추론 모델을 재작성해야 한다.
