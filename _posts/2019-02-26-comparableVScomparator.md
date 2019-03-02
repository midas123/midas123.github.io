---
  layout: single
  title: Java - Comparable와 Comparator 인터페이스
  tag: [java]
---


java.lang.Comparable과 java.util.Comparator 인터페이스는 collection의 객체를 정렬하는데 쓰인다. 자바에는 이미 array와 list의 primitive 타입의 데이터와 String, Wrapper 클래스 데이터를 정렬해주는 내장 메서드가 있지만 커스텀 클래스를 위한 정렬 메서드는 없다. 

커스텀 클래스의 객체를 Arrays 또는 Collections의 sort() 메서드를 이용해서 정렬하려면 먼저 커스텀 클래스에서 Comparable 인터페이스를 구현해야 한다. 이 인터페이스는 compareTo(Object obj) 메서드를 가지고 있다. 이 메서드는 앞에 객체가 클 경우 양수, 같을 경우 0, 작을 경우 음수를 반환한다. 비교할 수 없을 경우는 ClassCastException이 발생한다.(객체의 커스텀 클래스가 Comparable 인터페이스를 구현하지 않은 경우)   이 메서드는 알파벳, 숫자 등 간단한 정렬에 적합하다.



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



앞서 말했듯이 compareTo()메서드는 하나의 객체에 대해서 기본적인 비교만 가능하기 때문에 비교 메서드를 여러 번, 다른 방법으로 정의하고 싶을때 Comparator 인터페이스의 compare(Object o1, Object o2) 메서드를 사용한다. compare() 메서드는 o1이 o2보다 작을 경우 음수를, 같거나 클 경우 0을 반환한다.

Comparator 인터페이스의 compare() 메서드를 sort() 메서드의 파라미터로 전달하기 위해서 익명 클래스나 람다 표현식(java8부터)을 사용해야 한다.

-리스트 생성

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



-익명 클래스(anonymous class) 사용

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

-람다 표현식

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





참고:

[Comparable VS Comparator 스택오버플로우 답변](https://stackoverflow.com/a/4108616/10999770)

[comparable-and-comparator-in-java-example](https://www.journaldev.com/780/comparable-and-comparator-in-java-example)

[sorting-with-comparable-and-comparator-in-java.html](https://www.javaworld.com/article/3323403/learn-java/java-challengers-5-sorting-with-comparable-and-comparator-in-java.html?page=2)

[객체 비교시 (Object.a - Object.b) Integer flow에 대해서](https://stackoverflow.com/questions/2728793/java-integer-compareto-why-use-comparison-vs-subtraction)

[java-8-sort-lambda](https://www.baeldung.com/java-8-sort-lambda)

