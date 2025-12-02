## 1) 인덱스란?

인덱스는 **테이블의 특정 컬럼 값을 빠르게 찾기 위해** 별도 구조(보통 B-Tree)를 만들어 둔 것입니다. 도서관의 찾기 쉬운 색인처럼, `WHERE` 조건이나 `JOIN`에 쓰이는 컬럼을 기준으로 _필요한 행만 먼저 좁혀_ 디스크 I/O를 줄입니다.

### 왜 빠를까?

-   정렬된(또는 압축된) 탐색 구조를 사용 → 범위/동등 조건 탐색이 O(log N) 수준으로 단축
-   필요한 ROWID(실제 행 위치)만 먼저 얻고, 그 후 테이블로 _정확히_ 찾아감

### 언제 유리할까?

-   `WHERE`로 **전체의 10~15% 정도**만 가져올 때
-   조인 키, 자주 필터링/정렬하는 컬럼일 때
-   행이 아주 많은 대용량 테이블

**주의:** 인덱스가 많다고 무조건 빠르지 않습니다. DML(INSERT/UPDATE/DELETE) 시 인덱스도 함께 갱신되므로 _과도한 인덱스는 쓰기 성능을 떨어뜨립니다._

---

## 2) 인덱스 종류

| 분류 | 유형 | 설명 / 사용처 |
| --- | --- | --- |
| 고유성 | **Unique Index** | 컬럼 값의 중복을 허용하지 않음(보통 PK/UK 제약으로 자동 생성). 동등 비교에 최적. |
| 고유성 | **Non-Unique Index** | 중복 허용. 자주 거르는 조건·조인 키에 수동 생성. |
| 물리 구조 | **B-Tree Index** | 가장 일반적. 동등/범위 조건, 정렬 최적화. |
| 물리 구조 | **Bitmap Index** | 저카디널리티(값 종류가 적은) 컬럼에 유리. 읽기 위주 환경에서 강력. |

**참고:** PK/UK 제약을 추가하면 해당 컬럼에 _Unique Index가 자동 생성_됩니다. FK는 자동 생성되지 않으므로 **FK 컬럼에는 수동 인덱스**를 권장합니다.

---

## 3) 인덱스 생성 기준 & 문법

-   전체의 약 **10~15%** 수준을 조회하는 조건
-   `WHERE` / `JOIN` / `ORDER BY`에 자주 쓰는 컬럼
-   대용량 테이블(행이 매우 많은 경우)

```
-- 단일 컬럼
CREATE INDEX idx_student_sname
  ON student (sname);

-- 복합(컬럼 순서 중요: 선행 컬럼이 필터/정렬에 먼저 쓰여야 효율적)
CREATE INDEX idx_student_major_sname
  ON student (major, sname);

-- 함수/수식 기반(Function-based Index)
CREATE INDEX idx_student_avr_expr
  ON student (avr*4.5/4.0);

-- FK 컬럼(조인/삭제 시 잠금 경합 감소에 도움)
CREATE INDEX idx_course_pno_fk
  ON course (pno);

CREATE INDEX idx_score_cno_fk
  ON score (cno);
```

**실전 팁:** 복합 인덱스는 _왼쪽부터_ 사용합니다. 예를 들어 `(major, sname)` 인덱스는 `WHERE major = ?` 또는 `WHERE major=? AND sname LIKE 'A%'`에 잘 듣지만, `WHERE sname='조운'` 단독 조건엔 효과가 제한적입니다.

---

## 4) 인덱스 조회(메타 정보 확인)

\* Oracle 데이터사전 뷰 예시

```
-- 인덱스와 컬럼(순서, 고유성) 확인
SELECT c.index_name,
       c.column_name,
       c.column_position,
       i.uniqueness
FROM   user_indexes i
JOIN   user_ind_columns c
  ON   c.index_name = i.index_name
WHERE  i.table_name IN ('STUDENT','PROFESSOR','COURSE','SCORE')
ORDER BY i.table_name, c.index_name, c.column_position;

-- 함수/수식 기반 인덱스의 식 확인
SELECT index_name, column_expression
FROM   user_ind_expressions
WHERE  index_name = 'IDX_STUDENT_AVR_EXPR';
```

**해석 포인트:** `column_position`은 복합 인덱스에서 컬럼의 순서를 의미합니다. `column_expression`은 함수/수식 기반 인덱스의 실제 표현식을 보여줍니다.

---

## 5) FK 컬럼에 인덱스가 필요한 이유

-   부모 테이블 삭제/갱신 시 자식 테이블에서 관련 행을 _빠르게 찾기 위해_
-   자식 테이블 → 부모 테이블 `JOIN` 성능 향상
-   락 경합(또는 테이블 전체 스캔) 방지

```
-- 예: 자식 SCORE.cno, COURSE.pno
CREATE INDEX score_cno_fk ON score (cno);
CREATE INDEX course_pno_fk ON course (pno);
```

---

## 6) 인덱스 삭제 & 제약조건과의 관계

-   **PK/UK로 생성된 Unique Index는 직접 DROP 불가** → 제약을 `DISABLE`하거나 삭제해야 인덱스가 제거됩니다.

```
-- 수동 생성한 인덱스는 바로 삭제 가능
DROP INDEX idx_student_major_sname;

-- PK/UK에 의해 생성된 인덱스는 ORA-02429 발생
DROP INDEX student_sno_pk;   -- 오류

-- 제약을 비활성화(필요 시 CASCADE) 후 인덱스 제거 가능
ALTER TABLE student DISABLE CONSTRAINT student_sno_pk CASCADE;
```

**주의:** 제약 비활성화/삭제는 데이터 무결성에 영향을 줍니다. 테스트/점검 후 수행하세요.

---

## 7) 실습: 제품/판매전표/전표상세 스키마 설계 & 인덱스

### ① 테이블 생성

```
-- 1) 제품
CREATE TABLE product (
  pno        VARCHAR2(12)  CONSTRAINT pk_product PRIMARY KEY, -- PK → Unique Index 자동 생성
  pname      VARCHAR2(100) CONSTRAINT uk_product_pname UNIQUE, -- UK → Unique Index 자동 생성(선택)
  price      NUMBER        CONSTRAINT ck_product_price CHECK (price > 0)
);

-- 2) 판매전표
CREATE TABLE invoice (
  ino        VARCHAR2(12)  CONSTRAINT pk_invoice PRIMARY KEY,   -- PK
  idate      DATE          CONSTRAINT nn_invoice_idate NOT NULL,
  cname      VARCHAR2(50)  CONSTRAINT nn_invoice_cname NOT NULL,
  total_amt  NUMBER        CONSTRAINT ck_invoice_total CHECK (total_amt > 0)
);

-- 3) 전표상세
CREATE TABLE invoice_item (
  ino   VARCHAR2(12)  NOT NULL,
  pno   VARCHAR2(12)  NOT NULL,
  qty   NUMBER        CONSTRAINT ck_item_qty   CHECK (qty > 0),
  unit  NUMBER        CONSTRAINT ck_item_unit  CHECK (unit > 0),
  amt   NUMBER        CONSTRAINT ck_item_amt   CHECK (amt  > 0),
  CONSTRAINT pk_invoice_item PRIMARY KEY (ino, pno),            -- PK(복합)
  CONSTRAINT fk_item_invoice FOREIGN KEY (ino) REFERENCES invoice(ino),
  CONSTRAINT fk_item_product FOREIGN KEY (pno) REFERENCES product(pno)
);
```

### ② 필수 인덱스 추가(FK & 조회 패턴)

```
-- FK는 자동 인덱스가 생성되지 않으므로 수동 생성 권장
CREATE INDEX idx_item_ino_fk ON invoice_item (ino);
CREATE INDEX idx_item_pno_fk ON invoice_item (pno);

-- 조회 패턴 예: 제품명으로 검색이 많다면(=UK가 없다면) 인덱스 고려
CREATE INDEX idx_product_pname ON product (pname);

-- 날짜 범위 + 고객명 정렬이 잦다면 복합 인덱스
CREATE INDEX idx_invoice_idate_cname ON invoice (idate, cname);
```

복합 인덱스는 **왼쪽(선행) 컬럼**을 자주 필터링/정렬할 때 효과가 가장 큽니다.

### ③ 점검 쿼리

```
-- 인덱스/컬럼/순서/고유성
SELECT c.index_name, c.column_name, c.column_position, i.uniqueness
FROM   user_indexes i
JOIN   user_ind_columns c
  ON   c.index_name = i.index_name
WHERE  i.table_name IN ('PRODUCT','INVOICE','INVOICE_ITEM')
ORDER BY i.table_name, c.index_name, c.column_position;

-- 함수 기반 인덱스가 있다면 식 확인
SELECT index_name, column_expression
FROM   user_ind_expressions
WHERE  index_name LIKE 'IDX_%';
```

---

## 8) 성능 체크 포인트(요약)

-   **선택도(Selectivity)**: 전체의 10~15% 이하 범위면 인덱스 효과 ↑
-   **복합 인덱스 순서**: _가장 먼저 거를 컬럼_을 왼쪽에 배치
-   **함수 기반 인덱스**: `WHERE`에 동일한 표현식이 있어야 사용됨
-   **FK 인덱스**: 조인/삭제 시 경합 줄이고 효율↑ (자동 생성 X)
-   **과잉 인덱스 금지**: 쓰기 부하↑, 사용률 낮은 인덱스는 정리

---

## 자주 묻는 질문(FAQ)

**Q1. PK를 만들면 왜 인덱스가 자동 생기나요?**  
PK/UK는 _유일성 보장_이 필요합니다. 이를 보장·검증하려면 빠른 탐색 구조가 필수라 DBMS가 자동으로 Unique Index를 생성합니다.

**Q2. FK 인덱스를 꼭 만들어야 하나요?**  
권장합니다. 부모 삭제/갱신 시 자식 행 탐색, 그리고 자식→부모 조인에 결정적입니다. 미구성 시 전체 스캔·락 경합이 발생할 수 있어요.

**Q3. 복합 인덱스 컬럼 순서는 어떻게 정하죠?**  
_선행 컬럼=가장 강하게/자주 필터링되는 컬럼_이 우선입니다. 이어서 정렬·결합 패턴까지 고려해 순서를 확정하세요.

---
