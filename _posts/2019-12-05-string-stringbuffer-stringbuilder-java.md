---
layout: single
title: 자바 문자열- String, StringBuilder, StringBuffer
tag: [java, string, stringbuilder, stringbuffer]
kinds: 포스트
toc: true
toc_sticky: true
---



String 문자열을 다루는 알고리즘을 공부하다가 문자열을 변경하기 위해서 Stringbuilder를 사용하는 예제 코드를 보았습니다. String 문자열의 수정이 잦을 경우에 성능 이슈 때문에 Stringbuilder와 StringBuffer 같은 가변(mutable)를 사용해야 한다는 점을 알고 있었지만 자세한 특징과 차이점을 알고자 정리하게 되었습니다. 



# String

자바의 String은 한번 만들어진 후 변경될 수 없는 불변(immutable)클래스 입니다. 메모리 사용량을 최적화하기 위해서 인터닝(interning)을 사용하기 때문입니다.  만약 + 연산자로 String 리터럴을 연산하거나 String 클래스에 substring(), replace() 등의 메서드를 이용해서 문자열을 조작하면 기존 문자열 자체가 수정되지 않고 heap 메모리에 새로운 객체가 생성됩니다. 

인터닝이란, 각 **String 리터럴**에 대한 단 하나의 사본을  String Pool에 저장하고 참조해서 사용하는 것 입니다. 새로운 String 리터럴을 생성하면 JVM은 먼저 Pool에서 동등한 값이 있는지 확인합니다. 만약 동일한 값을 찾을 경우 추가적인 메모리 할당 없이 그 값의 메모리 주소를 참조합니다.

**new** 연산자를 사용해서 만든 **String 객체**는 모두 새로운 객체이므로 heap 메모리에 저장됩니다. 필요할 경우 String 객체를 intern() 메서드를 사용해서 수동으로 인터닝 할 수 있습니다.

java7 이전에는 String Pool은 고정된 사이즈 였고 OutOfMemory 에러가 발생 가능성이 많았지만, java7 부터는  String Pool이 heap 메모리 영역에 위치하므로 가비지 컬렉터에 관리를 받으므로 에러 가능성이 감소했습니다.

<br><br>

# StingBuffer , Stringbuilder

String 연산은 무거운 작업이기 때문에 java는 가변 클래스인 StringBuffer와 StringBuilder를 제공합니다. 이 2개의 클래스는 문자열을 서로 연결하거나 자르는 등, 문자열 연산에 사용합니다. 

둘이 제공하는 기능은 동일하지만 차이점이 있는데 StringBuffer는 스레드 세이프이지만 비교적 성능이 떨어지고 StringBuilder는 동기화를 보장하지 않지만 성능이 더 좋다는 점입니다.

<br><br>

# String 비교하기

2개의 String 문자열을 비교할때는 equals() 메서드를 사용해서 비교해야 합니다. "==" 비교 연산자는 레퍼런스를 비교하기 때문입니다.(같은 객체를 참조하고 있는지)

```java
String str = "abc";
String str2 = "abc";
String str3 = new String("abc");

System.out.println(str == str2); //true
System.out.println(str == str3); //false
System.out.println(str.equals.str3); //true
```

<br>

String 객체와 리터럴을 비교할때에는 아래처럼 equals() 메서드에 인자로 String 객체를 전달해야 NullPointException을 피할 수 있습니다. equals() 메서드 내부적으로 객체가 null 값을 가지고 있는지 확인하기 때문입니다.

```java
String str = null;
//good
if("abc".equals(str)){
	//코드
}

//bad, NullPointException 가능성
if(str.equals("abc")){
    //코드
}
```

<br>



**출처**

- https://www.baeldung.com/java-string-pool
- https://www.slipp.net/questions/271
- https://minwan1.github.io/2018/06/11/2018-06-11-String-StringBuffer-StringBuilder/