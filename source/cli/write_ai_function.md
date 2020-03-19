# Write AI Function

기존의 마이크로 서비스 아키텍쳐(a.k.a Serverless)에서 흔히 사용하던 함수 개념과 DCF에서 사용하는 함수 개념의 차이를 설명한다. 또한 생성한 함수 템플릿에서 인공지능 모델을 어떻게 작성하는지에 대해서 설명한다.



## Difference DCF between others

OpenFaas나 AWS의 Lambda와 같은 마이크로 서비스 아키텍쳐 플랫폼들은 함수를 이용해 마이크로 서비스를 구성한다. 예를 들어, 사용자들은 마이크로 서비스 아키텍쳐 플랫폼에 함수를 배포하기 위해 Python으로 함수를 작성한다면 아래와 같은 형태를 띤다.

```python
def handler(parameters):
    # some logic
    return result
```



위의 코드로 작성된 함수는 마이크로 서비스 아키텍쳐에 함수 컨테이너로 생성되어 배포되게 된다. 배포된 이후 사용자에게 함수 요청이 있을 때마다 진입점(entrypoint)인 `handler`가 실행되어 값을 반환하는 구조를 갖게 된다.



이런 형태의 함수는 인공지능 모델을 서빙(Serving)할 때 큰 문제를 발생시키는데, 이는 인공지능 모델의 추론 과정이 어떻게 되는지를 확인하면 이해하기 쉽다.

1. 정적 혹은 동적 추론 그래프 생성 (Model 객체 선언)
2. 미리 학습된 가중치 파일 로드
3. 데이터를 입력하고 추론 결과 획득



우리가 인공지능 모델을 기존의 마이크로 서비스 아키텍쳐의 함수에 배포한다고 가정해보면 아래와 같은 코드로 나타낼 수 있다. 단순하게 함수 레벨의 함수 컨테이너를 제공하는 마이크로 서비스 아키텍쳐 서비스에서 인공지능 모델 배포를 하게 되면 매 요청마다 추론 그래프 생성과 미리 학습된 가중치 파일 로드가 불필요하게 반복하게 되는 것을 알 수 있다.

```python
import tensorflow as tf or torch or import mxnet as mx # etc..

def handler(req):
    model = AI_Model(parameters)
    model.load_weights(filepath)
    
    return model.predict(req.input)
```



반복 실행되는 추론 그래프 생성과 학습된 가중치 파일 로드는 함수의 요청 응답시간을 크게 지연시키며(전체 추론 시간의 60%) 정적 그래프를 사용하는 일부 프레임워크에서는 별도의 Reuse 옵션을 요구하거나 GPU 메모리 아웃이 발생하게 된다. 특히 긴 함수의 요청 응답시간은 시스템이 실시간으로 작동하지 않아도 되는 경우에는 큰 문제가 되지 않으나 영상처리 알고리즘에서 가끔씩 요구되는 실시간 처리가 필요한 경우는 긴 함수의 요청 응답시간은 치명적일 수 있다.



DCF에서는 기존 마이크로 서비스 아키텍쳐의 단점을 보완하기 위해서 아래와 같은 함수 구조를 갖는다. 이러한 클래스를 Callable Object라고 하며 상태 값을 갖는 함수를 사용하고자 할 때 사용한다. DCF에서 제공하는 함수를 사용하게 되면 추론 그래피 생성과 학습된 가중치 파일 로드 중복을 제거할 수 있다. 이 덕분에 DCF의 함수는 추론 시간과 GPU 메모리 에러를 발생시키지 않는다.

```python
class Handler:
    def __init__(self, parameters):
        self.model = AI_Model(model_parameters)
        self.model.load_weights(filepath)
        
    def __call__(self, req):
        return self.model.predict(req.input)
        
```



## Input / Output Data Interfaces

DCF의 입/출력 데이터 형태는 정해져 있지 않으며 사용자가 원하는 모든 형태의 인터페이스를 사용할 수 있다. 이는 사용자가 정의하기 나름이다. 예를 들어 대표적으로 HTTP API 혹은 RESTful API에서 사용하는 XML/JSON 형태, Python 객체를 직렬화한 Pickle, 직접 바이너리 파일 Bytes Stream으로 전송할 수도 있다. 한 가지 주의해야 할 점은 DCF 함수를 호출할 시, 입력 데이터 타입은 Bytes Array 여야 한다.



본 챕터에서는  JSON, Pickle, Base64를 사용한 입/출력 인터페이스 예제를 소개한다.



### Pickle

Pickle을 이용하게 되면 Python-Python 간 통신에서 Python의 내장 자료 타입을 모두 사용할 수 있다는 장점이 있다. 하지만 Python 객체를 직렬화하고 이를 인터페이스로 사용한다면 해당 어플리케이션은 Pickle에 의존되어 Pickle을 지원하는 환경에서만 함수를 사용할 수 있게 된다.



Pickle을 함수에서 사용하는 방법은 아래와 같다. 

**clinet.py**

```python
import io
import pickle
from dcfgrpc.api import dcf

dict_data = {"a" : "0", "b" : "1"}
f = io.BytesIO()
pickle.dump(dict_data, f)
message = f.getvalue()

result = dcf(url='DCF IP ADDRESS:PORT',service="FUNCTION NAME", arg=message)
print(result)
```



**handler.py**

```python
import pickle
import json

class Handler:
    def __init__(self):
        pass

    def __call__(self, req):
        pickle_data = req.input
        data = pickle.loads(pickle_data)
        
        return json.dumps(data)
```



### JSON

JSON은 HTTP API나 RESTful API에서 많이 사용하는 인터페이스이다. JSON을 사용하게 되면 다양한 key-value의 입력값을 사용할 수 있다.



**Client.py**

```python
import json
from dcfgrpc.api import dcf

data = {"a": "0", "b": "1" }
json_str = json.dumps(data)
result = dcf(url="DCF IP ADDRESS:PORT",service="FUNCTION NAME", arg=json_str.encode())
print(result)

```



**handler.py**

```python
import json

class Handler:
    def __init__(self):
        pass

    def __call__(self, req):
        input_data = req.input
        json_data =json.loads(input_data.decode())
        
        return json.dumps(json_data)
```





### Base64

Base64는 8비트 바이너리 데이터를 ASCII 영역의 문자들로 변환해주는 인코딩 방식이다. 주로 이미지 데이터를 HTTP API로 보내기 위한 인코딩 방식으로 많이 쓰인다. Base64인코딩과 JSON을 같이 활용하면 이미지 데이터도 JSON에 넣어 함께 전송할 수 있다.



**Client.py**

```python
import json
import base64
from dcfgrpc.api import dcf

with open("IMAGE FILE PATH", "rb") as image_file:
    encoded_string = base64.encodestring(image_file.read())

result = dcf(url='DCF IP ADDRESS:PORT',service="FUNCTION NAME", arg=encoded_string)
print(result)
```



**handler.py**

```python
import io
import json
import base64
from PIL import Image

class Handler:
    def __init__(self):
        pass

    def __call__(self, req):
        base64Str = req.input
        imageData = base64.decodestring(base64Str)
        img = Image.open(io.BytesIO(imageData))
        
        return "Image size : {}".format(img.size)
```



**requirements.txt**

```
Pillow
```





## Convert AI Model into DCF Function

사용자가 설계한 인공지능 모델이 학습과 추론이 서로 분리되어 있지 않은 상태에서 DCF에 모델을 배포하고 싶다면 기존의 인공지능 모델을 DCF에 올릴 수 있도록 모델 구조를 변경해 주어야 한다.



예를 들어, PyTorch를 이용하여 분류기를 학습하는 코드가 있다고 가정하자. 예제코드는 [*Training a Classifier*](https://pytorch.org/tutorials/beginner/blitz/cifar10_tutorial.html#sphx-glr-beginner-blitz-cifar10-tutorial-py)를 참고했다.

**Model.py**

```python
import torch.nn as nn
import torch.nn.functional as F


class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(3, 6, 5)
        self.pool = nn.MaxPool2d(2, 2)
        self.conv2 = nn.Conv2d(6, 16, 5)
        self.fc1 = nn.Linear(16 * 5 * 5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        x = self.pool(F.relu(self.conv1(x)))
        x = self.pool(F.relu(self.conv2(x)))
        x = x.view(-1, 16 * 5 * 5)
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x

```



**Train.py**

```python
import torch.optim as optim

transform = transforms.Compose(
    [transforms.ToTensor(),
     transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5))])

trainset = torchvision.datasets.CIFAR10(root='./data', train=True,
                                        download=True, transform=transform)
trainloader = torch.utils.data.DataLoader(trainset, batch_size=4,
                                          shuffle=True, num_workers=2)

testset = torchvision.datasets.CIFAR10(root='./data', train=False,
                                       download=True, transform=transform)
testloader = torch.utils.data.DataLoader(testset, batch_size=4,
                                         shuffle=False, num_workers=2)

classes = ('plane', 'car', 'bird', 'cat',
           'deer', 'dog', 'frog', 'horse', 'ship', 'truck')
net = Net()
criterion = nn.CrossEntropyLoss()
optimizer = optim.SGD(net.parameters(), lr=0.001, momentum=0.9)

PATH = './cifar_net.pth'

for epoch in range(2):  # loop over the dataset multiple times

    running_loss = 0.0
    for i, data in enumerate(trainloader, 0):
        # get the inputs; data is a list of [inputs, labels]
        inputs, labels = data

        # zero the parameter gradients
        optimizer.zero_grad()

        # forward + backward + optimize
        outputs = net(inputs)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()

        # print statistics
        running_loss += loss.item()
        if i % 2000 == 1999:    # print every 2000 mini-batches
            print('[%d, %5d] loss: %.3f' %
                  (epoch + 1, i + 1, running_loss / 2000))
            running_loss = 0.0
            # Model save
            torch.save(net.state_dict(), PATH)

print('Finished Training')
```



위의 학습 코드를 DCF의 함수로 사용하고 싶다면 아래와 같이 추론 모델로 추상화하여 사용할 수 있다. 본 예제 코드 방식 말고도 다양한 방식을 사용할 수 있다.(참고: 해당 코드는 실제로 돌아가는 코드가 아님)

**Predictor.py**

```python
from model import Net

class Predictor:
    def __init__(self, model_parameters):
        self.model = Net(model_parameters)
        self.model.eval()
        
    def load_weights(self, filepath):
        self.model.load_state_dict(torch.load(filepath))
        
    def predict(self, input_data):
        return self.post_processing(self.model(input_data))
    
    def post_processing(self, input_tensor):
        # some logic like as detection or segmentation
        return result
    
```



**handler.py**

```python
class Handler:
    def __init__(self, parameters):
        self.predictor = Predictor(model_parameters)
        self.predictor.load_weights(filepath)
        
    def __call__(self, input_data):
        return self.predictor.predict(input_data)
        
```

