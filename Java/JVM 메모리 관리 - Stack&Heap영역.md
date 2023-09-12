# Java 메모리 관리 - 스택 & 힙 영역

## JAVA의 Stack과 Heap

![javamemory-stack-and-heap-dzone](https://github.com/byeongJoo05/Memo/assets/84984586/a13692ca-abb5-4ef3-b24f-f8a2d52d8da7)

### Stack

- Heap 영역에 생성된 Object 타입의 데이터의 참조값이 할당된다.
- 원시타입의 데이터가 값과 함께 할당된다.
- 지역변수들은 scope에 따른 visibility를 가진다.
- 각 Thread는 자신만의 stack을 가진다.

`Stack`에는 heap 영역에 생성된 Object 타입의 데이터들에 대한 참조를 위한 값들이 할당된다.

또한, 원시타입 - byte, short, int, long, double, float, boolean, char 타입의 데이터들이 할당된다. 이때 원시타입의 데이터들에 대해서는 참조값을 저장하는 것이 아니라 실제 값을 stack에 직접 저장하게 된다.

Stack 영역에 있는 변수들은 visibility를 가진다. 변수 scope에 대한 개념이다. 전역변수가 아닌 지역변수가 `foo()` 라는 함수 내에서 Stack에 할당된 경우, 해당 지역변수는 다른 함수에서 접근할 수 없다. 예를 들어, `foo()` 라는 함수에서 `bar()` 함수를 호출하고 `bar()` 함수의 종료되는 중괄호(`}`)가 실행된 경우 (== 함수가 종료된 경우), `bar()` 함수 내부에서 선언한 모든 지역변수들은 stack에서 pop 되어 사라진다.

Stack 메모리는 Thread 하나당 하나씩 할당된다. 즉, 스레드 하나가 새롭게 생성되는 순간 해당 스레드를 위한 stack도 함께 생성되며, 각 스레드에서 다른 스레드의 stack 영역에는 접근할 수 없다.

**Stack이 어떻게 활용되는지 코드를 보며 살피자.**

```java
public class Main {
    public static void main(String[] args) {
        int argument = 4;
        argument = someOperation(argument);
    }

    private static int someOperation(int param){
        int tmp = param * 3;
        int result = tmp / 2;
        return result;
    }
}
```

```java
int argument = 4;
```

위와 같이 변수 선언을 함으로써 스택에 `argument`라는 변수명으로 공간이 할당되고, argument 변수이 타입은 원시타입이므로 이 공간에는 실제 4라는 값이 할당된다.

<img width="207" alt="java-memory-management_stack-1" src="https://github.com/byeongJoo05/Memo/assets/84984586/62cf2178-1b03-41ed-a58f-c27859317272">

```java
argument = someOperation(argument);
```

위 코드에 의해 `someOperation()` 함수가 호출된다. 호출될 때 인자로 argument 변수를 넘겨주며 scope가 `someOperation()` 함수로 이동한다. scope가 바뀌면서 기존의 argument라는 값은 scope에서 벗어나므로 사용할 수 없다. 이때 인자로 넘겨받은 값은 파라미터인 param 에 복사되어 전달되는데, param 또한 원시타입이므로 stack에 할당된 공간에 값이 할당된다. 현재 스택의 상태는 아래와 같다.

<img width="212" alt="java-memory-management_stack-2" src="https://github.com/byeongJoo05/Memo/assets/84984586/9b7a5204-b5b6-4370-bd15-a1bcbaba2e8a">

```java
int tmp = param * 3;
int result = tmp / 2;
```

같은 방식으로 스택에 값이 할당되며 현재 스택의 상태는 아래와 같다.

<img width="205" alt="java-memory-management_stack-3" src="https://github.com/byeongJoo05/Memo/assets/84984586/4873ce21-2b84-4f8f-8e17-29244b40e57e">

다음으로, 닫는 괄호 `}`가 실행되어 `someOperation()` 함수호출이 종료되면 호출함수 scope 에서 사용되었던 모든 지역변수들은 stack에서 pop된다. 함수가 종료되어 지역변수들이 모두 pop되고, 함수를 호출했던 시점으로 돌아가면 스택의 상태는 아래와 같이 변한다.

<img width="187" alt="java-memory-management_stack-4" src="https://github.com/byeongJoo05/Memo/assets/84984586/72ab6463-b5c9-4865-a018-13d21c2a0954">

argument 변수는 4로 초기화 되었지만, 함수의 실행결과인 6이 기존 argument 변수에 재할당된다. 물론 함수호출에서 사용되었던 지역변수들이 모두 pop 되기 전에 재할당 작업이 일어날 것이다.

그리고 main() 함수도 종료되는 순간 stack에 있는 모든 데이터들은 pop 되면서 프로그램이 종료된다.

### Heap

- Heap 영역에는 주로 긴 생명주기를 가지는 데이터들이 저장된다. (대부분의 오브젝트는 크기가 크고, 서로 다른 코드블럭에서 공유되는 경우가 많다.)
- 애플리케이션의 모든 메모리 중 stack에 있는 데이터를 제외한 부분이라고 보면 된다.
- 모든 Object 타입(Integer, String, ArrayList, …)은 heap 영역에 생성된다.
- 몇 개의 스레드가 존재하든 상관없이 단 하나의 heap 영역만 존재한다.
- Heap 영역에 있는 오브젝트들을 가리키는 레퍼런스 변수가 stack에 올라가게 된다.

```java
public class Main {
    public static void main(String[] args) {
        int port = 4000;
        String host = "localhost";
    }
}
```

`int port = 4000;`에 의해서 기존처럼 stack에 4000 이라는 값이 port 라는 변수명으로 할당되어 스택의 상태는 아래와 같이 된다.

<img width="684" alt="java-memory-management_heap-1" src="https://github.com/byeongJoo05/Memo/assets/84984586/0e031342-15a1-4875-8626-639c3e274d9d">

String은 Object를 상속받아 구현되었으므로 String은 heap 영역에 할당되고 stack에 host 라는 이름으로 생성된 변수는 heap에 있는 “localhost” 라는 스트링을 레퍼런스 하게 된다. 그림으로 표현하면 아래와 같다.

<img width="675" alt="java-memory-management_heap-2" src="https://github.com/byeongJoo05/Memo/assets/84984586/bfded6b1-89f3-4db6-810c-5e48c2818b97">

기본적인 stack과 heap 영역에 대한 이해는 끝났으므로, 조금 더 복잡한 예제코드와 함께 각 영역의 메모리 할당과 해제가 어떻게 일어나는지 살펴보자.

```java
import java.util.ArrayList;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> listArgument = new ArrayList<>();
        listArgument.add("yaboong");
        listArgument.add("github");

        print(listArgument);
    }

    private static void print(List<String> listParam) {
        String value = listParam.get(0);
        listParam.add("io");
        System.out.println(value);
    }
}
```

프로그램의 시작과 함께 실행되는 첫 구문은 아래와 같다.

```java
List<String> listArgument = new ArrayList<>();
```

여기서 `new` 키워드는 특별한 역할을 한다. 생성하려는 오브젝트를 저장할 수 있는 충분한 공간이 heap 에 있는지 먼저 찾은 다음, 빈 List를 참조하는 listArgument 라는 로컬변수를 스택에 할당한다. 결과는 아래와 같다.

<img width="698" alt="java-memory-management_heap-3" src="https://github.com/byeongJoo05/Memo/assets/84984586/4741a402-7107-49a4-aa53-ae3fc0eea0b7">

```java
listArgument.add("yaboong");
```

위 구문은 `listArgument.add(new String("yaboong"));` 과 같은 역할을 한다.

즉, new 키워드에 의해 heap 영역에 충분한 공간이 있는지 확인한 후 “yaboong” 이라는 문자열을 할당하게 된다. 이때 새롭게 생성된 문자열인 “yaboong”을 위한 변수는 stack에 할당되지 않는다. List 내부의 인덱스에 의해 하나씩 `add()`된 데이터에 대한 레퍼런스 값을 갖게 된다. 그림으로 표현하면 아래와 같다.

<img width="683" alt="java-memory-management_heap-4" src="https://github.com/byeongJoo05/Memo/assets/84984586/c23b1269-2857-484f-bf2e-bdfd54ae1cf4">

```java
listArgument.add("github");
```

위 구문이 실행되면 List에서 레퍼런스하는 문자열이 하나 더 추가된다. 그림으로 표현하면 아래와 같다.

<img width="683" alt="java-memory-management_heap-5" src="https://github.com/byeongJoo05/Memo/assets/84984586/5605e251-940a-4a8b-a262-4beb68d74e33">

```java
print(listArgument);
```

구문에 의해 함수호출이 일어난다. 이때, listArgument 라는 참조변수를 넘겨주게 된다. 함수호출시 원시타입의 경우와 같이 넘겨주는 인자가 가지고 있는 값이 그대로 파라미터에 복사된다.

`print(List<String> listParam)` 메서드에서는 listParam 이라는 참조변수로 인자를 받게 되어있다. 따라서 print() 함수호출에 따른 메모리의 변화는 아래와 같다.

<img width="694" alt="java-memory-management_heap-6" src="https://github.com/byeongJoo05/Memo/assets/84984586/0800d48a-ee9d-40ef-8351-605a0879cb0e">

listParam 이라는 참조변수가 새롭게 stack에 할당되어 기존 List를 참조하게 되는데, 기존에 인자인 listArgument 가지고 있던 값(List에 대한 레퍼런스)을 그대로 listParam이 가지게 되는 것이다. 그리고 print() 함수 내부에서 listArgument는 scope 밖에 있게 되므로 **접근할 수 없는 영역**이 된다.

print() 함수 내부에서는 List에 있는 데이터에 접근하여 값을 value라는 변수에 저장한다. 이 때 print() 함수의 scope에서 stack에 value가 추가되고, 이 value는 listParam을 통해 List의 0번째 요소에 접근하여 그 참조값을 가지게 된다. 그리고나서 또 데이터를 추가하고, 출력함으로 print() 함수의 역할은 마무리된다.

```java
String value = listParam.get(0);
listParam.add("io");
System.out.println(value);
```

위 코드가 실행되고, 함수가 종료되기 직전의 stack과 heap은 아래와 같다.

<img width="678" alt="java-memory-management_heap-7" src="https://github.com/byeongJoo05/Memo/assets/84984586/092fe143-145d-4b21-a8cb-87141c03490d">

이제 함수가 닫는 중괄호 `}`에 도달하여 종료되면 print() 함수의 지역변수는 모두 stack에서 pop 되어 사라진다. 이때, List는 Object 타입이므로 지역변수가 모두 stack에서 pop되더라도 heap 영역에 그대로 존재한다. 즉, 함수호출시 레퍼런스 값을 복사하여 가지고 있던 listParam과 함수내부의 지역변수인 value만 스택에서 사라지고 나머지는 모두 그대로인 상태로 함수호출이 종료된다.

```java
print(listArgument);
```

위 함수호출이 종료된 시점에서 스택과 힙 영역은 아래와 같다.

<img width="703" alt="java-memory-management_heap-8" src="https://github.com/byeongJoo05/Memo/assets/84984586/28cccd0d-8b13-4c41-86e0-8ec6e3ad9f2b">

Object 타입의 데이터, 즉 heap 영역에 있는 데이터는 **함수 내부에서 파라미터로 copied value를 받아서 변경하더라도 함수호출이 종료된 시점에 변경내역이 반영되는 것**을 볼 수 있다.

### 아래 코드를 통해 지금까지 배운 stack과 heap 영역을 생각하며 분석해보자.

```java
public class Main {
    public static void main(String[] args) {
        Integer a = 10;
        System.out.println("Before: " + a);
        changeInteger(a);
        System.out.println("After: " + a);
    }

    public static void changeInteger(Integer param) {
        param += 10;
    }
}
```

- 실행 순서를 생각해보자.
    - Integer는 Object 타입이므로, 첫 구문인 `Integer a = 10;`에서 10은 heap영역에 할당되고, 10을 가리키는 레퍼런스변수 a가 스택에 할당된다.
    - 함수에 인자를 넘겨줄 때 파라미터는 copied value를 넘겨받는다.
    - 그러므로 `changeInteger(a);`에 의해, param이라는 레퍼런스 변수가 스택에 할당되고, 이 param은 main() 함수에서 a를 가리키던 곳을 똑같이 가리키고 있다.
    - main() 함수에서 레퍼런스하던 a와 같은 곳을 param이 가리키고 있으므로 param에 10을 더하면, changeInteger() 함수가 종료되고 a의 값을 출력했을 때 바뀐 값이 출력될 것이다.

**허나 값은 20이 나오지 않고 10이 나올 것이다.**

값이 바뀌지 않는 이유는 아래 코드를 생각해보면 될 것이다. String은 불변객체(immutable)로써 어떤 연산을 수행할 때마다 기존 오브젝트를 변경하는 것이 아니라 새로운 오브젝트를 생성하는 것이라고 알고 있을 것이다.

```java
public class Main {
    public static void main(String[] args) {
        String s = "hello";
        changeString(s);
        System.out.println(s);
    }
    public static void changeString(String param) {
        param += " world";
    }
}
```

- `changeString()` 내부동작을 살펴보자.
    - main() 메서드의 s변수가 레퍼런스하는 “hello” 오브젝트를 param에 복사하면서 changeString() 메서드가 시작된다.
    - `param += "world";` 를 실행하는 것은 heap에 “hello world”라는 스트링 오브젝트가 새롭게 할당되는 작업이다.
    - 기존 “hello” 오브젝트를 레퍼런스하고 있던 param으로 새롭게 생성된 스트링 오브젝트인 “hello world”를 레퍼런스 하도록 만드는 것이다.
    - changeString() 함수가 종료되면, 새롭게 생성된 “hello world” 오브젝트를 레퍼런스하는 param이라는 변수는 스택에서 pop되므로 어느 것도 레퍼런스하지 않는 상태가 된다.
    - 이런 경우 “hello world” 오브젝트는 **garbage**로 분류된다.

그러므로, changeString() 메서드를 수행하고 돌아가도 기존에 “hello”를 레퍼런스하고 있던 s 변수의 값은 그대로이다. `Immutable Object`는 불변객체로써, 값이 변하지 않는다. 변경하는 연산이 수행되면 변경하는 것처럼 보이더라도 실제 메모리에는 새로운 객체가 할당되는 것이다.

자바에서는 Wrapper Class에 해당하는 Integer, Character, Byte, Boolean, Long, Float, Short 클래스는 모두 Immutable이다. 그래서 heap에 있는 같은 오브젝트를 레퍼런스하고 있는 경우라도, 새로운 연산이 적용되는 순간 새로운 오브젝트가 heap에 새롭게 할당된다.

Wrapper 클래스의 구현을 보면 final이라는 키워드가 붙어있다. 클래스에 붙어있는 final은 값을 못하도록 하는 역할이 아닌, **상속을 제한하는 목적으로 붙이는 제어자**이다.

Integer 클래스를 까보면 내부에서 사용하는 실제 값인 value라는 변수가 있는데, 이 변수는 `private final int value;`로 선언 되어있다. 생성자에 의해 생성되는 순간에만 초기화되고 변경불가능한 값이 된다. 이것 때문에 Wrapper Class들도 String처럼 Immutable한 오브젝트가 되는 것이다.

### Garbage Collection 맛보기

```java
public class Main {
    public static void main(String[] args) {
        String url = "https://";
        url += "yaboong.github.io";
        System.out.println(url);
    }
}
```

```java
String url = "https://";
```

구문이 실행된 뒤 스택과 힙은 아래와 같다.

<img width="656" alt="java-memory-management_heap-11" src="https://github.com/byeongJoo05/Memo/assets/84984586/9ce742ee-f135-431f-8fde-8cef6c6a1190">

```java
url += "yaboong.github.io";
```

문자열 더하기 연산이 수행되는 과정에서, (String은 불변객체이므로) 기존에 있던 `https://` 스트링에 `"yaboong.github.io"` 를 덧붙이는 것이 아니라 문자열에 대한 더하기 연산이 수행된 결과가 새롭게 heap 영역에 할당된다. 그 결과를 그림으로 표현하면 아래와 같다.

<img width="659" alt="java-memory-management_heap-12" src="https://github.com/byeongJoo05/Memo/assets/84984586/5acdf2af-4629-4e07-91b5-7de0aefbd578">

Stack에는 새로운 변수가 할당되지 않는다. 문자열 더하기 연산의 결과인 `"https://yaboong.github.io"` 가 새롭게 heap 영역에 생성되고, 기존에 `"https://"` 를 레퍼런스 하고 있던 url 변수는 새롭게 생성된 문자열을 레퍼런스하게 된다.

> 기존의 `"https://"` 라는 문자열을 레퍼런스하고 있는 변수는 아무것도 없으므로 `Unreachable` 오브젝트가 된다.
> 

JVM의 Garbage Collector는 Unreachable Object를 우선적으로 메모리에서 제거하여 메모리 공간을 확보한다. Unreachable Object란 Stack에서 도달할 수 없는 Heap 영역의 객체를 말하는데, 지금의 예제에서 `"https://"` 문자열과 같은 경우가 되겠다. 아주 간단하게 이야기해서 이런 경우에 Garbage Collection이 일어나면 Unreachable 오브젝트들은 메모리에서 제거된다.

Garbage Collection 과정은 **Mark and Sweep**이라고 한다. JVM의 Garbage Collector가 스택의 모드 변수를 스캔하면서 각각 어떤 오브젝트를 레퍼런스 하고 있는 찾는 과정이 Mark다. Reachable 오브젝트가 레퍼런스하고 있는 오브젝트 또한 marking 한다. 첫번째 단계인 marking 작업을 위해 모든 스레드는 중단되는데 이를 stop the world라고 부르기도 한다. (System.gc()를 생각없이 호출하면 안되는 이유이기도 하다.)

그리고나서 mark 되어있지 않은 모든 오브젝트들을 힙에서 제거하는 과정이 Sweep이다.

Garbage Collection 이라고 하면 garbage들을 수집할 것 같지만 실제로는 garbage를 수집하여 제거하는 것이 아니라, garbage가 아닌 것을 따로 mark하고 그 외의 것은 모두 지우는 것이다. 만약 힙에 garbage만 가득하다면 제거 과정은 즉각적으로 이루어진다.

Garbage Collection이 일어난 후의 메모리 상태는 아래와 같을 것이다.

<img width="646" alt="java-memory-management_heap-13" src="https://github.com/byeongJoo05/Memo/assets/84984586/78114c23-96e9-4f8d-8cd6-1c530eaa3215">

### 참고자료

[자바 메모리 관리 - 스택 & 힙](https://yaboong.github.io/java/2018/05/26/java-memory-management/)