---
title: "TCP에서 연결 종료시 4-way Handshake가 필요한 이유"
description: "왜 연결할때는 3way이고 종료할때는 4way일까"
date: 2022-02-11
update: 2022-02-11
tags:
  - network
  - tcp
  - 작성중
series: "오늘의 개발일기"
---

- 왜 연결시에는 3-way 이고 연결 종료시에는 4-way인지 궁금해짐


## 들어가며

TCP통신을 위해 3-way Handshake가 필요하다는 것은 알게 되었습니다. 전이중 통신을 지향하기 때문에 양 측 모두 데이터를 송수신할 준비가 되었다는 것을 검증받기 위해 송수신 모두 문제가 없음을 검증하기 위해서 3-way 절차가 필요합니다.

근데, 4-way 는 왜 나온걸까요? 종료시에도 3-way로 하면 똑같이 검증할 수 있는 것이 아닐까요?
단순히 종료할떄는 4-way이다 라고 알고 넘어갔는데 계속 궁금증이 남아 찾아보았습니다.

TCP의 Handshake에 대해서는 너무나 많은 글이 있습니다. 연결 과정에 대해서는 자세히 적지 않으려 합니다.

## 4-way Handshake가 일어나는 과정



## 왜 4-way가 필요한가?



## 참고자료
- [TCP Connection Termination - GeeksforGeeks](https://www.geeksforgeeks.org/tcp-connection-termination/)
- [Network 3-way Handshake & 4-way Handshake](https://it-mesung.tistory.com/166)