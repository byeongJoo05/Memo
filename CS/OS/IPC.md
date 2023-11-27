# IPC

## IPC란?

Inter-Process Communication, 즉 Process간 통신을 의미한다. Process들 간 데이터와 정보를 교활 수 있게 해주는 것이다.

Process들은 OS 내부적으로 병렬적인 수행을 하기에 서로 독립적이면서 상호 협력적 관계에 있다. 이런 Process들 간의 협력을 통해 아래와 같은 이득을 취할 수 있다.

- 정보 공유 (Information Sharing): 여러 응용 프로그램들이 동일한 정보에 흥미를 가질 수 있기 때문에 병행적으로 접근이 가능한 것이 좋다.
- 계산 가속화 (Computation SpeedUp): 작업을 빨리 끝마치기 위해 작업을 서브 작업들로 분할하여 여러 process들에서 병렬적으로 수행하는 것이 좋다.
- 모듈성 (Modularity): 시스템의 기능들을 프로세스들로 나누어 모듈화하여 시스템을 구성할 수 있다.

IPC의 대표적인 두 가지 방법은 아래와 같다.

### Shared Memory

![285659098-c3ab8d22-9a09-4e88-b4eb-1db8afab9ed9](https://github.com/byeongJoo05/Memo/assets/84984586/0574727c-c726-44ea-98fe-d1ca3bebc4bc)

Shared Memory는 메모리에 서로 공유하고 있는 영역이 있어 그 부분에 자신이 원하는 데이터를 작성하는 방법이다. 공유하는 영역을 만드는 작업은 운영체제의 도움을 받아야 가능하다. Process들은 단순히 메모리에 데이터를 읽고 쓸 수 있으며, 어떤 업데이트가 일어나더라도 곧바로 모든 process들이 그 내용을 확인할 수 있다.

가장 중요한 장점은 **일단 공유 메모리 영역을 만들어 놓는다면, 그 뒤에는 Kernel의 어떤 도움도 필요로 하지 않는다는 것이다.** Kernel을 자주 이용한다는 것은 System call이 자주 호출된다는 것이며, 이 것은 overhead 부하가 심해진다는 것을 의미한다.

Shared memory 방식을 사용하기 위해 우선 process들은 자신의 주소 공간에 공유 공간으로 사용할 부분을 준비한다. 그리고 이 shared memory를 이용해 통신하고자 하는 다른 process들은 이 세그먼트를 자신의 주소 공간에 추가하여야 한다. 일반적으로 운영체제는 한 process가 다른 process의 memory에 접근하는 것을 금지하기에 Kernel의 도움을 받아서 이 제약 조건을 제거하는 과정이 필요하다. 이런 과정을 거친 이후 process들은 공유 영역에 메세지를 읽고 쓸 수 있다.

### Message Passing

Message Passing 방법은 데이터를 메세지처럼 전달하는 방법이다. Message Passing은 메세지를 보내는 send와 메세지를 받은 receive 두 가지 연산으로 실행된다.

- Direct Communication

![Direct-Messaging](https://github.com/byeongJoo05/Memo/assets/84984586/7a6a6f02-e014-4648-a350-e9b13943add6)

Direct Communication은 point-to-point로 직접 전달하는 방식이다. 

send 연산은 send(수신자 PID, message)로, receive 연산은 receive(송신자 PID, message)의 형태로 구성된다. 통신을 위해 상대의 ID만 알고 있다면 두 process 간 자동으로 연결이 구축된다. 연산 자체에 수신자 혹은 송신자의 ID를 적어줘야 하기 때문에 혹시 ID가 수정되면 코드 전체 수정을 해주어야되는 단점이 존재한다.

- Indirect Communication

Indirect Communication은 직접 메세지를 전달하지 않고 mailbox를 이용하는 형태이다.

![Indirect-messaging](https://github.com/byeongJoo05/Memo/assets/84984586/bddd3ca5-5618-4a96-b556-7cb2d5f1d787)

수신자는 mailbox에 원하는 메세지를 담고, 송신자는 mailbox에서 원하는 메세지를 가져가는 형식이다. 어떤 process라도 mailbox에 정보를 넣을 수 있고 빼갈 수 있다. 연산은 send(mailbox 이름, message), receive(mailbox 이름, message)의 형태를 가진다.

**Blocking과 Non-Blocking방식을 통한 메세지 교환**

1. Blocking Send: 송신하는 process는 메세지가 수신자에 의해 수신될 때까지 아무것도 하지 못하고 Blocking 된다.
2. Non-Blocking Send: 송신하는 process가 메세지를 보내고 그 수신 여부와 관계없이 다음 작업을 진행한다.
3. Blocking Receive: 수신하는 process는 메세지가 이용 가능할 때까지 아무것도 하지 못하고 Blocking 된다.
4. Non-Blocking Receive: 수신하는 process는 유효한 메세지인지 Null Checking을 계속한다.

## 참고자료

[[운영체제] Processes (3) - Interprocess Communication (IPC)](https://jooona.tistory.com/7)

[Inter process Communication - Message Passing System](https://notesformsc.org/message-passing-system/)