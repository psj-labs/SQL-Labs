## 내부 조인 (Inner Join)
두 테이블에서 **조건을 만족하는 행만** 가져오는 조인 방식.

    SELECT A.col, B.col
    FROM 테이블A A
    JOIN 테이블B B
      ON A.공통컬럼 = B.공통컬럼;

예전에는 JOIN 키워드 없이 다음과 같이 사용했다 → **암시적 Inner Join**

    SELECT ...
    FROM 테이블A A, 테이블B B
    WHERE A.공통컬럼 = B.공통컬럼;

---

## 셀프 조인 (Self Join)
**하나의 테이블을 두 번 불러서 내부 값을 비교**하는 조인.

### 예시: 동명이인 찾기

    SELECT DISTINCT s1.sno, s1.sname
    FROM student s1, student s2
    WHERE s1.sname = s2.sname
    AND s1.sno != s2.sno;

### 왜 DISTINCT?
`sno, sname` 조합이 같아야 중복이 제거된다.  
이름은 같지만 학번이 다르므로 DISTINCT가 필요하다.

### 표준 JOIN 문법

    SELECT DISTINCT s1.sno, s1.sname
    FROM student s1
    JOIN student s2
      ON s1.sname = s2.sname
     AND s1.sno != s2.sno;

➡ 하나의 테이블을 두 번 사용하므로 **Self Inner Join**.

---

## 외부 조인 (Outer Join) — Oracle (+)
Oracle 고전 문법에서 사용된 외부조인 방식.

### 예시: 교수가 있어도 담당 과목이 없을 수 있는 경우

    SELECT p.pname, c.cname
    FROM professor p, course c
    WHERE p.pno = c.pno(+)
    ORDER BY p.section;

➡ (+) 가 `course` 쪽에 있으므로  
**과목이 없어도 교수는 출력된다 → LEFT OUTER JOIN과 동일**

### 표준 문법

    SELECT p.pname, c.cname
    FROM professor p
    LEFT JOIN course c
      ON p.pno = c.pno
    ORDER BY p.section;

---

## 조인 표 핵심 요약

| 종류       | 설명                       | 예시 키워드                |
|-----------|---------------------------|-------------------------|
| 내부 조인   | 조건이 맞는 행만 출력     | JOIN / INNER JOIN            |
| 셀프 조인   | 같은 테이블을 두 번 비교 | A, A(별칭 두 개)                 |
| 외부 조인   | 매칭 실패한 행도 출력     | LEFT JOIN / (+)              |

---

## 마무리
조인은 결국 **“어떤 행을 남길지, 무엇을 기준으로 매칭할지 선택하는 과정”**이다.  
표준 JOIN 문법으로 이해하는 것이 가장 중요하다.
