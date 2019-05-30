---
  layout: single
  title: 오라클 JOIN
  tag: [oracle, join]
---

JOIN은 2개 이상의 테이블의 테이터를 조합한 결과를 만들때 사용한다. 오라클은 non-ANSI와 ANSI 조인(join) 문법을 제공한다. non-ANSI 조인은 오래전부터 사용되었고 지금도 여전히 인기가 많다. 이 문법은 FROM 절에 테이블을 열거하고 그 뒤에 WHERE 절에서 조인 조건을 정의한다. 이후 오라클 9i에서 도입된 ANSI 조인은 여러 이점이 있지만 오라클 개발자들은 습관적으로 또는 실행 단계에서 대부분의 ANSI 조인 문법은 오라클 옵티마이저에 의해 non-ANSI로 바뀌기 때문에 여전히 non-ANSI 조인 문법을 사용한다. (대부분 외형만 다를 뿐 성능은 동일하다는 의미)

하지만 언어 사용시 명확성을 따져보았을때 ANSI 조인 문법을 사용하는게 좋다.

<br>

# 오라클 ANSI 조인 문법

ANSI 조인 문법과 같거나 비슷한 non-ANSI 조인 문법을 정리되어 있습니다.

- INNER JOIN
- LEFT OUTER JOIN
- RIGHT OUTER JOIN
- FULL OUTER JOIN
- CROSS JOIN
- NATURAL JOIN
- INNER JOIN USING 

※( )안에 키워드는 생략 가능함. 테이블, 칼럼명은 한글로 표시

<br>

<br>

## (INNER) JOIN ... ON

양쪽 테이블의 특정 칼럼에서 일치하는 테이터만 결과로 보여준다.

*ANSI*

```
SELECT d.부서이름,
       e.직원이름
FROM   부서_테이블 d
       JOIN 직원_테이블 e ON d.부서ID = e.부서ID
WHERE  d.부서ID >= 30
ORDER BY d.부서이름;
```

쿼리 해석 ->

 부서와 직원 테이블에서 [부서 이름]과 [직원 이름]을 조합한 결과를 가져온다. Join 조건(ON)은 양 테이블의 부서ID가 같고 부서ID는 30이상(WHERE), 결과는 부서이름을 기준으로 오름차순(기본 값)으로 정렬(ORDER BY)한다.

*non-ANSI*

```
SELECT d.부서이름,
       e.직원이름
FROM   부서_테이블 d, 직원_테이블 e
WHERE  d.부서ID = e.부서ID
AND    d.부서ID >= 30
ORDER BY d.부서이름;
```

<br>

## LEFT [OUTER] JOIN

조인하는 테이블 중 왼쪽(부서 테이블)의 데이터 중 조건에 맞지 않는 것도 포함한 결과를 보여준다. 조건에 해당하지 않는 일부 row의 데이터(일치하는 부서ID가 없는 직원 테이블의 직원 이름 칼럼)는 null로 표시된다.

ON으로 일치하는 칼럼 지정해야 한다. 결과는 일치하지 않는 것도 포함한다. (왼쪽 테이블에 한해서 -> 부서_테이블 )

※[ ]는 필터, join 문법별 추가 필터의 위치와 형태가 다름

*ANSI*

```
SELECT d.부서이름,
       e.직원이름     
FROM   부서_테이블 d
       LEFT OUTER JOIN 직원_테이블 e ON d.부서ID = e.부서ID [AND e.연봉 >= 2000]
WHERE  d.부서ID >= 30
ORDER BY d.부서이름, e.직원이름;
```

*non-ANSI*

```
SELECT d.부서이름,
       e.직원이름      
FROM   부서_테이블 d, 직원_테이블 e
WHERE  d.부서ID = e.부서ID (+) 
[AND    e.연봉 (+) >= 2000]
AND    d.부서ID >= 30
ORDER BY d.부서이름, e.직원이름;
```

<br>

## RIGHT [OUTER] JOIN

위 LEFT JOIN에 반대다. LEFT를 RIGHT로 바꿔주면 된다. 

*ANSI*

```
SELECT d.부서이름,
       e.직원이름     
FROM   부서_테이블 d
       RIGHT OUTER JOIN 직원_테이블 e ON d.부서ID = e.부서ID [AND e.연봉 >= 2000]
WHERE  d.부서ID >= 30
ORDER BY d.부서이름, e.직원이름;
```

*non ANSI*

이 문법은 ANSI 조인과는 반대로 조인하는 테이블의 순서는 상관없다. LEFT, RIGHT의 구분이 의미가 없다. 그냥 OUTER 조인이다.

<br>

## FULL [OUTER] JOIN

양쪽 테이블 사이에서 조건에 일치하는 row와 조건에 일치하지 않는 row를 모두 포함한 결과를 보여준다. 데이터가 없을 경우 null로 표시된다.

*ANSI*

```
SELECT d.부서이름,
       e.직원이름     
FROM   부서_테이블 d
       FULL OUTER JOIN 직원_테이블 e ON d.부서id = e.부서id
WHERE  d.부서id >= 30
ORDER BY d.부서이름, e.직원이름;
```

*non ANSI*

비슷하게 대응되는 조인 문법은 없다. 하지만 UNION ALL 키워드를 이용해서 같은 결과를 만들 수 있다.

<br>

## CROSS JOIN

양쪽 테이블 사이에서 모든 가능한 조합을 결과로 보여준다.6개의 행을 가진 테이블과 7개의 행을 가진 테이블을 cross 조인한 결과는 42개의 행을 보여준다.

두 테이블간의 가능한 모든 조합을 만들어야 할때 사용한다.  테스트용 데이터를 생성하거나 중개 수수료 어플을 만들때 -> [스택오버플로우-답변](https://stackoverflow.com/a/220042)

*ANSI*

```
SELECT e.직원이름,
       d.부서이름
FROM   직원_테이블 e
       CROSS JOIN 부서_테이블 d
ORDER BY e.직원이름, d.부서이름;
```

*non ANSI*

```
SELECT e.직원이름,
       d.부서이름
FROM   직원_테이블 e, 부서_테이블 d
ORDER BY e.직원이름, d.부서이름;
```

<br>

## NATURAL JOIN

양쪽 테이블 사이에서 이름이 같은 칼럼으로 조인한 결과를 보여준다. (별도의 조건을 지정할 필요 없음)

문법과 실행결과가 심플한 이점이 있지만 동일한 칼럼에 대한 표기가 명확하지 않은 점, 추후 테이블에 칼럼이 추가될 경우 버그가 발생할 우려가 있기 때문에 사용시 주의가 필요함.

※WHEHE 절과 alias를 함께 사용할 수 없다. [ ] 안 처럼 사용할 수 있다.

*ANSI*

```
SELECT e.직원이름,
       d.부서이름
FROM   직원_테이블 e
       NATURAL JOIN 부서_테이블 d
[WHERE  부서id = 20]
ORDER BY e.직원이름, d.부서이름;
```

*non ANSI*

없음. non ANSI 조인 문법에서는 꼭 조건이 필요하다.

<br>

## [INNER] JOIN ... USING

NATURAL JOIN과 거의 비슷하다. 하지만 이 조인은 칼럼 이름을 특정한다.

칼럼 이름을 USING으로 특정하므로 추후 테이블 칼럼 추가에 영향을 받지 않는다. 

※WHEHE 절과 alias를 함께 사용할 수 없다. [ ] 안 처럼 사용할 수 있다.

*ANSI*

```
SELECT e.직원이름,
       d.부서이름
FROM   직원_테이블 e
       JOIN 부서_테이블 d USING (부서id)
[WHERE  부서id = 20]
ORDER BY e.직원이름;
```

