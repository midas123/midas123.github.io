---
 layout: single
 title: 피보나치 수열과 재귀 함수 & call stack
 tag: [java, recursion, algorithm, call stack]
 kinds: 포스트
---

피보나치수열을 재귀함수로 구현하는 방법을 공부하다가 재귀함수에 대해 더 깊은 이해를 하게 되어 이렇게 간단하게 정리하게 되었습니다. 

이전까지 제가 이해한 재귀함수는 아래 코드처럼 함수가 함수 안에서 자기자신을 호출하는 것이었습니다.

```java
public int someMethod(){
//코드
return someMethod();
}
```

물론 기본 조건(base case) 없이 위 코드 처럼 함수를 호출하면 무한 재귀 호출에 빠지게 됩니다. 그러므로 기본 조건을 넣어서 특정 조건에서 재귀 호출이 멈추도록 해야 합니다. 아래 코드는 n이 0과 같아지면 n을 리턴하면서 재귀 호출이 멈춥니다.

```java
public int someMethod(int n){
	if(n == 0){
	   return n; 
	} else {
	   return someMethod(n);	
	}
}
```

여기까지가 제가 그 동안 이해했던 재귀함수의 개념이었습니다. 

재귀함수에 대해 새로 알게된 점을 정리하기 전 먼저 피보나치 수열에 대해 알아보겠습니다. [피보나치 수열](https://ko.wikipedia.org/wiki/%ED%94%BC%EB%B3%B4%EB%82%98%EC%B9%98_%EC%88%98)에 법칙대로 '앞에 있는 두 수의 합은 그 뒤에 있는 수가 된다' 는 조건을 정의했습니다. 이 수열을 자바 배열로 구현한다면 n이 2보다 크거나 같은 조건에서 아래 공식이 성립합니다.

```java
number[n] = number[n-1] + number[n-2]
```

위에서 정의한 조건으로 재귀함수를 이용해서 피보나치 수열을 구하는 자바 코드를 만들었습니다. 아래 코드에서 fiboArrayByRecursion() 메서드는 수열 size 만큼의 비어있는 배열과 int n을 인자로 받아서 피보나치 수열을 담은 배열을 리턴 합니다. 

```java

public class Fibo {
	public static void main(String[] args) {
		int size = 7;
		int[] numbers = new int[size];
		int n=0;
		System.out.println(Arrays.toString(fiboArrayByRecursion(numbers, n)));
		//[0, 1, 1, 2, 3, 5, 8]
	}


        static public int[] fiboArrayByRecursion(int[] array, int n) {
                if(n < array.length) {
                    if(n>1) {
                        array[n] = array[n-1] + array[n-2];
                    } else {
                        array[n] = n;
                    }
                    n++;
                    return fiboArrayByRecursion(array, n);
                } else {
                    return array;
                }
            }
}
```

나름데로 맞는 답이긴 하지만 외부에서 기본 조건에 사용할 int 값과 비어있는 배열을 메서드 인자로 전달 하는 부분이 왠지 찝찝합니다. 

이 찝찝함을 해소하기 위해서 비로소 구글링을 해보았습니다. 그리고 아래와 같은 코드를 찾았습니다.

```java
static int fiboNumberByRecursion(int index) {
		if(index == 0 || index == 1) {
			return index;
		}else {
			return fiboNumberByRecursion(index-1) + fiboNumberByRecursion(index-2);
		}
	}
```

fiboNumberByRecursion() 메서드는 특정 항(index)의 피보나치 수열 값을 구합니다. 이 함수의 기본조건에 따라서 index가 0 또는 1일 경우 index 값을 리턴하면서 재귀 함수 호출이 멈추게 됩니다. 

그런데 else 조건 안에서 return 하는 부분이 잘 이해가 되지 않습니다. index가 0 또는 1이 되기 전까지fiboNumberByRecursion()를 여러번 호출하고 그 리턴 값을 더한 결과를 최종적으로 리턴하게 된다는 표면적인 부분은 이해가 되지만 그 자세한 과정이 머리 속에 그려지지 않습니다. 

일단 index 값을 대입해서 메서드가 실행된 결과를 노트에 손으로 써가는 과정에서 '[재귀](https://ko.wikipedia.org/wiki/%EC%9E%AC%EA%B7%80_(%EC%BB%B4%ED%93%A8%ED%84%B0_%EA%B3%BC%ED%95%99))'라는 단어에 뜻을 다시 한번 생각해보았습니다. ''자기 자신에게 되돌아온다.'' 또는 ''자기자신을 호출 한다.'' 그 전까지는 ''자기 자신을 호출한다.''는 부분에만 집중해서 "되돌아 온다"는 부분에 대해 미쳐 생각하지 못했던 것 입니다.   

''메서드가 반복해서 호출되고 그 메서드의 리턴 값이 누적되어 가장 처음으로 돌아오게 된다.''

그런데 어떤 원리로 이게 가능한 걸까요? 궁금해서 좀 더 검색해 보았습니다. 

java에서는 [**call stack**](https://en.wikipedia.org/wiki/Call_stack)이라는 자료구조를 사용합니다. call stack은 프로그램의 서브루틴을 저장합니다. call stack은 하나의 큰 stack 프레임 입니다. 위 코드의 베이스 조건에 맞게 호출된 fiboNumberByRecursion() 메서드는 먼저 호출된 순서로 stack 프레임 안에 추가 됩니다. 

stack은 후입선출(LIFO) 방식으로 데이터(메서드)가 추가/삭제되는 메모리 영역입니다. 그러므로 가장 나중에 호출된 메서드부터 실행됩니다. 

fiboNumberByRecursion() 메서드가 피보나치 수열을 Recursion(재귀)로 구현한 과정을 풀어보면, 먼저 입력 받은 index 값이 0 또는 1과 같아지는 if 조건에 만족할때까지 계속해서 메서드를 재귀 호출합니다. 그리고 if 조건에 만족하면 스택 프레임 안에서 가장 마지막에 있는 메서드부터 먼저 값을 리턴합니다. 즉, 피보나치 수열의 첫째항부터 항의 값부터 순서대로 계산하고 최종적으로 입력 받은 index 항의 값을 리턴합니다.

