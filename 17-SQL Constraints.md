## PK (Primary Key, 기본키)

**기본키(PK)**는 테이블 내에서 각 행(row)을 유일하게 식별하는 컬럼입니다.

-   중복 불가 (`UNIQUE`)
-   NULL 불가 (`NOT NULL`)
-   값이 변하지 않는 것이 원칙 (데이터의 ‘정체성’ 역할)
-   테이블당 하나만 지정 가능

```

CREATE TABLE student (
  sno CHAR(6) PRIMARY KEY,
  sname VARCHAR2(20),
  dept VARCHAR2(20)
);
```

**설명:** 학번(`sno`)이 학생의 고유 식별자 역할을 하며, 중복되거나 NULL일 수 없습니다.

\---

## FK (Foreign Key, 외래키)

**외래키(FK)**는 다른 테이블의 **PK**를 참조하여 테이블 간의 관계를 정의합니다.

-   자식 테이블이 부모 테이블의 PK를 참조
-   부모의 PK가 존재해야 자식의 FK가 입력 가능
-   부모 데이터 삭제 시, FK 제약 조건 위반 발생 가능

```

CREATE TABLE enroll (
  sno CHAR(6),
  cno CHAR(4),
  CONSTRAINT fk_enroll_student FOREIGN KEY (sno)
      REFERENCES student(sno)
);
```

| 구분 | 역할 | 예시 | 특징 |
| --- | --- | --- | --- |
| 부모 테이블 | 참조되는 쪽 (PK 존재) | student | PK 존재해야 참조 가능 |
| 자식 테이블 | 참조하는 쪽 (FK 존재) | enroll | 부모 PK를 바라봄 |

**참조 무결성(Referential Integrity)**  
부모 테이블의 값이 자식 테이블에서 참조되고 있을 경우, 부모의 값을 함부로 삭제하거나 변경할 수 없습니다.

\---

## UK (Unique Key, 고유키)

**Unique Key**는 특정 컬럼의 값이 **중복되지 않도록** 보장하지만, `NULL`은 허용됩니다.

```

CREATE TABLE emp3 (
  eno VARCHAR2(4),
  gno VARCHAR2(13),
  CONSTRAINT emp4_gno_uk UNIQUE (gno)
);
```

예: 주민번호(gno)는 중복되면 안 되지만, 입력이 없을 수도 있으므로 `NULL`은 허용합니다.

\---

## NOT NULL 제약조건

해당 컬럼에 **NULL 값을 허용하지 않음**을 의미합니다.

```

CREATE TABLE emp3 (
  ename VARCHAR2(50) CONSTRAINT emp4_nu_ename NOT NULL
);
```

**NOT NULL**은 반드시 **컬럼 레벨**에서만 정의 가능하며, `ALTER TABLE ... MODIFY`로 추가합니다.

\---

## CHECK 제약조건

**입력값의 조건**을 지정하여 데이터의 유효성을 검사합니다.

```

CREATE TABLE emp3 (
  sex VARCHAR2(2),
  CONSTRAINT emp4_sex_ch CHECK (sex IN ('남','여'))
);
```

예시: 성별 컬럼은 반드시 `'남'` 또는 `'여'`만 입력 가능.

\---

## 제약조건 추가 / 삭제 / 비활성화

### 1) 제약조건 추가

```

ALTER TABLE class
ADD CONSTRAINT class_cno_pk PRIMARY KEY (cno);

ALTER TABLE st
ADD CONSTRAINT st_cno_fk FOREIGN KEY (cno)
REFERENCES class (cno);
```

### 2) 제약조건 삭제

```

ALTER TABLE st DROP CONSTRAINT st_cno_fk;
ALTER TABLE class DROP CONSTRAINT class_cno_pk;
```

### 3) 제약조건 비활성화 / 활성화

```

ALTER TABLE st DISABLE CONSTRAINT st_sno_pk;
ALTER TABLE st ENABLE CONSTRAINT st_sno_pk;
```

**DISABLE 시 인덱스도 함께 삭제**됩니다.  
PK 또는 UK를 비활성화하려면 해당 키를 참조 중인 FK부터 비활성화해야 합니다.

\---

## PK와 FK 관계 요약

| 항목 | 변경 가능 여부 | 설명 |
| --- | --- | --- |
| PK 값 | ❌ 거의 불가 | 참조 중인 FK가 존재하면 변경 불가 |
| FK 값 | ⭕ 제한적 가능 | 부모에 존재하는 값으로만 변경 가능 |

**정리:**  
PK는 테이블의 유일한 식별자이며 변하지 않는 값.  
FK는 자식이 부모를 참조하는 연결 키.  
부모가 바뀌면 자식이 영향을 받지만, 자식이 바뀌어도 부모의 존재는 그대로 유지됨.

---

## 실습 예시

```

CREATE TABLE class (
  cno VARCHAR2(4),
  cname VARCHAR2(50)
);

CREATE TABLE st (
  sno VARCHAR2(4),
  sname VARCHAR2(50),
  cno VARCHAR2(4)
);

ALTER TABLE class ADD CONSTRAINT class_cno_pk PRIMARY KEY (cno);
ALTER TABLE class ADD CONSTRAINT class_cname_uk UNIQUE (cname);
ALTER TABLE st ADD CONSTRAINT st_sno_pk PRIMARY KEY (sno);
ALTER TABLE st ADD CONSTRAINT st_cno_fk FOREIGN KEY (cno) REFERENCES class (cno);
```

\---

## 요약 정리 표

| 제약조건 | 역할 | 중복 여부 | NULL 허용 여부 | 설정 위치 |
| --- | --- | --- | --- | --- |
| PK | 기본키, 유일 식별 | ❌ | ❌ | 테이블 레벨 / 컬럼 레벨 |
| FK | 참조키, 관계 설정 | ⭕ | ⭕ | 테이블 레벨 |
| UK | 고유 제약조건 | ❌ | ⭕ | 테이블 / 컬럼 레벨 |
| NOT NULL | NULL 금지 | ⭕ | ❌ | 컬럼 레벨만 |
| CHECK | 조건 검사 | ⭕ | ⭕ | 컬럼 / 테이블 레벨 |

---
