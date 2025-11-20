# 🚀 SQL ENUM

---

# 1️⃣ ENUM이란?

**ENUM은 컬럼 값이 지정된 목록 중 하나로 고정되는 데이터 타입이다.**

예:

```sql
status ENUM('PENDING','PAID','CANCEL')
```

위 컬럼은 반드시
PENDING / PAID / CANCEL 중 하나를 가져야 하며
다른 값은 입력되지 않는다.

---

# 2️⃣ ENUM의 내부 동작 방식 (중요)

MySQL에서 ENUM은 실제로 **문자열이 아니라 숫자 인덱스**를 저장한다.

예:

```sql
ENUM('A','B','C')
```

| 실 저장값 | 의미  |
| ----- | --- |
| 1     | 'A' |
| 2     | 'B' |
| 3     | 'C' |

즉, ENUM 컬럼은 “숫자 1~N”을 저장하고, 조회 시 문자열로 보일 뿐이다.

✔ 장점: 저장 공간 적음
✔ 장점: 비교 속도 빠름
✔ 주의: **순서(index)가 중요**함

---

# 3️⃣ ENUM의 장점

### ✔ 1) 저장 공간 절약 (TINYINT 수준)

문자열(예: 'PENDING') 전체를 저장하는 대신
**숫자 인덱스(1~N)만 저장**함.

---

### ✔ 2) 데이터 일관성 보장

허용된 값 외의 값은 들어갈 수 없음.

---

### ✔ 3) 비교(Operation) 성능 빠름

정렬·검색이 숫자 기반으로 처리되므로 속도가 매우 빠름.

---

### ✔ 4) 코드 가독성 증가

문자열로 의미 전달 → SQL 이해도 상승.

---

# 4️⃣ ENUM의 단점 (실무에서 아주 중요)

### ❌ 1) ENUM 값 추가·수정이 DDL 작업

ENUM 목록에 새로운 값을 넣는 행위는
ALTER TABLE 이 필요 → 비용 큼.

---

### ❌ 2) 순서(index)가 바뀌면 의미가 바뀔 수 있음

예:

```sql
ENUM('PENDING','PAID','CANCEL')
```

여기에 'NEW'를 앞에 추가하면:

```sql
ENUM('NEW','PENDING','PAID','CANCEL')
```

PENDING(기존 1)이 2가 되어버리고
정렬·검색 결과가 달라질 수 있다.

---

### ❌ 3) ENUM은 MySQL 전용

다른 RDBMS(Oracle, PostgreSQL, SQL Server)에서는
ENUM 타입이 없거나 거의 사용하지 않음.

---

# 5️⃣ ENUM vs CHECK 비교

| 기능       | ENUM     | CHECK  |
| -------- | -------- | ------ |
| 값 제한     | ✔ 정확     | ✔ 가능   |
| 내부 저장    | 숫자 인덱스   | 실제 문자열 |
| 변경 시 DDL | 필요       | 불필요    |
| DBMS 지원  | MySQL 전용 | 대부분 지원 |
| 성능       | 더 빠름     | 일반     |

실무에서는 ENUM 대신 CHECK + VARCHAR를 쓰는 기업도 많다.

---

# 6️⃣ 실습용 테이블 생성

ENUM을 활용한 주문(Order) 테이블을 만든다.

```sql
CREATE TABLE Orders (
    id        INT NOT NULL AUTO_INCREMENT,
    customer  VARCHAR(50),
    amount    INT,
    status    ENUM('PENDING','PAID','CANCEL','DONE'),
    PRIMARY KEY (id)
);
```

---

# 7️⃣ 샘플 데이터 삽입

```sql
INSERT INTO Orders (customer, amount, status) VALUES
('김철수', 50000, 'PENDING'),
('이영희', 120000, 'PAID'),
('박민수', 45000, 'CANCEL'),
('최가영', 70000, 'DONE'),
('홍길동', 30000, 'PENDING');
```

---

# 8️⃣ ENUM 실습 예제

---

## ✔ 예제 1) 전체 주문 조회

```sql
SELECT id, customer, amount, status
FROM Orders;
```

### 결과

| id | customer | amount | status  |
| -- | -------- | ------ | ------- |
| 1  | 김철수      | 50000  | PENDING |
| 2  | 이영희      | 120000 | PAID    |
| 3  | 박민수      | 45000  | CANCEL  |
| 4  | 최가영      | 70000  | DONE    |
| 5  | 홍길동      | 30000  | PENDING |

---

## ✔ 예제 2) 특정 ENUM 값 조회

```sql
SELECT *
FROM Orders
WHERE status = 'PENDING';
```

### 결과

| id | customer | amount | status  |
| -- | -------- | ------ | ------- |
| 1  | 김철수      | 50000  | PENDING |
| 5  | 홍길동      | 30000  | PENDING |

---

## ✔ 예제 3) ENUM 값 순서에 따른 자동 정렬 (숫자 기반)

```sql
SELECT customer, status
FROM Orders
ORDER BY status;
```

ENUM 순서: PENDING(1) → PAID(2) → CANCEL(3) → DONE(4)

### 결과

| customer | status  |
| -------- | ------- |
| 김철수      | PENDING |
| 홍길동      | PENDING |
| 이영희      | PAID    |
| 박민수      | CANCEL  |
| 최가영      | DONE    |

---

## ✔ 예제 4) 금액 기준 통계 (GROUP BY + ENUM)

```sql
SELECT status, COUNT(*) AS cnt, SUM(amount) AS total_amt
FROM Orders
GROUP BY status;
```

---

## ✔ 예제 5) ENUM으로 CASE 처리

```sql
SELECT
    customer,
    amount,
    status,
    CASE status
        WHEN 'PENDING' THEN '결제 대기'
        WHEN 'PAID' THEN '결제 완료'
        WHEN 'CANCEL' THEN '취소됨'
        WHEN 'DONE' THEN '배송 완료'
    END AS status_kor
FROM Orders;
```

---

## ✔ 예제 6) PENDING 상태 주문 금액 합계

```sql
SELECT SUM(amount) AS pending_total
FROM Orders
WHERE status = 'PENDING';
```

### 결과

| pending_total |
| ------------- |
| 80000         |

---

## ✔ 예제 7) 취소된 주문만 조회

```sql
SELECT *
FROM Orders
WHERE status = 'CANCEL';
```

---

## ✔ 예제 8) 상태별 최대 금액

```sql
SELECT status, MAX(amount) AS max_amt
FROM Orders
GROUP BY status;
```

---

## ✔ 예제 9) DONE 상태가 아닌 주문

```sql
SELECT *
FROM Orders
WHERE status <> 'DONE';
```

---

## ✔ 예제 10) ENUM 값 갯수 확인 (INFORMATION_SCHEMA)

```sql
SELECT COLUMN_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME='Orders' AND COLUMN_NAME='status';
```

### 결과

```
enum('PENDING','PAID','CANCEL','DONE')
```

---

## ✔ 예제 11) ENUM 값 추가 (DDL)

```sql
ALTER TABLE Orders
MODIFY status ENUM('NEW','PENDING','PAID','CANCEL','DONE');
```

---

## ✔ 예제 12) ENUM의 실제 저장값 확인 (숫자 인덱스)

```sql
SELECT id, status, status + 0 AS enum_index
FROM Orders;
```

### 예시 결과

| status  | enum_index |
| ------- | ---------- |
| PENDING | 2          |
| PAID    | 3          |
| …       | …          |

(ENUM 목록 순서에 따라 달라짐)

---

## ✔ 예제 13) 정렬이 문자열이 아닌 숫자 기준인지 확인

```sql
SELECT status
FROM Orders
ORDER BY status;
```

---

## ✔ 예제 14) ENUM 값 범위 검사

```sql
SELECT
    id,
    status,
    CASE
        WHEN status NOT IN ('PENDING','PAID','CANCEL','DONE')
             THEN 'ERROR'
        ELSE 'OK'
    END AS validate
FROM Orders;
```

---

## ✔ 예제 15) ENUM → VARCHAR 변환

```sql
SELECT CAST(status AS CHAR) AS status_str
FROM Orders;
```

---

# 9️⃣ ENUM 사용 시 실무 전문가 팁

---

### ✔ 1) 대규모 시스템에서는 ENUM 남용 금지

변경 시 ALTER TABLE 필요 → 서비스 중단 위험.

---

### ✔ 2) ENUM 순서(index)가 정렬·비교에 써진다는 걸 반드시 기억하라

ENUM('A', 'B', 'C') → 'A'=1, 'B'=2, 'C'=3

---

### ✔ 3) ENUM 값 추가 시 반드시 뒤에 추가하라

중간 삽입은 기존 데이터의 의미까지 변질됨.

---

### ✔ 4) ENUM은 MySQL 의존형 → PostgreSQL·Oracle·MSSQL 호환 문제

→ CHECK + VARCHAR 대체 전략 추천

---

### ✔ 5) API / 백엔드에서는 ENUM 값을 상수(Constant)로 관리

Java, Spring, Node, Python 등에서 ENUM 타입과 연결해 사용

