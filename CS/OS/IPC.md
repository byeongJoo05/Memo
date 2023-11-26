# IPC

## IPC란?

Inter-Process Communication, 즉 Process간 통신을 의미한다. Process들 간 데이터와 정보를 교활 수 있게 해주는 것이다.

Process들은 OS 내부적으로 병렬적인 수행을 하기에 서로 독립적이면서 상호 협력적 관계에 있다. 이런 Process들 간의 협력을 통해 아래와 같은 이득을 취할 수 있다.

- 정보 공유 (Information Sharing): 여러 응용 프로그램들이 동일한 정보에 흥미를 가질 수 있기 때문에 병행적으로 접근이 가능한 것이 좋다.
- 계산 가속화 (Computation SpeedUp): 작업을 빨리 끝마치기 위해 작업을 서브 작업들로 분할하여 여러 process들에서 병렬적으로 수행하는 것이 좋다.
- 모듈성 (Modularity): 시스템의 기능들을 프로세스들로 나누어 모듈화하여 시스템을 구성할 수 있다.

IPC의 대표적인 두 가지 방법은 아래와 같다.

### Shared Memory

![ipc-through-shared-memory](https://github.com/byeongJoo05/Memo/assets/84984586/c3ab8d22-9a09-4e88-b4eb-1db8afab9ed9)


Shared Memory는 메모리에 서로 공유하고 있는 영역이 있어 그 부분에 자신이 원하는 데이터를 작성하는 방법이다. 공유하는 영역을 만드는 작업은 운영체제의 도움을 받아야 가능하다. Process들은 단순히 메모리에 데이터를 읽고 쓸 수 있으며, 어떤 업데이트가 일어나더라도 곧바로 모든 process들이 그 내용을 확인할 수 있다.

가장 중요한 장점은 **일단 공유 메모리 영역을 만들어 놓는다면, 그 뒤에는 Kernel의 어떤 도움도 필요로 하지 않는다는 것이다.** Kernel을 자주 이용한다는 것은 System call이 자주 호출된다는 것이며, 이 것은 overhead 부하가 심해진다는 것을 의미한다.

Shared memory 방식을 사용하기 위해 우선 process들은 자신의 주소 공간에 공유 공간으로 사용할 부분을 준비한다. 그리고 이 shared memory를 이용해 통신하고자 하는 다른 process들은 이 세그먼트를 자신의 주소 공간에 추가하여야 한다. 일반적으로 운영체제는 한 process가 다른 process의 memory에 접근하는 것을 금지하기에 Kernel의 도움을 받아서 이 제약 조건을 제거하는 과정이 필요하다.

## 참고자료

[[운영체제] Processes (3) - Interprocess Communication (IPC)](https://jooona.tistory.com/7)