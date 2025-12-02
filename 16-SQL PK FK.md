## 제약조건(Constraints) 개요

제약조건은 **테이블 단위**로 정의되며 데이터의 **삽입 / 수정 / 삭제** 시  
규칙을 적용하여 무결성을 보장한다.

### 주요 제약 유형

- **PK (PRIMARY KEY)**  
- **FK (FOREIGN KEY)**  
- **UK (UNIQUE)**  
- **NN (NOT NULL)**  
- **CK (CHECK)**  

> 핵심  
> 관계(부모 → 자식)가 존재할 경우  
> **FK가 부적절한 삽입·수정·삭제를 차단하여 데이터 정합성 유지**

---

## PK – Primary Key

### 개념 및 특징

- 테이블당 **1개만 존재**
- 모든 행을 **유일하게 식별**
- **중복 불가 / NULL 불가**
- 생성 시 **고유 인덱스 자동 생성**
- 모든 속성은 PK에 **함수 종속**

**예시**

> 이름 ❌ (중복 가능, 변경 가능)  
> 학번 ✅ (유일·불변)

---

## FK – Foreign Key

### 개념 및 특징

- 테이블 사이의 **관계(Relationship)** 표현
- **부모(PK/UK) → 자식(FK)**
- 참조 컬럼은 반드시 **데이터 타입 일치**

> FK 의미  
> **"이 값은 반드시 부모 테이블 키에 존재해야 한다"**

---

## PK / FK 설정

### 컬럼 레벨 설정

```sql
CREATE TABLE child (
  cid NUMBER CONSTRAINT pk_child PRIMARY KEY,
  pid NUMBER CONSTRAINT fk_child_parent
             REFERENCES parent(pid)
);
```

---

### 테이블 레벨 설정

```sql
CREATE TABLE child (
  cid NUMBER,
  pid NUMBER,
  CONSTRAINT pk_child PRIMARY KEY (cid),
  CONSTRAINT fk_child_parent FOREIGN KEY (pid)
      REFERENCES parent(pid)
      ON DELETE CASCADE
);
```

**ON DELETE CASCADE**

- 부모 삭제 → 자식 자동 삭제
- **존속 의존 관계에만 제한적으로 사용**

---

### 활성 / 비활성

```sql
ALTER TABLE 테이블명 DISABLE CONSTRAINT 제약이름;
ALTER TABLE 테이블명 ENABLE  CONSTRAINT 제약이름;
```

대량 적재 전 임시 해제 후 재적용.

---

## 제약조건 조회

```sql
SELECT c.table_name, c.constraint_name, c.constraint_type, 
       c.status, s.column_name
FROM   user_constraints c
JOIN   user_cons_columns s
  ON   c.constraint_name = s.constraint_name
WHERE  c.table_name IN ('DEPT','EMP')
ORDER  BY c.table_name;
```

#### 부모–자식 관계 조회

```sql
SELECT  p.table_name AS parent_table,
        c.table_name AS child_table,
        c.constraint_name
FROM    user_constraints p
JOIN    user_constraints c
  ON    c.r_constraint_name = p.constraint_name
ORDER BY p.table_name;
```

---

## 실습 – DEPT / EMP 테이블

### 초기화

```sql
DROP TABLE emp;
DROP TABLE dept;
PURGE RECYCLEBIN;
```

---

### 테이블 생성

```sql
CREATE TABLE dept (
  dno   VARCHAR2(2),
  dname VARCHAR2(15),
  loc   VARCHAR2(9),
  CONSTRAINT dept_dno_pk PRIMARY KEY (dno)
);

CREATE TABLE emp (
  eno   VARCHAR2(4),
  ename VARCHAR2(15),
  mgr   VARCHAR2(4),
  hdate DATE,
  sal   NUMBER,
  comm  NUMBER,
  dno   VARCHAR2(2),
  CONSTRAINT emp_eno_pk PRIMARY KEY (eno),
  CONSTRAINT emp_mgr_fk FOREIGN KEY (mgr) REFERENCES emp(eno),
  CONSTRAINT emp_dno_fk FOREIGN KEY (dno) REFERENCES dept(dno)
);
```

---

### 데이터 입력

```sql
INSERT INTO dept VALUES ('10','개발','서울');
INSERT INTO emp (eno, ename, dno) VALUES ('2000','문시현','10');
COMMIT;
```

---

## 무결성 에러 예시

| 동작 | 결과 |
|------|------|
| PK 중복 INSERT | ORA-00001 |
| FK 없는 값 INSERT | ORA-02291 |
| FK 참조 부모 삭제 | ORA-02291 |
| FK 존재 안 하는 값 UPDATE | ORA-02291 |

---

## ERD 관계 해석

### 관계 구조

- **DEPT (부모) 1 → N EMP (자식)**
- **EMP 자기참조 관계**  
  `EMP.mgr → EMP.eno`

---

### 테이블 구성

#### DEPT

| 컬럼 | 타입 | 제약 |
|------|------|------|
| dno | VARCHAR2(2) | PK |
| dname | VARCHAR2(15) | - |
| loc | VARCHAR2(9) | - |

#### EMP

| 컬럼 | 타입 | 제약 |
|------|------|------|
| eno | VARCHAR2(4) | PK |
| ename | VARCHAR2(15) | - |
| mgr | VARCHAR2(4) | FK → EMP(eno) |
| hdate | DATE | - |
| sal | NUMBER | - |
| comm | NUMBER | - |
| dno | VARCHAR2(2) | FK → DEPT(dno) |

---

## 실습 과제

### 공장–제품–출고 모델

```sql
CREATE TABLE factory (
  fno   VARCHAR2(4),
  fname VARCHAR2(40),
  loc   VARCHAR2(40),
  CONSTRAINT pk_factory PRIMARY KEY (fno)
);

CREATE TABLE goods (
  gno     VARCHAR2(6),
  gname   VARCHAR2(60),
  pri     NUMBER(10,2),
  fac_no  VARCHAR2(4),
  CONSTRAINT pk_goods PRIMARY KEY (gno),
  CONSTRAINT fk_goods_factory FOREIGN KEY (fac_no)
    REFERENCES factory(fno)
);

CREATE TABLE prod (
  s_num NUMBER,
  gno   VARCHAR2(6),
  pri   NUMBER(10,2),
  pdate DATE,
  CONSTRAINT pk_prod PRIMARY KEY (s_num),
  CONSTRAINT fk_prod_goods FOREIGN KEY (gno)
    REFERENCES goods(gno)
);
```
---

## 요약

**SQL 제약조건은 PK와 FK로 무결성을 강제하여 데이터 구조의 신뢰성을 지키는 핵심 장치다.**
