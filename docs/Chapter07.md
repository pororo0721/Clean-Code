- [7장 오류 처리](#오류-처리)
    
    [오류 코드 보다 예외를 사용하라](#오류-코드-보다-예외를-사용하라)

    [try-catch-finally 문부터 작성하라](#try-catch-finally-문부터-작성하라)

    [미확인 예외를 사용하라](#미확인unchecked-예외를-사용하라)

    [예외에 의미를 제공하라](#예외에-의미를-제공하라)

    [호출자를 고려해 예외 클래스를 정의하라](#호출자를-고려해-예외-클래스를-정의하라)

    [null을 반환하지 마라](#null을-반환하지-마라)

- [7장을 돌이켜보며](#7장을-돌이켜보며)

***

## 오류 처리

오류 처리는 프로그램에 존재하는 필수 요소 중 하나이다.

그럼에도, 여기저기 흩어진 오류 처리 코드는 실제 코드가 하는 일을 파악하기 불가능하게 만든다.

그렇기 때문에 오류 처리를 작성하는 것도 중요할 수 밖에 없다.

### 오류 코드 보다 예외를 사용하라

예외를 지원하지 않는 프로그래밍 언어가 많았던 때도 있었다.   
예외를 지원하지 않는 언어는 오류를 처리하고 보고하는 방법이 제한적이었다.  오류 플래그를 설정하거나, 호출자에게 오류 코드를 반환하거나 등등.

<pre>
<code>
public void sendShutDown(){
    DeviceHandle handel = getHandle(DEV1);
    // 디바이스 상태를 점검한다.
    if(handel != DeviceHandle.INVALID) {
        // 레코드 필드에 디바이스 상태를 저장한다.
        retrieveDeviceRecord(handle);
        // 디바이스가 일시정지 상태가 아니라면 종료한다.
        if(record.getStatus() != DEVICE_SUSPENDED) {
            pause.Device(handle);
            clearDeviceWorkQueue(handle);
            closeDevice(handle);
        }
        else {
            logger.log("Device suspended. Unable to shut down");
        }
    } else {
        logger.log("Invalid handle for: " + DEV1.toString());
    }
    ...
}
</code>
</pre>

위와 같은 방법을 사용하면, 함수를 호출한 즉시 오류를 확인해야 하기 때문에 호출자 코드가 복잡하다.

이렇기에 오류가 발생하면 예외를 던지는 편이 낫다.    아래 예제는 오류를 발견하면 예외를 던지는 경우이다.

<pre>
<code>
public void sendShutDown(){
    try{
        tryToShutDown(); 
    } catch (DeviceShutDownError e){
        logger.log(e);
    }      
}
</code>
</pre>

이렇게 하면, 예외를 처리하는 함수와 예외가 발생할 수 있는 함수를 명확히 구분할 수 있게 된다.

또한, 각 개념을 독립적으로 살펴보고 이해할 수 있다.


***

### Try-Catch-Finally 문부터 작성하라 

예외가 발생할 코드를 짤 때는 try-catch-finally 문으로 시작하는 편이 낫다.

try 블럭에서 무슨 일이 생기든지, 호출자가 기대하는 상태를 정의하기 쉬워지기 때문이다.

또한, try-catch 구조로 범위를 정의한 후, TDD를 사용해 필요한 나머지 논리를 추가할 수도 있다. (TDD란? Test Driven Development)

그러면 자연스럽게 try 블럭의 트랜잭션 범위부터 구현하여, 범위 내에서 트랜잭션의 본질을 유지하기 쉬워진다.

***

### 미확인(Unchecked) 예외를 사용하라 

이것과 관련된 논쟁은, 여러 해 동안 자바 프로그래머들 사이에서 확인된(Checked) 예외의 장단을 놓고 논쟁을 벌여왔었다.

처음 **확인된 예외**을 선보였을 때, 멋지다고 생각했지만 이 개념이 없는 언어도 많다. C++, C#, 파이썬, 루비 등.

그럼에도 불구하고 해당 언어들은 안정적인 소프트웨어를 구현하기에 무리가 없다.

오히려 **확인된 예외**은 OCP(Open Closed Principle)을 위반한다. 

상위 함수에서 하위 함수로 이뤄진 구조에서 하위 함수에서 추가로 예외가 발생할 수 있는 코드가 추가되면 예외 핸들러를 찾을 때까지 모든 함수가 예외를 던져야 한다.

즉 **확인된 예외** 때문에 하위 단계에서 코드를 변경하면, 상위 단계 메서드 선언부를 고쳐야 한다.

물론 때로는 **확인된 예외**가 유용하다.   아주 중요한 라이브러리의 경우는 모든 예외를 잡아야 한다.

그렇지만 일반적인 애플리케이션의 경우는, 의존성이라는 비용이 이익보다 크기에, 우리는 **미확인 예외**를 사용하도록 하자.

***

### 예외에 의미를 제공하라

예외를 던질 땐 전후 상황에 대한 정보를 충분히 덧붙여서 전달하자. 

그러면 오류를 발생한 원인과 위치를 찾기 쉬워진다. 

***

### 호출자를 고려해 예외 클래스를 정의하라

오류를 분류하는 방법은 수없이 많다.   오류가 발생한 우치로 분류도 가능하다. 발생한 컴포넌트나 유형으로도 가능하다.

그렇지만, 오류를 형편없이 분류하는 것은 안된다.

중복이 심한 경우 또한 그것에 속한다.

대다수 상황에서 우리는 오류를 처리할 때

1. 오류를 기록한다
2. 프로그램을 계속 수행해도 좋은지 확인한다

라는 프로세스로 처리한다.    이 경우는 예외 유형과 무관하게 대응 방식이 거의 동일하여, 코드를 간결하게 고치기 아주 쉽다.

***

### null을 반환하지 마라

우리가 흔히 저지르는 바람에 오류를 유발할 수 있다. 

그 중 첫번째가 null을 반환하는 습관이다.

이로 인해 null을 체크하는 코드로 애플리케이션이 가득한 문제가 발생한다. 

<pre>
<code>
public void registerItem(Item item){
        if(item != null){
            ItemRegistry registry = persistenceStore.getItemRegistry();
            if(registry != null) {
                Item existing = registry.getItem(item.getID());
                ...
            }
        }
}
</code>
</pre>

위 예제를 보라, getItemRegistry()가 null을 반환함으로서, null 체크가 가득해지고 있다.

또한, 우리는 위 예제에서 persistenceStore에 대한 null 체크가 사라진 것을 모르고 넘어갈 수도 있다. 

그렇기에 null을 반환하고픈 유혹이 든다면, 차라리 예외를 던지거나 특수 사례 객체를 반환하라.

#### null을 전달하지 마라

메소드에 null을 반환하는 방식은 나쁘지만, 메소드로 null을 전달하는 것은 더 나쁘다.

정상적인 인수로 null을 기대하는 API가 아니라면, null을 전달하는 코드는 피하도록 하자.

***

### 7장을 돌이켜보며

깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다. 라는 것이 결론에 나오는 장입니다.

처음 개발을 접할 때, 예외 처리에 대해 미숙했던 부분들이 떠오르는 장이었습니다.

막상 실행해보면, 여기 저기서 예외 처리, 오류 등이 발생하고, 디버깅을 통해서 하나씩 고쳐나가며 습득했던 경험들이, 장을 통해서 다음 프로젝트를 진행해볼 땐,   이번 장에서 얻은 정보와 그간 잡고 있던 나만의 규칙을 어느 정도 적용을 시켜보아야 겠다는 생각이 들었습니다.

오류 처리를 프로그램 논리와 분리하면, 독립적 추론이 가능해져 코드 유지 보수성이 높아진다. 를 늘 염두에 두어야 할 것 같습니다.

***

## 해시태그 ##
#코딩 #개발자 #노마드코더 #북클럽 #노개북

***

## 작성시기 ##
2022.05.05

***