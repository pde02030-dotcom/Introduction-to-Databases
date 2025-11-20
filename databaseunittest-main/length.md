# 🚀 SQL LENGTH() 

---

# 1️⃣ LENGTH 함수란?

### ✔ LENGTH(string)

문자열의 **바이트(Byte)** 길이를 반환한다.

예: ASCII 문자열은 1글자 = 1바이트
예: UTF-8 기반 한글은 1글자 = 3바이트

```
LENGTH('ABC') → 3
LENGTH('가') → 3   (UTF-8 기준)
```

---

# 2️⃣ LENGTH와 CHAR_LENGTH / CHARACTER_LENGTH 차이

| 함수                                          | 의미            | 단위        |
| ------------------------------------------- | ------------- | --------- |
| **LENGTH()**                                | 문자열의 “바이트 수”  | Byte      |
| **CHAR_LENGTH()** 또는 **CHARACTER_LENGTH()** | 문자열의 “문자의 개수” | Character |

예:

```
LENGTH('가나다') → 9 (3글자 × 3바이트)
CHAR_LENGTH('가나다') → 3 (3글자)
```

이 차이를 모르면,
✔ 입력 데이터 길이 검증
✔ 이메일 길이 제약
✔ UTF8 문자열 처리
✔ 데이터 마스킹
에서 큰 문제가 발생합니다.

---

# 3️⃣ DBMS별 LENGTH 동작 차이

| DBMS           | LENGTH 의미                      | 주의                    |
| -------------- | ------------------------------ | --------------------- |
| **MySQL**      | 바이트 길이(UTF8 → 한글 3바이트)         | CHAR_LENGTH 따로 존재     |
| **Oracle**     | LENGTH = 문자 수 (CHAR semantics) | LENGTHB 사용 시 바이트 길이   |
| **PostgreSQL** | LENGTH = 문자 수                  | OCTET_LENGTH 사용 시 바이트 |
| **SQL Server** | LEN() = 문자 수                   | DATALENGTH() = 바이트 길이 |

---

## 📌 MySQL/Oracle 차이 예시

문제 문자열: `'가'`

| DBMS                 | 결과 |
| -------------------- | -- |
| MySQL: LENGTH('가')   | 3  |
| Oracle: LENGTH('가')  | 1  |
| Oracle: LENGTHB('가') | 3  |

---

# 4️⃣ LENGTH가 중요한 실무 예

✔ 문자열 포맷이 일정한지 검증
✔ 전화번호·주민번호 길이 체크
✔ 이메일 최소 길이/최대 길이 검증
✔ 입력 데이터 정제(validation)
✔ 로그에서 고정길이 필드 파싱
✔ ETL에서 문자열 byte size 확인

---

# 5️⃣ 실습용 테이블 생성

SUBSTR 실습과 동일한 데이터셋을 사용합니다.

```sql
CREATE TABLE LogData (
    id        INT NOT NULL AUTO_INCREMENT,
    user_id   VARCHAR(20),
    phone     VARCHAR(20),
    email     VARCHAR(50),
    log_msg   VARCHAR(200),
    PRIMARY KEY (id)
);
```

---

# 6️⃣ 샘플 데이터 삽입

```sql
INSERT INTO LogData (user_id, phone, email, log_msg) VALUES
('U-2024-33', '010-1234-5678', 'kim@test.com', '[ERROR] Login failed'),
('X-2023-11', '010-7777-2222', 'lee@sample.co.kr', '[WARN] Password attempt'),
('A-9999-01', '02-333-5555', 'john.doe@gmail.com', '[INFO] Session start'),
('U-2022-58', '031-888-9999', 'minsu@company.com', '[ERROR] Token expired'),
('Z-7777-77', '010-0000-1111', 'guest@domain.net', '[DEBUG] Debug message');
```

---

# 7️⃣ LENGTH 실습 예제

---

## ✔ 예제 1) user_id 길이 확인

```sql
SELECT user_id, LENGTH(user_id) AS byte_len
FROM LogData;
```

### 결과

| user_id   | byte_len |
| --------- | -------- |
| U-2024-33 | 9        |
| X-2023-11 | 9        |
| A-9999-01 | 9        |
| U-2022-58 | 9        |
| Z-7777-77 | 9        |

---

## ✔ 예제 2) phone 길이 확인

```sql
SELECT phone, LENGTH(phone) AS len
FROM LogData;
```

### 결과

| phone         | len |
| ------------- | --- |
| 010-1234-5678 | 13  |
| 010-7777-2222 | 13  |
| 02-333-5555   | 11  |
| 031-888-9999  | 12  |
| 010-0000-1111 | 13  |

---

## ✔ 예제 3) 이메일 길이 확인

```sql
SELECT email, LENGTH(email) AS len
FROM LogData;
```

---

## ✔ 예제 4) 로그 메시지 전체 길이

```sql
SELECT log_msg, LENGTH(log_msg) AS bytes
FROM LogData;
```

---

## ✔ 예제 5) ’ERROR’ 문자열 길이

```sql
SELECT LENGTH('[ERROR]') AS err_len;
```

### 결과

| err_len |
| ------- |
| 7       |

---

## ✔ 예제 6) 한글 길이 비교 (ASCII vs UTF8)

```sql
SELECT
    LENGTH('가나다') AS byte_len,
    CHAR_LENGTH('가나다') AS char_len;
```

### 결과

| byte_len | char_len |
| -------- | -------- |
| 9        | 3        |

---

## ✔ 예제 7) 이메일 아이디 부분 길이

(SUBSTR + LENGTH)

```sql
SELECT 
    email,
    SUBSTR(email, 1, INSTR(email, '@') - 1) AS email_id,
    LENGTH(SUBSTR(email, 1, INSTR(email, '@') - 1)) AS email_id_len
FROM LogData;
```

---

## ✔ 예제 8) 도메인 길이 계산

```sql
SELECT
    email,
    SUBSTR(email, INSTR(email, '@') + 1) AS domain,
    LENGTH(SUBSTR(email, INSTR(email, '@') + 1)) AS domain_len
FROM LogData;
```

---

## ✔ 예제 9) 로그 레벨의 길이

```sql
SELECT
    log_msg,
    SUBSTR(log_msg, 2, INSTR(log_msg, ']') - 2) AS log_level,
    LENGTH(SUBSTR(log_msg, 2, INSTR(log_msg, ']') - 2)) AS level_len
FROM LogData;
```

---

## ✔ 예제 10) 전화번호 마지막 4자리 길이 (항상 4여야 함)

```sql
SELECT phone, LENGTH(SUBSTR(phone, LENGTH(phone) - 3, 4)) AS len_last4
FROM LogData;
```

---

## ✔ 예제 11) 데이터 품질 검사 – user_id가 9바이트인지 체크

```sql
SELECT
    user_id,
    LENGTH(user_id) AS len,
    CASE WHEN LENGTH(user_id) = 9 THEN 'OK' ELSE 'ERROR' END AS status
FROM LogData;
```

---

## ✔ 예제 12) 이메일이 5바이트 이하인 경우 오류 처리

```sql
SELECT
    email,
    LENGTH(email) AS len,
    CASE WHEN LENGTH(email) < 6 THEN 'Invalid' ELSE 'Valid' END AS validate
FROM LogData;
```

---

## ✔ 예제 13) 로그 메시지의 “본문 내용 길이” 계산

```sql
SELECT
    log_msg,
    SUBSTR(log_msg, INSTR(log_msg, ']') + 2) AS body,
    LENGTH(SUBSTR(log_msg, INSTR(log_msg, ']') + 2)) AS body_len
FROM LogData;
```

---

## ✔ 예제 14) 데이터 레코드 전체 길이 합산

```sql
SELECT
    LENGTH(user_id) +
    LENGTH(phone) +
    LENGTH(email) +
    LENGTH(log_msg) AS total_bytes
FROM LogData;
```

---

## ✔ 예제 15) LENGTH 조건 필터링

예: 전화번호가 **12바이트 이상**인 사용자만 조회

```sql
SELECT *
FROM LogData
WHERE LENGTH(phone) >= 12;
```

---

# 8️⃣ LENGTH 사용 시 반드시 알아야 할 전문가 팁

---

### ✔ 1) LENGTH는 문자 수가 아닌 “바이트 수”

UTF8 다국어 환경(한국어/일본어/중국어)에서는
문자 수와 바이트 수가 다릅니다.

---

### ✔ 2) VARCHAR(n)은 “문자 수” 제한

예를 들어 VARCHAR(10)에 한글 10자를 넣으면?
→ 오류 발생 (10자가 30바이트)

---

### ✔ 3) Oracle의 LENGTH는 문자 수, LENGTHB가 바이트 수

다른 DBMS와 혼동하면 장애 발생.

---

### ✔ 4) LENGTH는 인덱스를 사용하지 않음 (WHERE 절)

```sql
WHERE LENGTH(email) < 10  -- 인덱스 X
```

---

### ✔ 5) 바이너리(Binary) 비교가 필요한 경우 OCTET_LENGTH 사용 (Postgres)

