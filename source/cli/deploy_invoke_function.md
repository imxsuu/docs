# Deploy & Invoke Function

함수 템플릿을 이용하여 인공지능 추론을 위한 함수를 작성했다면 작성한 함수를 실제 테스트를 하거나 DCF에 배포하는 과정을 진행할 수 있다.



## Build

DCF는 서버리스 환경에서 작동하는 인공지능 추론 플랫폼이기 때문에 먼저 작성한 함수를 도커 이미지로 빌드해야 한다. DCF의 CLI(Command-line Interface)는 작성한 인공지능 모델 함수를 도커 이미지로 빌드하는 기능을 지원한다.



DCF의 CLI를 활용하여 도커 이미지로 빌드하는 방법은 다음과 같다.

```
$ dcf-cli function build -f config.yaml <function-name> -v
>>>
Building function (<function-name>) ...
Sending build context to Docker daemon  312.3MB
Step 1/49 : ARG ADDITIONAL_PACKAGE
Step 2/49 : ARG REGISTRY
Step 3/49 : ARG PYTHON_VERSION
Step 4/49 : ARG GRPC_PYTHON_VERSION=1.4.0
Step 5/49 : ARG WATCHER_VERSION=0.1.0
Step 6/49 : ARG handler_file=handler.py
Step 7/49 : ARG handler_name=Handler
Step 8/49 : ARG handler_dir=/dcf/handler
Step 9/49 : ARG handler_file_path=${handler_dir}/src/${handler_file}
Step 10/49 : ARG CUDA_VERSION=9.0
Step 11/49 : ARG CUDNN_VERSION=7
Step 12/49 : ARG UBUNTU_VERSION=16.04
Step 13/49 : ARG CUDA_VERSION_BACKUP=${CUDA_VERSION}
Step 14/49 : FROM ${REGISTRY}/watcher:${WATCHER_VERSION}-python3 as watcher
...
```



빌드가 완료되었다면 도커 이미지가 생성되는데, 해당 도커 이미지는 도커 명령어를 통해서 확인할 수 있다.

```
$ docker images
>>>
REPOSITORY                                TAG                            IMAGE ID            CREATED             SIZE
keti.asuscomm.com:5001/tensorflow-gpu     latest                         28aa05b3ae25        3 months ago        4.59GB
keti.asuscomm.com:5001/function-test      latest                         ba5f1e8b441e        3 months ago        3.25GB
```

*빌드를 할 때에는 `-f` 옵션을 항상 추가해 주어야 한다.*



### Build Options

`build` 명령어는 여러가지 옵션이 있다. 이러한 옵션으로 빌드할 때 발생하는 에러, config.yaml 파일, gateway 주소 등을 출력 및 조정할 수 있다. 옵션은 다음과 같은 방법으로 확인할 수 있다.

```
$ dcf-cli function build --help
>>
Build DCF function Image via the supplied YAML config using the "-f" flag.
Usage:
  dcf function build -f <YAML_CONFIG_FILE> [flags]

Examples:

    dcf-cli function build -f config.yaml
    dcf-cli function build -f echotest.yaml
    dcf-cli function build -f config.yaml -v
    

Flags:
  -v, --buildverbose    Print function build log
  -f, --config string   Path to YAML config file describing function(s)
  -h, --help            help for build
      --nocache         Do not use cache when building runtime image

Global Flags:
  -g, --gateway string   Set gateway URL
```



- `-v`:  함수를 도커 이미지로 빌드할 때, 로그 메세지를 출력한다.
- `-f`:  함수 생성시 설정한 yaml 파일을 입력한다.
- `-h`:  함수에 대한 도움말을 확인한다.
- `--nocache`:  이전에 빌드한 캐시값을 사용하지 않는다.
- `-g`:  DCF의 gateway 주소를 직접 명시한다.



## Test

작성한 인공지능 모델 함수를 도커 이미지로 빌드하였으니 빌드된 도커 이미지를 활용하여 배포 및 테스트를 진행할 수 있다. 빌드된 도커 이미지를 바로 배포하여도 되지만, 작성한 인공지능 모델 함수가 도커 이미지에서 잘 작동하는지 확인 후에 배포를 진행하고 싶다면 로컬 환경에서 테스트하여 배포 전 작동 여부를 확인할 수 있다.



아래 명령어를 활용하여 테스트 할 수 있다.

`echo "Hello DCF"`는 인공지능 모델 함수로 전달될 입력 데이터이며, `dcf-cli function run <Function name>`이 실행되는 도커 이미지가 된다. 함수가 정상적으로 작동한다면 인공지능 모델 함수에서 의도한 출력값을 `Hanlder reply`에서 확인할 수 있다.



만약 함수에 문제가 있다면, 테스트를 하는 과정에서 에러 메세지가 발생하므로 디버깅 작업을 통하여 배포 시에는 작동이 가능한 모델만 업로드할 수 있다.

*테스트 과정에서 함수가 작동하지 않지만 에러를 확인할 수 없는 경우, 배포를 진행할 때 에러 메세지를 확인할 수 있다.*

```
$ echo "Hello DCF" | dcf-cli function run [Function name]
>>> echo "Hello DCF" | dcf-cli function run echo
Checking VGA Card...
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GM206GL [Quadro M2000] [10de:1430] (rev a1)
01:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:0fba] (rev a1)


Checking Driver Info...

==============NVSMI LOG==============

Timestamp                           : Mon Oct  7 14:17:07 2019
Driver Version                      : 390.116

Attached GPUs                       : 1
...
Running image (keti.asuscomm.com:5001/echo) in local
Starting [dcf-watcher] server ...
Call echo in user's local
Handler request: Hello

Handler reply: Hello
```



### Test Options

`run` 명령어에도 여러가지 옵션이 있다. 이러한 옵션으로 테스트 할 때  config.yaml 파일, gateway 주소 등을 조정할 수 있다. 옵션은 다음과 같은 방법으로 확인할 수 있다.

```
$ dcf-cli fn run --help
>>
Run DCF container that created when execute "build" command for debugging in local
Usage:
  dcf function run <FUNCTION_NAME> [flags]

Examples:

    echo "hi" | dcf-cli function run echo-service


Flags:
  -f, --config string   Path to YAML config file describing function(s)
  -h, --help            help for run

Global Flags:
  -g, --gateway string   Set gateway URL
```



- `-f`:  함수 생성시 설정한 yaml 파일을 입력한다.
- `-h`:  함수에 대한 도움말을 확인한다.
- `-g`:  DCF의 gateway 주소를 직접 명시한다.



## Deploy

이제 빌드한 도커 이미지를 DCF에 배포해보자. 배포는 아래와 같이 진행할 수 있다.

```
$ dcf-cli function deploy -f config.yaml -v
>>>
Is docker registry(registry: keti.asuscomm.com:5001) correct ? [y/n] y
Pushing: echo, Image: keti.asuscomm.com:5001/echo in Registry: keti.asuscomm.com:5001 ...
The push refers to repository [keti.asuscomm.com:5001/echo]
519b1665e7d6: Preparing 
913823b0a3b0: Preparing 
abcdb1a22c59: Preparing 
10b48c649b87: Preparing 
3d2effb69d5a: Layer already exists 
8e9de3569873: Waiting 
...
```



### Deploy Options

`deploy` 명령어에도 여러가지 옵션이 있다. 이러한 옵션으로 배포할 때 발생하는 에러 config.yaml 파일, gateway 주소 등을 조정할 수 있다. 옵션은 다음과 같은 방법으로 확인할 수 있다.

```
$ dcf-cli fn deploy --help
>>
Push DCF function Image & Deploy DCF function via the supplied YAML config using the "-f" flag. Also write docker private registry using the "--registry" flag for push docker image into registry.
Usage:
  dcf function deploy -f <YAML_CONIFIG_FILE> [flags]

Examples:

    dcf-cli function deploy -f helloworld.yml
    dcf-cli fn deploy -f config.yml --replace=false --update=true
    dcf-cli function deploy -f config.yml -v
    dcf-cli fn deploy -f config.yml --registry 127.0.0.1:5000
    dcf-cli function deploy -f config.yml -g 10.0.0.180:31113


Flags:
  -f, --config string     Path to YAML config file describing function(s)
  -v, --deployverbose     Print function build log
  -h, --help              help for deploy
      --registry string   Docker private registry url
      --replace           Remove and re-create existing function(s)
      --update            Perform rolling update on existing function(s)

Global Flags:
  -g, --gateway string   Set gateway URL
```



- `-f`:  함수 생성시 설정한 yaml 파일을 입력한다.
- `-v` :  함수를 배포 할 때, 로그 메세지를 출력한다.
- `-h`:  함수에 대한 도움말을 확인한다.
- `--registry string` : 사설 도커 레지스트리를 지정한다.
- `--replace` : 이미 존재하는 함수가 있다면 지우고 다시 생성한다.
- `--update` : 이미 존재하는 함수에 롤링 업데이트를 실행한다.
- `-g`:  DCF의 gateway 주소를 직접 명시한다.



## Funtion List

배포된 함수는 DCF의 CLI에서 지원하는 업로드된 모델 리스트 확인 기능을 통해서 확인할 수 있다. 이때 `Status`가 `Ready`로 되어야 해당 함수가 작동할 준비가 된 것이다.

```
$ dcf-cli function list
>>>
Function           Image                   Maintainer         Invocations    Replicas      Status        Description                             
echo               $(repo)/echo                               0             1             Ready
```



### List Options

`list` 명령어에도 gateway 주소를 설정할 수 있는 옵션이 있다. 옵션은 다음과 같은 방법으로 확인할 수 있다.

```
$ dcf-cli fn list --help
>>
Lists DCF function
Usage:
  dcf function list [flags]

Examples:
  dcf-cli function list
        dcf-cli fn list


Flags:
  -h, --help   help for list

Global Flags:
  -g, --gateway string   Set gateway URL
```



- `-h`:  함수에 대한 도움말을 확인한다.
- `-g`:  DCF의 gateway 주소를 직접 명시한다.





## Call Function

함수가 DCF에 배포되고 `Status`가 `Ready`가 되었다면 CLI에서 지원하는 두 가지 명령어를 통해 함수가 정상적으로 호출되는지 확인해볼 수 있다.

```
$ echo "Hello DCF" | dcf-cli function invoke [function name]
>>> echo "Hello DCF" | dcf-cli function invoke echo
Hello, DCF
```

```
$ echo "Hello DCF" | dcf-cli function call [function name]
>>> echo "Hello DCF" | dcf-cli function call echo
Hello, DCF
```



### Call Options

`invoke`나 `call` 명령어에도 gateway 주소를 설정할 수 있는 옵션이 있다. 옵션은 다음과 같은 방법으로 확인할 수 있다.

```
$ dcf-cli fn invoke --help
>>
Call DCF function and reads with STDIN for Handler(user defined function)'s input(bytes)
Usage:
  dcf function call <FUNCTION_NAME> [flags]

Aliases:
  call, invoke

Examples:
  dcf-cli function call echo-service
        cat "image.png" | dcf-cli function call inception-service
        echo "hello" | dcf-cli function call echo-service


Flags:
  -h, --help   help for call

Global Flags:
  -g, --gateway string   Set gateway URL
```



- `-h`:  함수에 대한 도움말을 확인한다.
- `-g`:  DCF의 gateway 주소를 직접 명시한다.



## Delete Function

함수를 배포한 이후, 함수가 의도한 데로 작동하지 않을 때에나 함수를 업데이트해야 할 때, 기존의 함수를 삭제해야 할 필요가 있을 수 있다. 



이러한 경우 DCF의 CLI에서 지원하는 세 가지 명령어를 통해 이미 DCF에 배포된 함수를 삭제할 수 있다.

```
$ dcf-cli function delete <function-name>
>>>
Deleted: <function name>
```

```
$ dcf-cli function remove <function-name>
>>>
Deleted: <function name>
```

```
$ dcf-cli function rm <function-name>
>>>
Deleted: <function name>
```

*함수 삭제는 시간이 걸릴 수 있으니 이를 `dcf-cli function list`명령어로 확인한 후 재배포를 진행해야 한다.*



### Delete Options

`delete`, `remove`, `rm`  명령어에도 config.yaml파일, gateway 주소를 설정할 수 있는 옵션이 있다. 옵션은 다음과 같은 방법으로 확인할 수 있다.

```
$ dcf-cli fn delete --help
>>
Delete DCF function via the supplied YAML config using
the "-f" flag (which may contain multiple function definitions)
Usage:
  dcf function delete -f <YAML_CONIFIG_FILE> [flags]

Aliases:
  delete, remove, rm

Examples:
  dcf-cli function delete -f config.yml
        dcf-cli function delete echo-service


Flags:
  -f, --config string   Path to YAML config file describing function(s)
  -h, --help            help for delete

Global Flags:
  -g, --gateway string   Set gateway URL
```



- `-f`:  함수 생성시 설정한 yaml 파일을 입력한다.
- `-h`:  함수에 대한 도움말을 확인한다.
- `-g`:  DCF의 gateway 주소를 직접 명시한다.



## Log of Function

함수의 `Status`가 `Ready`로 변하지 않았거나 배포까지는 정상적으로 확인이 되었는데 호출에 이상이 있을 때, 디버깅을 위해서 배포된 함수가 정상적으로 작동하지 않는 이유를 확인해야 한다. 이럴 때에는 CLI의 Log 기능을 활용하여 함수의 로그 메세지를 확인할 수 있다.

```
$ dcf-cli function log <function-name>
```



### Log Options

`log`명령어에도 gateway 주소를 설정할 수 있는 옵션이 있다. 옵션은 다음과 같은 방법으로 확인할 수 있다.

```
$ dcf-cli fn log --help
>>
Display DCF function logs
Usage:
  dcf function log <FUNCTION_NAME> [flags]

Examples:
  dcf-cli function log resizeImg


Flags:
  -h, --help   help for log

Global Flags:
  -g, --gateway string   Set gateway URL
```



- `-h`:  함수에 대한 도움말을 확인한다.
- `-g`:  DCF의 gateway 주소를 직접 명시한다.

