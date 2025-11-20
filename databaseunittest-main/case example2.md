# 🎯 SQL CASE WHEN

**– 실제 테이블 생성 + 데이터 삽입 + 실무 CASE WHEN 예제 –**

이 글에서는 단순한 코드 조각이 아니라
**실제로 테스트 가능한 테이블 + 데이터 + 결과 예시**까지 포함하여
CASE WHEN을 완전히 이해할 수 있도록 구성했습니다.

---

# 1️⃣ 테스트용 테이블 만들기 (CREATE TABLE)

예제로 사용할 테이블은 `member` 입니다.
회원 나이, 성별, 구매금액 등이 포함되어 있어
CASE WHEN 활용하기에 딱 좋은 데이터 구조입니다.

```sql
CREATE TABLE member (
    member_id     INT AUTO_INCREMENT PRIMARY KEY,
    name          VARCHAR(50) NOT NULL,
    gender        CHAR(1),        -- M / F / NULL
    age           INT,
    total_spent   INT,            -- 총 구매 금액
    phone         VARCHAR(20),    -- NULL 또는 공백 가능
    reg_date      DATE
);
```

---

# 2️⃣ 테스트용 데이터 INSERT

```sql
INSERT INTO member (name, gender, age, total_spent, phone, reg_date) VALUES
('김철수', 'M', 25, 120000, '01012345678', '2024-11-01'),
('이영희', 'F', 32, 45000,  NULL,          '2024-11-05'),
('박민수', 'M', 17, 8000,   '',            '2024-10-28'),
('최지은', 'F', 41, 380000, '01055558888', '2024-11-10'),
('홍길동', NULL, 29, 0,      NULL,          '2024-11-08');
```

이제 실제로 데이터를 기반으로 CASE WHEN이 어떻게 동작하는지 살펴봅니다.

---

# 3️⃣ CASE WHEN 실전 예제

---

# 📌 예제 1 — 점수 대신 “구매금액 기준 등급 나누기”

### SQL

```sql
SELECT 
    name,
    total_spent,
    CASE
        WHEN total_spent >= 300000 THEN 'VIP'
        WHEN total_spent >= 100000 THEN 'GOLD'
        WHEN total_spent >= 50000  THEN 'SILVER'
        ELSE 'BRONZE'
    END AS grade
FROM member;
```

### 결과

| name | total_spent | grade  |
| ---- | ----------- | ------ |
| 김철수  | 120000      | GOLD   |
| 이영희  | 45000       | BRONZE |
| 박민수  | 8000        | BRONZE |
| 최지은  | 380000      | VIP    |
| 홍길동  | 0           | BRONZE |

---

# 📌 예제 2 — NULL/빈문자열 전화번호 처리하기

### SQL

```sql
SELECT
    name,
    phone,
    CASE
        WHEN phone IS NULL OR phone = '' THEN '미입력'
        ELSE '등록됨'
    END AS phone_status
FROM member;
```

### 결과

| name | phone       | phone_status |
| ---- | ----------- | ------------ |
| 김철수  | 01012345678 | 등록됨          |
| 이영희  | NULL        | 미입력          |
| 박민수  | ''          | 미입력          |
| 최지은  | 01055558888 | 등록됨          |
| 홍길동  | NULL        | 미입력          |

---

# 📌 예제 3 — 나이대 그룹 생성 (GROUP BY와 함께)

### SQL

```sql
SELECT
    CASE
        WHEN age < 20 THEN '10대 이하'
        WHEN age < 30 THEN '20대'
        WHEN age < 40 THEN '30대'
        ELSE '40대 이상'
    END AS age_group,
    COUNT(*) AS cnt
FROM member
GROUP BY
    CASE
        WHEN age < 20 THEN '10대 이하'
        WHEN age < 30 THEN '20대'
        WHEN age < 40 THEN '30대'
        ELSE '40대 이상'
    END;
```

### 결과

| age_group | cnt |
| --------- | --- |
| 10대 이하    | 1   |
| 20대       | 2   |
| 30대       | 1   |
| 40대 이상    | 1   |

---

# 📌 예제 4 — 조건별 카운팅(보고서에서 매우 자주 사용하는 패턴)

### SQL

```sql
SELECT
    COUNT(*) AS total,
    SUM(CASE WHEN gender = 'M' THEN 1 ELSE 0 END) AS male_count,
    SUM(CASE WHEN gender = 'F' THEN 1 ELSE 0 END) AS female_count,
    SUM(CASE WHEN gender IS NULL THEN 1 ELSE 0 END) AS unknown_gender
FROM member;
```

### 결과

| total | male_count | female_count | unknown_gender |
| ----- | ---------- | ------------ | -------------- |
| 5     | 2          | 2            | 1              |

---

# 📌 예제 5 — 이메일 도메인 분류 (단순 CASE)

테이블에 email 컬럼이 없으므로 추가 insert만 하겠습니다.

### 추가 UPDATE

```sql
UPDATE member SET email = 'chulsoo@naver.com'  WHERE name = '김철수';
UPDATE member SET email = 'younghee@gmail.com' WHERE name = '이영희';
UPDATE member SET email = 'minsu@yahoo.com'    WHERE name = '박민수';
UPDATE member SET email = 'jieun@naver.com'    WHERE name = '최지은';
UPDATE member SET email = 'hong@test.com'      WHERE name = '홍길동';
```

(테이블에 email 컬럼이 없다면 아래처럼 먼저 추가)

```sql
ALTER TABLE member ADD COLUMN email VARCHAR(100);
```

### SQL

```sql
SELECT
    name,
    email,
    CASE
        WHEN email LIKE '%naver%' THEN 'NAVER'
        WHEN email LIKE '%gmail%' THEN 'GOOGLE'
        WHEN email LIKE '%yahoo%' THEN 'YAHOO'
        ELSE 'OTHER'
    END AS email_provider
FROM member;
```

### 결과

| name | email                                           | email_provider |
| ---- | ----------------------------------------------- | -------------- |
| 김철수  | [chulsoo@naver.com](mailto:chulsoo@naver.com)   | NAVER          |
| 이영희  | [younghee@gmail.com](mailto:younghee@gmail.com) | GOOGLE         |
| 박민수  | [minsu@yahoo.com](mailto:minsu@yahoo.com)       | YAHOO          |
| 최지은  | [jieun@naver.com](mailto:jieun@naver.com)       | NAVER          |
| 홍길동  | [hong@test.com](mailto:hong@test.com)           | OTHER          |

---

# 📌 예제 6 — CASE WHEN + ORDER BY (커스텀 정렬)

VIP/GOLD/SILVER/BRONZE 순으로 정렬하고 싶을 때
일반 ORDER BY로는 불가능합니다.
이럴 때 CASE WHEN이 정렬 기준이 됩니다.

### SQL

```sql
SELECT 
    name,
    total_spent
FROM member
ORDER BY
    CASE
        WHEN total_spent >= 300000 THEN 1
        WHEN total_spent >= 100000 THEN 2
        WHEN total_spent >= 50000  THEN 3
        ELSE 4
    END;
```

### 결과 (높은 등급 먼저)

| name | total_spent |
| ---- | ----------- |
| 최지은  | 380000      |
| 김철수  | 120000      |
| 이영희  | 45000       |
| 박민수  | 8000        |
| 홍길동  | 0           |

---

# 📌 예제 7 — WHERE에서 CASE WHEN? (가능하지만 비권장)

WHERE 구문에서 CASE도 사용은 됩니다.

```sql
SELECT * FROM member
WHERE CASE 
          WHEN gender = 'M' THEN 1
          ELSE 0
      END = 1;
```

하지만 가독성·성능 측면에서 아래처럼 쓰는 것이 훨씬 낫습니다.

```sql
SELECT * FROM member
WHERE gender = 'M';
```

---

# 📌 예제 8 — 날짜 기반 분기

### SQL

```sql
SELECT
    name,
    reg_date,
    CASE
        WHEN reg_date < '2024-11-01' THEN '이전 가입자'
        WHEN reg_date BETWEEN '2024-11-01' AND '2024-11-07' THEN '초기 가입자'
        ELSE '신규 가입자'
    END AS reg_type
FROM member;
```

---

# 4️⃣ CASE WHEN 핵심 요약

| 포인트                                         | 설명                       |
| ------------------------------------------- | ------------------------ |
| WHEN 조건은 위에서부터 평가                           | 첫 TRUE만 실행               |
| SELECT / ORDER BY / GROUP BY / HAVING 모두 가능 | WHERE에서는 권장하지 않음         |
| 집계함수와 조합하면 강력                               | 조건별 COUNT/SUM 가능         |
| NULL 처리에 최적                                 | phone IS NULL or '' 등    |
| 커스텀 정렬 가능                                   | VIP > GOLD > SILVER 순서 등 |


