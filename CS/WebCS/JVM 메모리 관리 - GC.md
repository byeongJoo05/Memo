# JAVA 메모리 관리 - GC

## Java Virtual Machine (JVM)

자바는 OS의 메모리 영역에 직접적으로 접근하지 않고 JVM이라는 가상머신을 이용해서 간접적으로 접근한다. JVM은 C로 쓰여진 또 다른 프로그램인데, 오브젝트가 필요해지지 않는 시점에서 알아서 `free()`를 수행하여 메모리를 확보한다. 웹 애플리케이션을 만들 때 모든 것을 다 직접 개발하여 쓰기보다 검증된 라이브러리나 프레임워크를 이용하는 것이 더 안전하고 편리한 것처럼 메모리 관리라는 까다로운 부분을 자바 가상머신에 모두 맡겨버리는 것이다.

프로그램 실행 시 JVM 옵션을 주어서 OS에 요청한 사이즈 만큼의 메모리를 할당 받아 실행하게 된다. 할당 받은 이상의 메모리를 사용하게 되면 에러가 나면서 자동으로 프로그램이 종료된다. 그러므로 현재 프로세스에서 메모리 누수가 발생하더라도 현재 실행 중인 것만 죽고, 다른 것에는 영향을 주지 않는다.

이렇게 자바는 가상머신을 사용함으로써 OS 레벨에서의 `memory leak`은 불가능하게 된다는 장점이 있다.

자바가 메모리 누수현상을 방지하는 또 다른 방법이 가비지 컬렉션이다.

## Garbage Collection

가비지 컬렉션이라는 개념은 자바에서 처음 사용된 것은 아니다. LISP라는 언어에서 처음 도입된 개념이다. 하지만 자바가 가비지 컬렉션이란 개념을 더욱 대중화 시킨데 기여한 부분은 있다.

프로그래머는 힙을 사용할 수 있는 만큼 자유롭게 사용하고, 더이상 사용되지 않는 오브젝트들은 가비지 컬렉션을 담당하는 프로세스가 자동으로 메모리에서 제고하도록 하는 것이 가비지 컬렉션의 기본 개념이다.

자바는 가비지 컬렉션에 아주 단순한 규칙을 적용한다.

> Heap 영역의 오브젝트 중 stack에서 도달 불가능한(Unreachable) 오브젝트들은 가비지 컬렉션의 대상이 된다.
> 

## Garbage Collection 코드로 이해하기

```java
public class Main {
    public static void main(String[] args) {
        String url = "https://";
        url += "yaboong.github.io";
        System.out.println(url);
    }
}
```

위 코드에서

```java
String url = "https://";
```

구문이 실행된 뒤 스택과 힙은 아래와 같다.

<img width="656" alt="java-memory-management_heap-11" src="https://github.com/byeongJoo05/Memo/assets/84984586/ea09eae2-ab9b-4cd4-8db7-221983eb510f">

다음 구문인

```java
url += "yaboong.github.io";
```

문자열 더하기 연산이 수행되는 과정에서 기존에 있던 `"https://"` 스트링에 `"yaboong.github.io"` 를 덧붙이는 것이 아니라 문자열에 대한 더하기 연산이 수행된 결과가 새롭게 heap 영역에 할당된다. 그 결과는 아래와 같다.

<img width="659" alt="java-memory-management_heap-12" src="https://github.com/byeongJoo05/Memo/assets/84984586/d1bed3a4-46e4-40ee-8148-114401b7058a">

Stack에는 새로운 변수가 할당되지 않는다. 문자열 더하기 연산의 결과인 `"https://yaboong.github.io"` 가 새롭게 heap 영역에 생성되고, 기존에 `"https://"` 를 레퍼런스하고 있는 url 변수는 새롭게 생성된 문자열을 레퍼런스하게 된다.

> 기존의 `"https://"` 라는 문자열을 레퍼런스하고 있는 변수는 아무것도 없으므로 **Unreachable** 오브젝트가 된다.
> 

JVM의 Garbage Collector는 Unreachable Object를 우선적으로 메모리에서 제거하여 메모리 공간을 확보한다. Unreachable Object란 Stack에서 도달할 수 없는 Heap 영역의 객체를 말하는데, 지금 예제에서 `"Https://"` 문자열과 같은 경우가 되겠다. 아주 간단하게 이야기해서 이런 경우에 Garbage Collection이 일어나면 Unreachable 오브젝트들은 메모리에서 제거된다.

Garbage Collection 과정은 **`Mark and Sweep`** 이라고도 한다. JVM의 Garbage Collector가 스택의 모든 변수를 스캔하면서 각각 어떤 오브젝트를 레퍼런스하고 있는지 찾는 과정이 `Mark`다. Reachable 오브젝트가 레퍼런스하고 있는 오브젝트 또한 marking 한다. 첫번째 단계인 marking 작업을 위해 모든 스레드는 중단되는데 이를 `stop the world` 라고 부르기도 한다. (`System.gc()`를 생각없이 호출하면 안되는 이유 중 하나다.)

그리고 나서 mark 되어있지 않은 모든 오브젝트들을 힙에서 제거하는 과정이 `Sweep`이다.

Garbage Collection 이라고 하면 garbage들을 수집할 것 같지만 실제로는 garbage를 수집하여 제거하는 것이 아니라 garbage가 아닌 것을 따로 mark하고 그 외의 것은 모두 지우는 것이다. 만약 힙에 garbage만 가득하다면 제거 과정은 즉각적으로 이루어진다.

Garbage Collection이 일어난 후의 메모리 상태는 아래와 같다.

<img width="646" alt="java-memory-management_heap-13" src="https://github.com/byeongJoo05/Memo/assets/84984586/0c895052-7ae1-49ac-b560-9b40ad2616b5">

## 메모리 구성 - Metaspace & Heap

### Metaspace

자바 8에 적용된 변화로 람다, 스트림, 인터페이스의 default 지시자가 대표적이다. 하지만 메모리 관점에서 가장 큰 변화로 PermGen이 사라지고 Metaspace가 이를 대체하게 되었다는 것이다.

PermGen은 자바 7까지 클래스의 메타데이터를 저장하던 영역이었고 Heap의 일부였다.

주요 내용은 다음과 같다.

- Permanent Generation은 힙 메모리 영역 중에 하나로 자바 애플리케이션을 실행할 때 클래스의 메타데이터를 저장하는 영역이다.
- 아래와 같은 것들이 Java Heap이나 native heap 영역으로 이동했다.
    - Symbols → native heap
    - Interned String → Java Heap
    - Class statics → Java Heap
- `OutOfMemoryError: PermGen Space error`는 더 이상 볼 수 없고 JVM 옵션으로 사용했던 PermSize와 MaxPermSize는 더 이상 사용할 필요가 없다. 대신 MetaspaceSize 및 MaxMetaspaceSize가 새롭게 사용되게 되었다.

자바 8부터 클래스들은 모두 힙이 아닌 네이티브 메모리를 사용하는 Metaspace에 할당된다고 한다.

### Heap - Old & Young (Eden, Survivor)

![java-memory-management_gc-6](https://github.com/byeongJoo05/Memo/assets/84984586/899c36e4-cf1a-4a14-b47b-b13c2a55e306)

Heap은 `Young Generation, Old Generation`으로 크게 두 개의 영역으로 나누어지며, Young Generation은 또 다시 `Eden, Survivor Space 0,1` 로 세분화 되어진다. S0, S1으로 표시되는 영역이 Survivor Space 0, 1이다. 각 영역의 역할은 가비지 컬렉션 프로세스를 알면 알 수 있다.

### 가비지 컬렉션 프로세스

- 새로운 오브젝트는 Eden 영역에 할당된다. 두 개의 Survivor Space는 비워진 상태로 시작한다.
- Eden 영역이 가득차면, MinorGC가 발생한다.
- MinorGC가 발생하면, Reachable 오브젝트들은 S0 으로 옮겨진다. Unreachable 오브젝트들은 Eden 영역이 클리어될 때까지 함께 메모리에서 사라진다.
- 다음 MinorGC가 발생할 때, Eden 영역에는 3번과 같은 과정이 발생한다. Unreachable 오브젝트들은 지워지고, Reachable 오브젝트들은 Survivor Space로 이동한다. 기존에 S0에 있었던 Reachable 오브젝트들은 S1으로 옮겨지는데, 이때 age 값이 증가되어 옮겨진다. 살아남은 모든 오브젝트들이 S1으로 모두 옮겨지면 S0와 Eden은 클리어 된다. `Survivor Space에서 Survivor Space로의 이동은 이동할 때마다 age값이 증가한다.`
- 다음 MinorGC가 발생하면, 4번 과정이 반복되는데, S1이 가득차 있었으므로 S1에서 살아남은 오브젝트들은 S0로 옮겨지면서 Eden과 S1은 클리어된다. 이 때에도, age 값이 증가되어 옮겨진다. `Survivor Space에서 Survivor Space로의 이동은 이동할 때마다 age 값이 증가한다.`
- Young Generation에서 계속해서 살아남으며 age 값이 증가하는 오브젝트들은 age 값이 특정 값 이상이 되면 Old Generation 으로 옮겨지는데 이 단계를 Promotion 이라고 한다.
- MinorGC가 계속해서 반복되면 Promotion 작업도 꾸준히 발생하게 된다.
- Promotion 작업이 계속해서 반복되면서 Old Generation이 가득차게 되면 MajorGC가 발생하게 된다.

> 용어정리 <br>
MinorGC - Young Generation에서 발생하는 GC <br>
MajorGC - Old Generation (Tenured Space)에서 발생하는 GC <br>
FullGC - Heap 전체를 clear 하는 작업 (Young/Old 공간 모두)
> 

위 과정의 반복이 가비지 컬렉션이다.

## Garbage Collector 종류

### Serial GC

`-XX:+UseSerialGC` 옵션을 줘서 사용할 수 있는 Serial GC는 JAVA SE 5, 6에서 사용되는 디폴트 가비지 컬렉터이다.

- MinorGC, MajorGC 모두 순차적으로 시행된다.
- Mark-Compact collection method를 사용한다.

Mark-Compact collection method란 새로운 메모리 할당을 빠르게 하기 위해 기존의 메모리에 있던 오브젝트들을 힙의 시작위치로 옮겨 놓는 방법이다. 창고에서 필요없는 물건들을 버린 후, 창고에 물건을 차곡차곡 쌓기 위해 창고 안을 정리하는 것이라 생각할 수 있다. 아래 그림을 참고하자.

![java-memory-management_gc-7](https://github.com/byeongJoo05/Memo/assets/84984586/b8f4d25b-0162-4ce6-9553-d2ae357487b5)

### Parallel GC

`-XX:+UseParallelGC` 옵션으로 사용 가능한Parallel 가비지 컬렉터는 young generation 에 대한 가비지 컬렉션 수행 시 멀티스레드를 사용한다. 멀티스레딩을 할 수 있는 ParallelGC를 사용하도록 옵션을 주었더라도, 호스트 머신이 싱글 CPU라면 디폴트 가비지 컬렉터(Serial GC)가 사용된다. 하지만, 호스트의 CPU가 두 개 이상이라면 young generation의 가비지 컬렉션 시간을 줄일 수 있다.

가비지 컬렉터 스레드 개수는 디폴트로 CPU 개수만큼이 할당되는데 `XX:ParallelGCThread=<N>` 옵션으로 조절가능하다. 또한, `-XX:+UseParallelOldGC` 옵션을 사용한다면, old generation 의 가비지 컬렉션에서도 멀티스레딩을 활용할 수 있다.

### Concurrent Mark Sweep (CMS) Collector

Concurrent Low Pause Collector라고도 불리는 CMS 컬렉터는 `-XX:+UseConcMarkSweepGC` 옵션으로 사용할 수 있다.

대부분 가비지 컬렉션 작업을 애플리케이션 스레드와 동시에 수행함으로써 가비지 컬렉션으로 인한 stop-the-world 시간을 최소화하는 GC이다.

CMS 컬렉터는 young generation 에 대한 가비지 컬렉션시 Parallel GC와 같은 알고리즘을 사용하는데, `-XX:ParallelCMSThreads=<N>` 옵션으로 스레드 개수를 설정할 수 있다.

일반적으로 CMS 컬렉터는 살아있는 오브젝트들에 대한 compact 작업을 수행하지 않으므로 메모리의 파편화(Fragmentation)가 문제가 된다면 더 큰 힙사이즈를 할당해야 한다.

### G1 Garbage Collector

Garbage First 라는 의미의 G1 가비지 컬렉터는 Java 7 부터 사용가능하며, 장기적으로 CMS 컬렉터를 대체하기 위해 만들어졌다. `-XX:+UseG1GC` 옵션으로 사용가능하다. G1 가비지 컬렉터는 이전까지 이야기한 것들과는 다른 방식으로 동작한다.