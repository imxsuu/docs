# Inquire Runtime list

Digital Companion Framework의 함수는 런타임(프로그래밍 언어)으로 Python과 Golang을 지원한다. 

CLI의 `runtime list` 명령줄을 이용하여 Digital Companion Framework에서 지원하는 런타임 종류를 조회할 수 있다. 런타임 종류는 나중에 함수를 새로 만들게 될 때, 필요한 인자값으로 사용하게 된다.

```bash
$ dcf-cli runtime list
>>>
Supported Runtimes are:
- python
- go
```



> 현재 Digital Companion Framework는 Python 3.x 이상을 권장하며 Python 2.x의 종료에 따라 Python 2.x의 지원을 하지 않는다.  하지만 사용자의 필요에 따라 설정값을 변경하여 Python 2.x도 사용할 수 있다.

