## DDL 주요 명령

| 명령 | 설명 |
|------|------|
| **CREATE** | 테이블, 뷰, 시퀀스 등 객체 생성 |
| **ALTER** | 기존 객체 구조 변경 |
| **DROP** | 객체 전체 삭제 (구조까지 완전 제거) |
| **RENAME** | 객체 이름 변경 |
| **TRUNCATE** | 테이블 데이터 전체 삭제 (ROLLBACK 불가, 물리공간 반환) |

---

## 테이블 생성 규칙

- 반드시 **문자로 시작**
- 길이 **30자 이내**
- 사용 가능 문자:
  - 영문, 숫자, `_`, `$`, `#`
- **한글 사용 가능**하지만 비권장
- 동일 스키마 내 **이름 중복 금지**
- **예약어 사용 불가**
- 대소문자 구분 없음 → 내부 저장은 모두 대문자

```sql
CREATE TABLE student (
  sid   NUMBER(3),
  sname VARCHAR2(20),
  dept  VARCHAR2(15)
);
```

---

## Oracle 데이터 타입

| 타입 | 설명 |
|------|------|
| **VARCHAR2(n)** | 가변 길이 문자형 (1~4000 byte) |
| **CHAR(n)** | 고정 길이 문자형 (1~2000 byte) |
| **NUMBER(n,p)** | 숫자형 (n: 전체 자리수 / p: 소수 자리수) |
| **DATE** | 날짜/시간 (`YYYY/MM/DD HH24:MI:SS`) |
| **LONG** | 최대 2GB 문자 (테이블당 1개) |
| **CLOB** | 최대 4GB 문자 |
| **BLOB** | 최대 4GB 바이너리 |
| **BFILE** | 4GB 외부 파일 참조 |
| **ROWID** | 행의 고유 물리 주소 |

---

## 문자형 데이터 처리 기준

- 주민번호, 학번, 군번  
  → 숫자로 보이지만 **연산 불필요 → VARCHAR2**
- 금액·계량 데이터  
  → 연산 필요 → **NUMBER**

**VARCHAR = VARCHAR2 (사실상 동일)**  
Oracle 표준 권장은 **VARCHAR2**

### CHAR vs VARCHAR2

| 비교 | CHAR | VARCHAR2 |
|------|------|-----------|
| 길이 | 고정 | 가변 |
| 저장 | 공백 패딩 발생 | 입력 길이만 저장 |
| 효율 | 낮음 | 높음 |
| 권장 | ❌ | ✅

---

## CHAR vs VARCHAR2 실습

```sql
CREATE TABLE comp(
  co1 CHAR(4),
  co2 VARCHAR2(4)
);

INSERT INTO comp VALUES ('AA','AA');

SELECT LENGTHB(co1), LENGTHB(co2) FROM comp;

SELECT * FROM comp WHERE co1='AA';
SELECT * FROM comp WHERE co2='AA';

SELECT * FROM comp WHERE co1=co2;  -- FALSE
```

➡ **CHAR**는 공백 패딩 → 비교 불일치 발생  
➡ 실무에서는 **VARCHAR2 권장**

---

## DATE 타입 입력 & 조회

```sql
CREATE TABLE hd (
  no NUMBER,
  hdate DATE
);

INSERT INTO hd VALUES (1, SYSDATE);
```

### 날짜 조회 시 주의

❌ 문자열 비교 (비권장)
```sql
SELECT * FROM hd WHERE hdate = '2021/10/02';
```

✅ **TO_DATE 사용 권장**
```sql
SELECT * 
FROM hd 
WHERE hdate = TO_DATE('2021/10/02','YYYY/MM/DD');
```

✅ 시간 제거 조회
```sql
SELECT * 
FROM hd 
WHERE TRUNC(hdate) = TO_DATE('2021/10/02','YYYY/MM/DD');
```

✅ 날짜 출력 포맷

```sql
SELECT no, TO_CHAR(hdate, 'YYYY/MM/DD HH24:MI:SS')
FROM hd;
```

✅ 범위 검색

```sql
SELECT *
FROM hd
WHERE hdate BETWEEN 
      TO_DATE('2021/10/02','YYYY/MM/DD')
  AND TO_DATE('2021/10/03','YYYY/MM/DD');
```

---

## 테이블 삭제 – DROP

테이블 구조까지 완전히 삭제  
단, Oracle은 휴지통(**RECYCLEBIN**)에 남아 복구 가능

```sql
SELECT * FROM tab;

DROP TABLE board;
DROP TABLE comp;
DROP TABLE hd;

SELECT * FROM tab;

PURGE RECYCLEBIN;
```

---

## FLASHBACK – 테이블 복구

```sql
SELECT object_name, original_name
FROM recyclebin;

FLASHBACK TABLE original_name TO BEFORE DROP;
-- 또는
FLASHBACK TABLE original_name 
TO BEFORE DROP 
RENAME TO new_name;
```

**FLASHBACK은 `ROLLBACK`이 아닌 별도 복구 명령**

---

## 실습 예제 – 테이블 복구

```sql
DROP TABLE test;

SELECT * FROM tab;

SELECT object_name, original_name
FROM recyclebin;

FLASHBACK TABLE test TO BEFORE DROP;

SELECT * FROM tab;
```

---

## 정리

- **DDL은 DB 구조 관리 전담 언어**
- 주요 구문: `CREATE`, `ALTER`, `DROP`, `TRUNCATE`
- 문자 비교 → `VARCHAR2` 권장
- 날짜 조회 → **반드시 `TO_DATE` 또는 `TRUNC` 사용**
- DROP 후 테이블은 **RECYCLEBIN** → `FLASHBACK`으로 복구 가능
