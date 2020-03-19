# Introduction

Digital Companion Framework(이하 DCF)의 CLI는 아래와 같은 기능에 한하여 DCF와 상호작용을 할 수 있다.



1. 함수 런타임 목록 조회
2. 함수 생성
3. 함수 빌드
4. 함수 테스트
5. 함수 배포
6. 함수 삭제
7. 함수 호출
8. 함수 로그 확인



DCF의 함수관리는 CLI를 통해서만 이루어지게 되며, DCF에 배포된 함수는 CLI 혹은 개발자의 클라이언트 프로그램을 이용해서 호출할 수 있다. 전체 DCF의 함수 라이프 사이클은 아래와 같다.

![DCF 함수 라이프사이클](https://user-images.githubusercontent.com/13328380/71800226-d97a7700-309a-11ea-9c56-c95230ebab4f.png)