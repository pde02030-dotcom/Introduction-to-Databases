
# 🚀 SQL CHECK 제약조건

---

# 1️⃣ CHECK 제약조건이란?

CHECK 제약조건은 **해당 컬럼(또는 행)이 특정 조건식을 반드시 만족해야 하는 제약조건**이다.

```sql
CHECK (age > 0)
CHECK (score BETWEEN 0 AND 100)
CHECK (status IN ('Y','N'))
```

조건을 만족하지 못하면 INSERT/UPDATE 자체가 거부된다.

---

# 2️⃣ CHECK는 언제 사용되는가?

✔ 데이터 입력 실수를 방지
✔ 비즈니스 규칙을 DB 레벨에서 강제
✔ 애플리케이션 레이어 오류 감소
✔ 제약조건 기반의 데이터 검증 자동화
✔ ENUM 대체 수단(MySQL, PostgreSQL 등)

---

# 3️⃣ CHECK 동작 원리

1. 사용자가 INSERT / UPDATE 실행
2. CHECK 조건 검증
3. 조건 위반 시 오류 메시지 발생
4. 트랜잭션은 ROLLBACK 됨

---

# 4️⃣ MySQL의 CHECK 주의점 (아주 중요)

MySQL 8.0부터는 CHECK가 완전히 지원됨!
8.0 이전에는 문법은 가능하지만 **실제로는 작동하지 않는 제약조건이었음.**

---

# 5️⃣ CHECK와 다른 제약조건 비교

| 제약조건            | 역할           |
| --------------- | ------------ |
| **NOT NULL**    | NULL 허용 여부   |
| **UNIQUE**      | 중복 허용 여부     |
| **FOREIGN KEY** | 참조 무결성       |
| **CHECK**       | 특정 논리적 조건 강제 |

CHECK는 이 네 가지 제약조건 중 가장 **유연하고 표현 범위가 넓다.**

---

# 6️⃣ 실습용 테이블 생성

CHECK 연습을 위해 학생 점수·나이·상태를 가진 테이블을 만든다.

```sql
CREATE TABLE Student (
    id      INT NOT NULL AUTO_INCREMENT,
    name    VARCHAR(50),
    age     INT CHECK (age >= 0 AND age <= 120),
    score   INT CHECK (score BETWEEN 0 AND 100),
    status  VARCHAR(10) CHECK (status IN ('ACTIVE','INACTIVE')),
    PRIMARY KEY (id)
);
```

---

# 7️⃣ 샘플 데이터 삽입

```sql
INSERT INTO Student (name, age, score, status) VALUES
('김철수', 25, 85, 'ACTIVE'),
('이영희', 31, 92, 'ACTIVE'),
('박민수', 19, 58, 'INACTIVE');
```

---

# 8️⃣ CHECK 실습 예제

---

## ✔ 예제 1) 정상 입력되는 케이스

```sql
INSERT INTO Student (name, age, score, status)
VALUES ('최가영', 30, 100, 'ACTIVE');
```

→ ✔ 성공

---

## ✔ 예제 2) 나이가 음수일 경우 (CHECK 오류)

```sql
INSERT INTO Student (name, age, score, status)
VALUES ('홍길동', -1, 50, 'ACTIVE');
```

### 결과

❌ ERROR 3819: Check constraint 'age check' is violated

---

## ✔ 예제 3) 점수가 0~100 범위를 벗어났을 경우

```sql
INSERT INTO Student (name, age, score, status)
VALUES ('테스트', 20, 120, 'ACTIVE');
```

→ 오류: Check constraint violated

---

## ✔ 예제 4) status 값이 허용 목록에 없을 때

```sql
INSERT INTO Student (name, age, score, status)
VALUES ('AAA', 22, 50, 'UNKNOWN');
```

→ ❌ 오류

---

## ✔ 예제 5) CHECK 여러 조건 동시 만족

```sql
INSERT INTO Student (name, age, score, status)
VALUES ('정우성', 45, 75, 'INACTIVE');
```

→ ✔ 성공

---

## ✔ 예제 6) CHECK 조건을 가진 컬럼 UPDATE 테스트

```sql
UPDATE Student
SET score = 110
WHERE name = '김철수';
```

→ ❌ 오류

---

## ✔ 예제 7) 점수 정상 UPDATE

```sql
UPDATE Student
SET score = 90
WHERE name = '박민수';
```

→ ✔ 성공

---

## ✔ 예제 8) CHECK 제약조건 ALTER로 추가

```sql
ALTER TABLE Student
ADD CONSTRAINT chk_age CHECK (age >= 0 AND age <= 150);
```

---

## ✔ 예제 9) CHECK 제약조건 DROP

```sql
ALTER TABLE Student
DROP CHECK chk_age;
```

---

## ✔ 예제 10) CHECK + LIKE 패턴 검사(정규식 필요한 경우)

```sql
ALTER TABLE Student
ADD CONSTRAINT chk_name CHECK (name LIKE '%이%');
```

이후 아래 입력:

```sql
INSERT INTO Student (name, age, score, status)
VALUES ('박철수', 20, 80, 'ACTIVE');
```

→ ❌ 오류 (이 포함되지 않음)

---

## ✔ 예제 11) CHECK + 날짜 조건

```sql
ALTER TABLE Student
ADD CONSTRAINT chk_age_real CHECK (age >= 0 AND age <= 200);
```

→ 날짜가 아니므로 주의, 나이는 정수 조건

---

## ✔ 예제 12) CHECK + 복합 조건 (다중 컬럼)

```sql
ALTER TABLE Student
ADD CONSTRAINT chk_status_logic
CHECK (
    (status = 'ACTIVE' AND score >= 60)
    OR
    (status = 'INACTIVE')
);
```

### 의미

ACTIVE 학생은 점수가 60 이상이어야 함.

```sql
UPDATE Student SET score = 30 WHERE name='김철수';
```

→ ❌ 오류

---

## ✔ 예제 13) CHECK + BETWEEN (이미 내부적으로 사용 가능)

```sql
ALTER TABLE Student
ADD CONSTRAINT chk_score_range CHECK (score BETWEEN 0 AND 100);
```

---

## ✔ 예제 14) CHECK 제약조건 조회(INFORMATION_SCHEMA)

```sql
SELECT CONSTRAINT_NAME, CHECK_CLAUSE
FROM INFORMATION_SCHEMA.CHECK_CONSTRAINTS
WHERE CONSTRAINT_SCHEMA = DATABASE();
```

---

## ✔ 예제 15) CHECK + INSERT ALL OR NOTHING (트랜잭션 연계)

```sql
START TRANSACTION;

INSERT INTO Student (name, age, score, status)
VALUES ('AAA', 20, 90, 'ACTIVE');

INSERT INTO Student (name, age, score, status)
VALUES ('BBB', 20, 900, 'ACTIVE');   -- CHECK 위반

ROLLBACK;
```

→ ✔ 모든 INSERT 취소됨 (원자성 보장)

---

# 9️⃣ CHECK 제약조건 실무 전문가 팁

---

### ✔ 1) CHECK는 비즈니스 규칙을 DB 레벨에서 강제하는 훌륭한 장치

애플리케이션 오류보다 DB 오류가 더 신뢰할 수 있음.

---

### ✔ 2) ENUM보다 CHECK가 더 유연 (PostgreSQL·Oracle 등에서 표준 지원)

MySQL 8부터는 완전 지원.

---

### ✔ 3) CHECK 조건 변경은 ALTER TABLE → DDL → Auto Commit

운영 환경에서는 매우 주의해야 함.

---

### ✔ 4) 복합 CHECK는 너무 복잡해지면 앱 로직으로 분리하는 것이 낫다

---

### ✔ 5) CHECK 오류 메시지를 사용자에게 직접 전달하면 UX가 좋아짐

예: "점수는 0~100 사이여야 합니다."

---

### ✔ 6) CHECK는 컬럼 단위뿐 아니라 **ROW 단위 조건**도 가능

(status='ACTIVE' → score>=60 요구하는 예처럼)

---

### ✔ 7) CHECK는 인덱스를 만들지 않는다

단순히 조건만 강제할 뿐이다.

---

