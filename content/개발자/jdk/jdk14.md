---
title: "jdk 14"
---

	https://openjdk.java.net/projects/jdk/14/

## Features

## 305:	Pattern Matching for instanceof (Preview)
instanceof에 Pattern matching을 지원하는 기능이 이야기 되고 있는 것으로 보인다.

#### Summary
Pattern matching allows common logic in a program

#### 기존 instanceof 사용의 예.
```
if (obj instanceof String) {
    String s = (String) obj;
    // use s
}
```

#### Motivation
기존의 처리는 1) type 확인, 2) type casting, 3) 새로운 local variable를 사용하고있다.
이런 사용은 코드를 장황하게 만든다. 그리고 3가지의 요인으로 프로그램에 에러가 발생할 수있는 상황을 만들 수있다.
그래서 java가 pattern matching을 수용해야 한다고 생각하였다.

#### Description

```
if (obj instanceof String s) {
    // can use s here
} else {
    // can't use s here
}
```
위 조건문 내에서 만약 obj instanceof String이 true이면 obj를 String으로 casting 후 binding variable s에 값을 할당한다. 그리고 true block 안에서 binding variable s는 유요한 상태이다.
만약 if의 결과가 false인 경우 false block 내에서는 s를 사용할 수없다.

```
if (!(obj instanceof String s)) {
    .. s.contains(..) ..	// field in the enclosing class
} else {
    .. s.contains(..) ..	// binding variable
}
```
true block의 s는 binding variable가 아니고 class field이다.
false block의 s는 binding variable이다.
즉 if 조건이 만족하는 block 에서 binding variable를 사용할 수 있는 것으로 보인다.

```
if (obj instanceof String s && s.length() > 5) {.. s.contains(..) ..}
```
&&의 오른쪽조건은 instanceof가 무조건 true인 경우에만 동작을 한다. 그래서 block내의 s는 binding variable이다.

```
if (obj instanceof String s || s.length() > 5) {.. s.contains(..) ..}
```
||의 경우 s는 binding variable가 아닌 class field이다. (테스트 필요)



## 343:	Packaging Tool (Incubator)
## 345:	NUMA-Aware Memory Allocation for G1
## 349:	JFR Event Streaming
## 352:	Non-Volatile Mapped Byte Buffers
## 358:	Helpful NullPointerExceptions
## 359:	Records (Preview)
## 361:	Switch Expressions (Standard)
Switch Expressions을 본격적으로 사용이 되는것 같다. switch에 대한 가독성 높고 명확한 code가 가능해진것 같다.

#### Summary
extend switch는 기존의 "case ... :" labels과 새로운 "case ... ->" labels 둘다 사용할 수 있고 switch expression에서 값을 생성 할 수 있는 새로운 statement(yield)가 추가되었다.

이 변경은 일상적인 코딩을 단순화 할 것이고, switch에서 pattern matching 사용을 위한 방법을 준비한다.

JDK 12 and JDK 13 preview language feature 였다.


#### History
Switch expressions는 2017년 12월 JEP 325에 의해 제안되었다. JEP 325는 preview feature로 2018년 8월 JDK 12를 대상으로했다.

JEP 325의 한가지는 break statement의 result value return을 위해 overloading하는 것이었다.

JEP 325의 또 하나는 switch expression에서 결과 값 return을 위해 break statement의 overloading 하는 것이다.
(https://openjdk.java.net/jeps/325)

JDK 12에 대한 피드백으로 이 같은 break 사용은 혼란스럽다 제안되었다. 피드백에 대한 응답으로, JEP 354는 JEP 325의 진화로 만들어 졌고 새로운 statement, yield 그리고 break 본래 의미 복원을 제안했다. (JEP 354는 2019년 6월 JDK 13을 대상으로 preview feature을 제공했다.)

Feedback on JDK 13 suggested that switch expressions were ready to become final and permanent in JDK 14 with no further changes.
JDK 13의 피드백은 switch expressions는 JDK 14에서 더이상 변경되지 않고 최종적이고 영구적이 될 준비가 되었다고 제안했다.


#### Motivation
As we prepare to enhance the Java programming language to support pattern matching (JEP 305), several irregularities of the existing switch statement -- which have long been an irritation to users -- become impediments. 
pattern matching 지원하여 Java를 향상시킬 준비를 할 때(JEP 305), 몇몇 불규칙성들이 -- 오랜시간 사용자에게 짜증나게 하는것 -- 장애가 되었다.

Java's switch statement의 현재 설계는 C and C++ 같은 언어를 따른다.

이전의 control flow은 low-level code 작성에는 유용하지만(such as parsers for binary encodings) switch가 higher-level contexts에서 사용되므로써 오류가 발생하기 쉬운 특성이 유연성보다 높아졌다.


#### 기존 switch 사용의 예
```
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        System.out.println(6);
        break;
    case TUESDAY:
        System.out.println(7);
        break;
    case THURSDAY:
    case SATURDAY:
        System.out.println(8);
        break;
    case WEDNESDAY:
        System.out.println(9);
        break;
}
```
많은 break statements는 불픽요하게 장황하고 visual noise(가독성?)는 종종 오류를 디버그하기 어렵게 만든다.


#### 신규 switch 사용의 예
```
switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> System.out.println(6);
    case TUESDAY                -> System.out.println(7);
    case THURSDAY, SATURDAY     -> System.out.println(8);
    case WEDNESDAY              -> System.out.println(9);
}
```

우리는 case L이 매칭되면 오른쪽의 label만 실행되는 새로운 형태의 switch label "case L ->" 소개를 제안한다. 또한 commas 구분으로 단일 case multiple constants 허용한다.

"case L ->" switch label 의 오른쪽 코드는 expression(표현식), block, throw statement로 제한된다.

이는 arm이 local variable를 도입해야하는 결과를 가져왔다. (switch block 안의 다른 arm에 대한 scope에는 속하지 않는다.) 그리고 이전에 전체 block이었던 switch block에서의 성가신것을 제거한다.


```
switch (day) {
    case MONDAY:
    case TUESDAY:
        int temp = ...     // The scope of 'temp' continues to the }
        break;
    case WEDNESDAY:
    case THURSDAY:
        int temp2 = ...    // Can't call this variable 'temp'
        break;
    default:
        int temp3 = ...    // Can't call this variable 'temp'
}
```
위 코드는 temp의 범위는 switch block 전체지만 각 arm에서는 사용할 수 없는 문제점을 보여준다.


```
int numLetters;
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        numLetters = 6;
        break;
    case TUESDAY:
        numLetters = 7;
        break;
    case THURSDAY:
    case SATURDAY:
        numLetters = 8;
        break;
    case WEDNESDAY:
        numLetters = 9;
        break;
    default:
        throw new IllegalStateException("Wat: " + day);
}
```
위 코드는 switch statements는 switch expressions의 시뮬레이션이고, 각 arm은 공통 target variable 또는 returns value 둘 중 하나에 할당하고 있다.

이 같은 표현은 우회적이며(roundabout), 반복적이고(repetitive) 오류발생이 쉽다(error-prone).

```
int numLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
    case WEDNESDAY              -> 9;
};
```
위 코드는 switch expression을 사용하였고 이 같은 switch expression은 이전의 코드보다 더 명확하고 안전한하다.


#### Description
##### 1) Arrow labels

"case L ->" labels을 추가하였고 만약 L이 매치되며 "->" 오른쪽을 실행한다.

```
For example, given the following switch statement that uses the new form of labels:

static void howMany(int k) {
    switch (k) {
        case 1  -> System.out.println("one");
        case 2  -> System.out.println("two");
        default -> System.out.println("many");
    }
}
```
```
The following code:

howMany(1);
howMany(2);
howMany(3);
results in the following output:

one
two
many
```

##### 2) Switch expressions

switch statement를 expression으로 사용할 수 있다.

```
static void howMany(int k) {
    System.out.println(
        switch (k) {
            case  1 -> "one";
            case  2 -> "two";
            default -> "many";
        }
    );
}
```
```
In the common case, a switch expression will look like:

T result = switch (arg) {
    case L1 -> e1;
    case L2 -> e2;
    default -> e3;
};
```

switch expression는 poly expression이다. 만약 target type을 알고있다면 target type
은 각 arm으로 적용된다.

type을 알고 있으면 switch expression의 type은 target type이다. 만약 그렇지 않으면 
standalone type은 각 case arm의 유형을 합하여 계산됩니다.
(arm에서 서로 다른 타입을 전달하면 어떻게 되는지??)

##### 3) Yielding a value

대부분의 switch expressions는 "case L ->" switch label 오른쪽에 single expression을 가지고 있다.

full block을 사용하고 return value가 필요한 경우 enclosing switch expression으로 yield를 사용한다.

```
int j = switch (day) {
    case MONDAY  -> 0;
    case TUESDAY -> 1;
    default      -> {
        int k = day.toString().length();
        int result = f(k);
        yield result;
    }
};
```
위 코드에서 yield result 를 사용하여 return 하고 있다.

yield는 이전의 "case L:" switch labels에서도 사용 할 수 있다.

```
int result = switch (s) {
    case "Foo": 
        yield 1;
    case "Bar":
        yield 2;
    default:
        System.out.println("Neither Foo nor Bar, hmmm...");
        yield 0;
};
```
위 코드는 이전 switch labels에서의 yield 사용을 예로 보여준다.

break 및 yield의 두 statements는 switch statements과 switch expressions 간에 쉽게 구분 할 수 있다. switch statement는 break statement의 target일 수 있지만 switch expression는 아니다. 반대로 switch expression은 yield statement의 target일 수 있지만 switch statement는 아니다. (이해가 안된다.)

yield는 제한된 식별자이다.(var) 클래스 이름으로 yield는 불가하다.


##### 4) Exhaustiveness

switch expression의 경우 가능한 모든 값에 매치되는 switch label이어야 한다.

실제로 이것은 default가 필요함을 의미한다. 그러나 알려진 모든 constants를 포괄하는 enum switch expression의 경우 enum definition가 compile-time과 runtime간에 변경되었음을 나타내는 default 절이 compiler에 의해 삽입됩니다.

implicit default clause 삽입을 사용하면 코드가 좋아진다. 이제 코드가 다시 컴파일 될 때 컴파일러는 모든 사례가 명시 적으로 처리되는지 확인합니다.

developer가 explicit default clause를 하면 가능한 error를 숨길 것이다.

switch expression은 value 또는 exception으로 완료하여한다.

이것은 많은 결과를 초래한다.


compiler는 모든 switch label에 대해 일치하고 값을 얻을 수 있는지 검사한다.
```
int i = switch (day) {
    case MONDAY -> {
        System.out.println("Monday"); 
        // ERROR! Block doesn't contain a yield statement
    }
    default -> 1;
};
i = switch (day) {
    case MONDAY, TUESDAY, WEDNESDAY: 
        yield 0;
    default: 
        System.out.println("Second half of the week");
        // ERROR! Group doesn't contain a yield statement
};
```
위 코드는 compiler의 switch label 확인 결과를 보여준다.

control statements, break, yield, return 및 continue는 다음과 같이 switch expression을 넘을 수 없다.
```
z: 
    for (int i = 0; i < MAX_VALUE; ++i) {
        int k = switch (e) { 
            case 0:  
                yield 1;
            case 1:
                yield 2;
            default: 
                continue z; 
                // ERROR! Illegal jump through a switch expression 
        };
    ...
    }
```


#### Dependencies
이 JEP는 JEP 325와 JEP 354로부터 확장되었지만 두 JEP에 의존하지 않는다.
JEP 305로 시작하는 패턴 일치에 대한 향후 지원은 이 JEP를 기반으로합니다.


## 362:	Deprecate the Solaris and SPARC Ports

## 363:	Remove the Concurrent Mark Sweep (CMS) Garbage Collector

Concurrent Mark Sweep (CMS) Garbage Collector를 삭제 한다고 한다.


#### Non-Goals
- 다른 GC 삭제는 목표는 아니다.
- JEP를 대상으로하는 릴리스와 이전의 릴리스에서 CMS GC 삭제는 목표가 아니다.


#### CMS GC를 삭제하는 이유.
2년전 JEP 291에서는 우리는 다른 collectors의 개발을 가속화 하고 미래 릴리즈에서 삭제를 위해 CMS collector 사용하지 않았다. 이 시간 동안, 신뢰할 수 있는 contributors가 CMS 유지 관리를 시작하지 않았다.(??이해를 못함.)

이 시간 동안, contributors는 CMS 유지 관리를 시작하지 않았다.

그리고 또한 JDK 6 CMS의 후속 모델인 G1의 추가 개선 사항과 함께 두가지 새로운 collectors를 소개하였다.(ZGC, Shenandoah)

이 시점에서 Hotspot JVM에서 사용 가능한 GC가 만약 CMS 성능을 능가하지 않는 경우 CMS를 제거하기에 충분한 오버 헤드가 있다.

향후 기존 GC의 개선으로 CMS의 필요성이 더욱 줄어들 것으로 예상된다.

#### 설명
이 변경으로 CMS 컴파일이 비활성화되고 소스트리에서 gc/cms 디렉토리의 내용이 제거되고 CMS에만 관련된 옵션이 제거된다.

또한 문서에서 CMS에 대한 References는 제거된다. 

그리고 CMS를 사용하려는 테스트는 제거되거나 필요에 따라 적용된다.

#### 메시지
Trying to use CMS via the -XX:+UseConcMarkSweepGC option will result in the following warning message:
```
Java HotSpot(TM) 64-Bit Server VM warning: Ignoring option UseConcMarkSweepGC; \
support was removed in <version>
and the VM will continue execution using the default collector.
```

#### 대안
CMS를 위한 코드는 repository 안에 유지할 수 있지만 컴파일 할 수 없습니다.

사용자는 G1 GC 또는 다른 collectors 옮길수 있습니다.

절대적으로 CMS가 필요한 사용자는 이전 릴리스에서 지원되는 동안 사용할 수 있습니다.



## 364:	ZGC on macOS
## 365:	ZGC on Windows
## 366:	Deprecate the ParallelScavenge + SerialOld GC Combination
## 367:	Remove the Pack200 Tools and API
## 368:	Text Blocks (Second Preview)
## 370:	Foreign-Memory Access API (Incubator)


