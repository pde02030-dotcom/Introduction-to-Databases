# 🚀 SQL ALTER TABLE

---

# 1️⃣ ALTER TABLE 이란?

**ALTER TABLE은 기존 테이블의 구조(스키마)를 변경하는 SQL DDL(Data Definition Language) 명령어이다.**

즉, 테이블 생성 후 스키마를 수정할 때 반드시 사용한다.

가능한 작업:

✔ 컬럼 추가
✔ 컬럼 삭제
✔ 컬럼 이름 변경
✔ 자료형 변경
✔ 제약조건 추가 / 삭제
✔ 기본값 변경
✔ 인덱스 추가 / 삭제
✔ ENUM 값 추가(MySQL)

---

# 2️⃣ ALTER TABLE의 중요한 특징

### ✔ 1) DDL이므로 **Auto-Commit**

ROLLBACK이 되지 않는다 → 매우 위험!

### ✔ 2) 테이블에 Lock 발생

테이블 구조 변경 동안 INSERT/UPDATE/DELETE가 진행 불가할 수 있음.

### ✔ 3) 데이터 손실 위험

자료형 변경 시 값이 잘리는 경우도 있음.

### ✔ 4) 운영 환경에서는 변경 계획 필수

대용량 테이블의 ALTER는 매우 오랜 시간이 걸릴 수 있음.

---

# 3️⃣ 실습용 테이블 생성

ALTER 실습을 위해 기본 테이블을 만든다.

```sql
CREATE TABLE Member (
    id        INT NOT NULL AUTO_INCREMENT,
    name      VARCHAR(50),
    email     VARCHAR(100),
    reg_date  DATE,
    PRIMARY KEY (id)
);
```

---

# 4️⃣ 샘플 데이터 삽입

```sql
INSERT INTO Member (name, email, reg_date) VALUES
('김철수', 'kim@example.com', '2024-01-01'),
('이영희', 'lee@example.com', '2024-01-05'),
('박민수', 'minsu@example.com', '2024-01-10');
```

---

# 5️⃣ ALTER TABLE 실습 예제

---

## ✔ 예제 1) 컬럼 추가 (ADD COLUMN)

```sql
ALTER TABLE Member
ADD COLUMN phone VARCHAR(20);
```

### 결과 (DESCRIBE Member)

| Field | Type        |
| ----- | ----------- |
| phone | varchar(20) |

---

## ✔ 예제 2) 기본값이 있는 컬럼 추가

```sql
ALTER TABLE Member
ADD COLUMN status VARCHAR(20) DEFAULT 'ACTIVE';
```

### 결과

모든 기존 row의 status = ACTIVE 자동 입력.

---

## ✔ 예제 3) 컬럼 삭제 (DROP COLUMN)

```sql
ALTER TABLE Member
DROP COLUMN phone;
```

---

## ✔ 예제 4) 컬럼 이름 변경 (CHANGE)

```sql
ALTER TABLE Member
CHANGE COLUMN email email_address VARCHAR(150);
```

---

## ✔ 예제 5) 자료형 변경 (MODIFY)

VARCHAR → TEXT로 변경

```sql
ALTER TABLE Member
MODIFY COLUMN name VARCHAR(100);
```

---

## ✔ 예제 6) NOT NULL 추가

```sql
ALTER TABLE Member
MODIFY COLUMN name VARCHAR(100) NOT NULL;
```

---

## ✔ 예제 7) 컬럼 순서 변경 (MySQL Only)

```sql
ALTER TABLE Member
ADD COLUMN age INT AFTER name;
```

---

## ✔ 예제 8) 테이블 이름 변경

```sql
ALTER TABLE Member
RENAME TO MemberInfo;
```

---

## ✔ 예제 9) Primary Key 변경

### 1단계: 기존 PK 삭제

```sql
ALTER TABLE MemberInfo
DROP PRIMARY KEY;
```

### 2단계: 새 PK 추가

```sql
ALTER TABLE MemberInfo
ADD PRIMARY KEY(id, email_address);
```

---

## ✔ 예제 10) UNIQUE 제약조건 추가

```sql
ALTER TABLE MemberInfo
ADD CONSTRAINT u_email UNIQUE(email_address);
```

---

## ✔ 예제 11) UNIQUE 제약조건 삭제

```sql
ALTER TABLE MemberInfo
DROP INDEX u_email;
```

---

## ✔ 예제 12) CHECK 제약조건 추가 (MySQL 8+)

```sql
ALTER TABLE MemberInfo
ADD CONSTRAINT chk_age CHECK (age >= 0);
```

---

## ✔ 예제 13) CHECK 삭제

```sql
ALTER TABLE MemberInfo
DROP CHECK chk_age;
```

---

## ✔ 예제 14) ENUM 값 추가(DDL 필요)

```sql
ALTER TABLE MemberInfo
MODIFY status ENUM('ACTIVE','INACTIVE','BLOCKED');
```

---

## ✔ 예제 15) 인덱스 추가 + 삭제

### 인덱스 추가

```sql
ALTER TABLE MemberInfo
ADD INDEX idx_email (email_address);
```

### 인덱스 삭제

```sql
ALTER TABLE MemberInfo
DROP INDEX idx_email;
```

---

# 6️⃣ ALTER TABLE 작업 후 실제 데이터 조회 예시

```sql
SELECT * FROM MemberInfo;
```

### 예시 결과

| id | name | email_address                                 | reg_date   | status | age |
| -- | ---- | --------------------------------------------- | ---------- | ------ | --- |
| 1  | 김철수  | [kim@example.com](mailto:kim@example.com)     | 2024-01-01 | ACTIVE | 25  |
| 2  | 이영희  | [lee@example.com](mailto:lee@example.com)     | 2024-01-05 | ACTIVE | 32  |
| 3  | 박민수  | [minsu@example.com](mailto:minsu@example.com) | 2024-01-10 | ACTIVE | 29  |

(실습 예제들의 영향으로 스키마가 풍부해짐)

---

# 7️⃣ ALTER TABLE 전문가 실무 팁

---

### ✔ 1) ALTER TABLE 은 잠금(Lock) 발생 → 운영에서 위험

대규모 테이블에서는 서비스 중단 가능.

---

### ✔ 2) ALTER는 Auto-Commit → ROLLBACK 불가능

DDL은 원자성이 적용되지 않음.

---

### ✔ 3) ALTER + MODIFY 자료형 변경 시 데이터 손실 가능

예: VARCHAR(200)→VARCHAR(50) → 길이 초과 부분 잘림.

---

### ✔ 4) ENUM / CHECK 수정은 무조건 DDL

운영 중 ENUM 변경은 주의 필요.

---

### ✔ 5) ALTER TABLE RENAME 은 초경량

실제 데이터 이동 없음 → 메타데이터 변경만 수행.

---

### ✔ 6) 제약조건 이름을 명시적으로 주는 게 좋음

DROP 시 이름 필요하기 때문.

---

### ✔ 7) 인덱스 관리도 ALTER TABLE로 처리 가능

ADD INDEX / DROP INDEX

---


