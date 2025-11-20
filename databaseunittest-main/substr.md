# 🚀 SQL SUBSTR

---

# 1️⃣ SUBSTR 함수란?

**SUBSTR는 문자열에서 특정 위치부터 일정 길이만큼 잘라 반환하는 함수**입니다.

### ✔ 공식 정의

```
SUBSTR(string, start, length)
```

| 매개변수       | 설명                  |
| ---------- | ------------------- |
| **string** | 자를 대상 문자열           |
| **start**  | 시작 위치 (1부터 시작)      |
| **length** | 가져올 문자 수 (OPTIONAL) |

### ✔ length 생략 시

해당 시작 위치부터 **문자열 끝까지** 반환합니다.

```
SUBSTR('abcdef', 3) → 'cdef'
```

---

# 2️⃣ SUBSTR의 동작 원리 (전문가 관점)

### 🔸 (1) 문자열을 “문자 단위(Character Unit)”로 파싱

SQL의 SUBSTR은 바이트가 아니라 **문자 단위**로 계산합니다.
UTF-8 기반이라도 `문자` 기준으로 처리합니다.

예:
`'한글ABC'` 에서 SUBSTR(‘한글ABC’, 1, 2) → ‘한글’

---

### 🔸 (2) 인덱스는 1부터 시작

SUBSTR이 **대부분의 DBMS에서 인덱스가 1부터** 시작합니다.

즉, 0 또는 -1로 시작하면 DBMS마다 동작이 다릅니다.

---

### 🔸 (3) DBMS별 차이가 존재함

SUBSTR은 거의 모든 DBMS에서 지원하지만, **음수 인덱스 지원 여부**와 **함수 이름**이 조금 다릅니다.

---

# 3️⃣ DBMS별 SUBSTR 비교

| DBMS           | 함수명               | 인덱스 시작 | 음수 인덱스   | 비고                      |
| -------------- | ----------------- | ------ | -------- | ----------------------- |
| **MySQL**      | SUBSTR, SUBSTRING | 1      | ❌ 지원 안함  | LEFT, RIGHT도 존재         |
| **Oracle**     | SUBSTR            | 1      | ✔ 지원     | 음수는 뒤에서부터 계산            |
| **PostgreSQL** | SUBSTRING         | 1      | ❌ 기본 미지원 | REGEXP 기반 substring도 강력 |
| **SQL Server** | SUBSTRING         | 1      | ❌ 지원 안함  | len, charindex와 조합      |

### ✔ Oracle에서 음수 예시

```
SUBSTR('ABCDEFG', -3, 2) → 'EF'
```

“뒤에서 3번째부터 2글자” 의 의미입니다.

MySQL·PostgreSQL에서는 이 문법이 에러입니다.

---

# 4️⃣ SUBSTR vs LEFT/RIGHT vs REGEXP_SUBSTR

SUBSTR은 범용성이 최고지만,
특정 패턴에서는 다음 내장 함수를 부담없이 사용합니다.

| 함수                | 설명        | 예시                |
| ----------------- | --------- | ----------------- |
| **SUBSTR**        | 가장 범용적    | SUBSTR(str, 3, 2) |
| **LEFT**          | 앞에서 n글자   | LEFT(str, 3)      |
| **RIGHT**         | 뒤에서 n글자   | RIGHT(str, 4)     |
| **REGEXP_SUBSTR** | 정규식 기반 추출 | 주민번호, 전화번호 파싱     |

실무에서는
**SUBSTR → LEFT/RIGHT → REGEXP_SUBSTR**
순으로 난이도가 올라갑니다.

---

# 5️⃣ SUBSTR + LENGTH 조합이 중요한 이유

SUBSTR은 “정적 인덱스”를 기준으로 잘라냅니다.
하지만 real-world 데이터는 포맷이 일정하지 않습니다.

**LENGTH()와 결합하면 가변 길이 문자열에서도 유연**하게 파싱할 수 있습니다.

예:

마지막 4자리 추출:

```
SUBSTR(phone, LENGTH(phone) - 3, 4)
```

이 조합은 로그 파싱, 코드 처리, 전화번호 처리에 필수입니다.

---

# 6️⃣ 실습용 테이블 생성


```sql
CREATE TABLE LogData (
    id        INT NOT NULL AUTO_INCREMENT,
    user_id   VARCHAR(20),      -- 예: U-2024-33
    phone     VARCHAR(20),      -- 예: 010-1234-5678
    email     VARCHAR(50),      -- 예: kim@test.com
    log_msg   VARCHAR(200),     -- 예: [ERROR] Login failed
    PRIMARY KEY (id)
);
```

---

# 7️⃣ 샘플 데이터 삽입

```sql
INSERT INTO LogData (user_id, phone, email, log_msg) VALUES
('U-2024-33', '010-1234-5678', 'kim@test.com', '[ERROR] Login failed'),
('X-2023-11', '010-7777-2222', 'lee@sample.co.kr', '[WARN] Password attempt'),
('A-9999-01', '02-333-5555', 'john.doe@gmail.com', '[INFO] Session start'),
('U-2022-58', '031-888-9999', 'minsu@company.com', '[ERROR] Token expired'),
('Z-7777-77', '010-0000-1111', 'guest@domain.net', '[DEBUG] Debug message');
```

---

# 8️⃣ SUBSTR 실습


---

## ✔ 예제 1) user_id 앞 1글자(유형코드) 추출

```sql
SELECT user_id, SUBSTR(user_id, 1, 1) AS type_cd
FROM LogData;
```

### 결과

| user_id   | type_cd |
| --------- | ------- |
| U-2024-33 | U       |
| X-2023-11 | X       |
| A-9999-01 | A       |
| U-2022-58 | U       |
| Z-7777-77 | Z       |

---

## ✔ 예제 2) 연도(year)만 추출

U-2024-33 → 2024

```sql
SELECT user_id, SUBSTR(user_id, 3, 4) AS year
FROM LogData;
```

### 결과

| user_id   | year |
| --------- | ---- |
| U-2024-33 | 2024 |
| X-2023-11 | 2023 |
| A-9999-01 | 9999 |
| U-2022-58 | 2022 |
| Z-7777-77 | 7777 |

---

## ✔ 예제 3) user_id 마지막 2자리 동적 추출

```sql
SELECT user_id,
       SUBSTR(user_id, LENGTH(user_id) - 1, 2) AS seq_no
FROM LogData;
```

### 결과

| user_id   | seq_no |
| --------- | ------ |
| U-2024-33 | 33     |
| X-2023-11 | 11     |
| A-9999-01 | 01     |
| U-2022-58 | 58     |
| Z-7777-77 | 77     |

---

## ✔ 예제 4) 전화번호 앞자리(지역/통신사번호)

```sql
SELECT phone, SUBSTR(phone, 1, 3) AS prefix
FROM LogData;
```

### 결과

| phone         | prefix |
| ------------- | ------ |
| 010-1234-5678 | 010    |
| 010-7777-2222 | 010    |
| 02-333-5555   | 02-    |
| 031-888-9999  | 031    |
| 010-0000-1111 | 010    |

---

## ✔ 예제 5) 전화번호 뒤 4자리

```sql
SELECT phone,
       SUBSTR(phone, LENGTH(phone) - 3, 4) AS last_4
FROM LogData;
```

### 결과

| phone         | last_4 |
| ------------- | ------ |
| 010-1234-5678 | 5678   |
| 010-7777-2222 | 2222   |
| 02-333-5555   | 5555   |
| 031-888-9999  | 9999   |
| 010-0000-1111 | 1111   |

---

## ✔ 예제 6) 이메일에서 '@' 앞의 ID만 추출

```sql
SELECT email,
       SUBSTR(email, 1, INSTR(email, '@') - 1) AS email_id
FROM LogData;
```

### 결과

| email                                           | email_id |
| ----------------------------------------------- | -------- |
| [kim@test.com](mailto:kim@test.com)             | kim      |
| [lee@sample.co.kr](mailto:lee@sample.co.kr)     | lee      |
| [john.doe@gmail.com](mailto:john.doe@gmail.com) | john.doe |
| [minsu@company.com](mailto:minsu@company.com)   | minsu    |
| [guest@domain.net](mailto:guest@domain.net)     | guest    |

---

## ✔ 예제 7) 이메일 도메인 추출

```sql
SELECT email,
       SUBSTR(email, INSTR(email, '@') + 1) AS domain
FROM LogData;
```

### 결과

| email                                           | domain       |
| ----------------------------------------------- | ------------ |
| [kim@test.com](mailto:kim@test.com)             | test.com     |
| [lee@sample.co.kr](mailto:lee@sample.co.kr)     | sample.co.kr |
| [john.doe@gmail.com](mailto:john.doe@gmail.com) | gmail.com    |
| [minsu@company.com](mailto:minsu@company.com)   | company.com  |
| [guest@domain.net](mailto:guest@domain.net)     | domain.net   |

---

## ✔ 예제 8) 로그 메시지에서 레벨([ERROR], [WARN]) 추출

```sql
SELECT log_msg,
       SUBSTR(log_msg, 2, INSTR(log_msg, ']') - 2) AS log_level
FROM LogData;
```

### 결과

| log_msg                 | log_level |
| ----------------------- | --------- |
| [ERROR] Login failed    | ERROR     |
| [WARN] Password attempt | WARN      |
| [INFO] Session start    | INFO      |
| [ERROR] Token expired   | ERROR     |
| [DEBUG] Debug message   | DEBUG     |

---

## ✔ 예제 9) 로그 내용만 추출

```sql
SELECT log_msg,
       SUBSTR(log_msg, INSTR(log_msg, ']') + 2) AS message
FROM LogData;
```

### 결과

| log_msg                 | message          |
| ----------------------- | ---------------- |
| [ERROR] Login failed    | Login failed     |
| [WARN] Password attempt | Password attempt |
| [INFO] Session start    | Session start    |
| [ERROR] Token expired   | Token expired    |
| [DEBUG] Debug message   | Debug message    |

---

## ✔ 예제 10) 전화번호 중간 4자리

```sql
SELECT phone, SUBSTR(phone, 5, 4) AS mid_4
FROM LogData;
```

---

## ✔ 예제 11) user_id에서 두 번째 '-' 이후 값 추출

(LOCATE + SUBSTR 조합)

```sql
SELECT user_id,
       SUBSTR(
           user_id,
           LOCATE('-', user_id, LOCATE('-', user_id) + 1) + 1
       ) AS last_token
FROM LogData;
```

---

## ✔ 예제 12) SUBSTR + WHERE → user_id가 ‘U’로 시작하는 사용자

```sql
SELECT *
FROM LogData
WHERE SUBSTR(user_id, 1, 1) = 'U';
```

---

## ✔ 예제 13) SUBSTR + CASE → 로그 레벨 한국어 변환

```sql
SELECT
    log_msg,
    SUBSTR(log_msg, 2, INSTR(log_msg, ']') - 2) AS level_cd,
    CASE 
        WHEN SUBSTR(log_msg, 2, 5) = 'ERROR' THEN '오류'
        WHEN SUBSTR(log_msg, 2, 4) = 'WARN'  THEN '경고'
        WHEN SUBSTR(log_msg, 2, 4) = 'INFO'  THEN '정보'
        ELSE '기타'
    END AS level_kor
FROM LogData;
```

---

## ✔ 예제 14) 이메일 TLD(.com, .net 등) 추출 (가변 길이)

```sql
SELECT
    email,
    SUBSTR(email, LENGTH(email) - LENGTH(SUBSTRING_INDEX(email, '.', -1)) + 1)
        AS tld
FROM LogData;
```

---

## ✔ 예제 15) 로그 메시지 앞 10글자 요약

```sql
SELECT log_msg,
       SUBSTR(log_msg, 1, 10) AS summary
FROM LogData;
```


