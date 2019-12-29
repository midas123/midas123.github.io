---
  layout: single
  title: 객체 정렬하기 - Collections.sort와 Comparable, Comparator 
  tag: [java, Comparable, Comparator]
  kinds: 포스트
  toc: true
  toc_sticky: true
---

# java.util.Collections.sort()

자바 Collection 프레임워크는 객체 그룹을 저장하고 조작할 수 있는 다양한 구조체를 제공합니다. 이 구조체에 서는 대해서 탐색, 정렬, 삽입, 삭제 등에 조작이 가능합니다.

- interface - Set, List, Queue, Deque
- class - ArrayList, Vector, LinkedList, PriorityQueue, HashSet, LinkedHashSet, TreeSet



<br>이런 컬렉션을 정렬하려면 Collection.sort()를 사용합니다. 그런데 이 메서드가 동작하기 위한 조건이 있습니다. 컬렉션에 저장된 객체의 클래스가  java.lang.Comparable 인터페이스를 구현해야 합니다. 
만약 객체 클래스에서  Comparable를 구현하고 코드를 직접 수정할 수 없는 경우에는 별도로 Comparator 인터페이스를 구현하는 클래스를 만들고 객체를 비교하는 compare() 메서드를 오버라이드해서 정렬합니다.

String, Date 관련 클래스와 Integer, Double 등 primitive 타입에 맞춰 제공되는 wrapper 클래스는 이미 Comparable를 인터페이스를 구현하기 때문에 이러한 객체 타입이 저장된 컬렉션은 바로 Collection.sort() 메서드로 정렬할 수 있습니다. 이 메서드는 기본적으로 알파벳, 숫자 등을 오름차순으로 정렬합니다.

https://www.javatpoint.com/collections-in-java

<br>

##  java.lang.Comparable 

위에서 말한 wrapper 클래스처럼 자바에서 제공하는 클래스 객체는 컬렉션 정렬이 바로 가능합니다. 그러나 유저가 만든 커스텀 객체가 담긴 컬렉션을 정렬하려면 먼저 커스텀 객체 클래스에서 Comparable를 구현하고 compareTo() 메서드를 오버라이드 해야 합니다. 

예를 들어, 객체 간에 멤버 변수의 숫자 값을 비교하는 경우, compareTo() 메서드가 두 객체의 숫자를  "-" 산술 연산자로 계산한 결과를 리턴하도록 구현할 수 있습니다. 비교 기준은 리턴하는 결과 값이 양수일 경우 앞에 있는 피연산자가 더 큰 값이 되고, 0은 서로 동일한 경우, 음수는 앞에 있는 피연산자가 더 작은 경우가 됩니다. 
return 키워드에 이항 연산식 안에 피연산자의 순서가 정렬 순서(오름/내림차순)를 결정 합니다. Comparable를 구현한 객체 클래스의 this.멤버변수가 앞에 위치하고 compareTo() 메서드 인자로 전달 받은 외부객체의 멤버변수가 그 다음에 올 경우 정렬 결과는 오름차순이 됩니다. 그리고 이 순서를 반대로 하면 내림차순이 됩니다.

```java
import java.util.Map;
import java.util.TreeMap;

class personClass implements Comparable<Object> {
	private String name;
	private int age;
	
	public personClass(String name, int age) {
		this.name = name;
		this.age = age;
	}
	@Override
	public int compareTo(Object o) {
		personClass obj = (personClass) o;
		//return (this.age - obj.age); //primitive 타입 비교 그러나 Integer overflow 발생 가능성 있음
        //return (this.age > obj.age ? 1 : (this.age == obj.age ? 0 : -1)) //삼항식으로 대체
		//return this.name.compareTo(obj.name);//String 객체 비교, 알파벳 오름차순
		return obj.name.compareTo(this.name);//String 객체 비교, 알파벳 내림차순
	}
	@Override
	public String toString() {
		return "name:"+this.name +" "+"age:"+this.age;
	}
}

public class personSortingClass{
	public static void main(String[] args) {
		personClass obj1 = new personClass("john", 20);
		personClass obj2 = new personClass("aimy", 27);
		System.out.println(obj1.compareTo(obj2));
		
		Map<personClass, String> people = new TreeMap<>();
		people.put(obj1, "Employee");
		people.put(obj2, "Employee");
		people.put(new personClass("bob", 25), "Manager");
		
		System.out.println(people.toString()); 
		//출력 결과: {name:john age:20 rank=Employee, name:bob age:25 rank=Manager, name:aimy age:27 rank=Employee}
		//위에서 오버라이드한 compareTo() 반환 코드에서 비교 객체의 위치를 바꾸면서 내림차순으로 정렬됨
	}
}
```

<br>

## java.utilComparator 

Comparable를 직접 구현할 수 없는 경우, 다시 말해서 객체 클래스의 코드를 변경할 수 없는 경우, Comparator 인터페이스를 구현합니다. Comparator를 구현하는 별도의 클래스를 만든 후 비교할 두 객체를 인자로 전달하는compare(o1, o2) 메서드를 오버라이드 합니다. 이때 전달 받는 인자는 정렬하려는 클래스 객체의 타입 입니다.
또한, Comparator는 다양한 정렬 기준이 필요할때에도 사용합니다. 예를 들어 위 person 객체의 username을 기준으로 정렬하는 클래스와 age 순으로 정렬하는 클래스를 따로 구현해서 사용할 수 있습니다.
또한, 자바에서 제공하는 클래스 객체를 다른 기준으로 정렬시에도 사용할 수도 있습니다. 예를 들면 String 문자열을 알파벳 순서가 아니라 문자열 길이로 정렬할 때 입니다.

compare() 메서드는 o1이 o2보다 작을 경우 음수를, 같을 경우 0, 큰 경우 양수를 리턴하도록 구현합니다.

Comparator 인터페이스를 구현하는 클래스를 별도로 만드는 방법 외에 익명 클래스나 람다 표현식(java8부터)을 사용할 수도 있습니다.

<br>

**-리스트 생성**

```java
		personClass obj1 = new personClass("john", 20);
		personClass obj2 = new personClass("aimy", 27);
		personClass obj3 = new personClass("kim", 17);
		
		List<personClass> list1 = new ArrayList<>();
		list1.add(obj1);
		list1.add(obj2);
		list1.add(obj3);
		
		System.out.println(list1.toString());
		//[name:john age:20 rank, name:aimy age:27 rank, name:kim age:17 rank]
```



**-익명 클래스(anonymous class) 사용**

```java
	    Collections.sort(list1, new Comparator<personClass>(){
		@Override
		public int compare(personClass o1, personClass o2) {
			return Integer.compare(o1.getAge(), o2.getAge());//오름차순
			}
		});
		
		System.out.println(list1.toString());
		//[name:kim age:17 rank, name:john age:20 rank, name:aimy age:27 rank]

		Comparator<personClass> sortAge = new Comparator<personClass>(){
		@Override
		public int compare(personClass o1, personClass o2) {
			return Integer.compare(o2.getAge(), o1.getAge());//내림차순
			}
		};
		
		Collections.sort(list1, sortAge);
		System.out.println(list1.toString());
		//[name:aimy age:27 rank, name:john age:20 rank, name:kim age:17 rank]
```

**-람다 표현식**

```java
		list1.sort((personClass o1, personClass o2) -> Integer.compare(o1.getAge(), o2.getAge()));
		System.out.println(list1.toString());
		//[name:kim age:17, name:john age:20, name:aimy age:27]
		
		list1.sort(personClass::sortAge);//personClass에 static sortAge()
		System.out.println(list1.toString());
		//[name:aimy age:27, name:john age:20, name:kim age:17]
		
		Collections.sort(list1, Comparator.comparing(personClass::getAge));
		System.out.println(list1.toString());
		//[name:kim age:17, name:john age:20, name:aimy age:27]
		
		Comparator<personClass> com = (o1, o2) -> Integer.compare(o1.getAge(), o2.getAge());
		list1.sort(com.reversed());
		System.out.println(list1.toString());
		//[name:aimy age:27, name:john age:20, name:kim age:17]
```

<br>

## 결론

Comparable의 compareTo() 메서드와 Comparator의 compare() 메서드는 둘다 객체를 비교하는 기능을 합니다. 그러므로 상황에 적합한 메서드를 사용해서 정렬을 구현합니다. 

정렬하려는 클래스에서 인터페이스를 구현 가능하고 기본 정렬 기준을 사용한다면 Comparable를 사용하고 반대로 클래스 내의 코드를 추가/변경 할 수 없는 상황이거나 추가적인 정렬 기준이 필요하다면 Comparator를 사용합니다.

<br>

참고:

https://www.callicoder.com/java-comparable-comparator/

[객체 비교시 (Object.a - Object.b) Integer flow에 대해서](https://stackoverflow.com/questions/2728793/java-integer-compareto-why-use-comparison-vs-subtraction)

[java-8-sort-lambda](https://www.baeldung.com/java-8-sort-lambda)

