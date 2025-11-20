# 🔍 SQL 문자열 함수: SUBSTR / LENGTH 제대로 이해하기

**– Oracle · MySQL · PostgreSQL 비교 가이드 –**

데이터 분석이나 백엔드 개발을 하다 보면 문자열을 다루는 일이 정말 많습니다.
그중에서도 **가장 자주 등장하는 함수 두 가지가 바로 `SUBSTR`와 `LENGTH`** 입니다.

두 함수는 이름만 보면 단순해 보이지만,
**DBMS마다 동작 방식이 미묘하게 다르고(특히 LENGTH), 한글 처리에서 더 큰 차이**가 나기 때문에
정확히 이해하지 않으면 의도치 않은 결과가 나오기 쉽습니다.

이번 글에서는 다음을 중심으로 정리합니다.

* `SUBSTR`로 문자열 자르기 (Oracle / MySQL / PostgreSQL 비교)
* `LENGTH`는 문자 수? 바이트 수?
* 한글 문자열 처리에서 반드시 알아야 할 차이
* 실무 패턴 & 단위평가/면접 포인트

---

# 1. SUBSTR – 문자열 일부를 잘라내는 함수

## 📌 기본 형태

```sql
SUBSTR(문자열, 시작위치 [, 길이])
```

* 시작 인덱스는 대부분 DB에서 **1부터 시작**
* `길이` 생략 시: **끝까지**
* 일부 DB는 **음수 시작 위치** 지원 → 뒤에서부터 자르기 가능

---

# 2. Oracle의 SUBSTR

Oracle은 `SUBSTR` 하나로 충분합니다.

### ✔ 기본 예제

```sql
SELECT SUBSTR('HELLO WORLD', 1, 5) FROM dual;
-- 결과: 'HELLO'
```

### ✔ 시작위치 음수 지원

```sql
SELECT SUBSTR('HELLO WORLD', -5, 3) FROM dual;
-- 'WOR'
```

뒤에서 5번째 문자(W)부터 3글자 잘라냄.

### ✔ 길이 생략

```sql
SELECT SUBSTR('HELLO WORLD', 7) FROM dual;
-- 'WORLD'
```

---

# 3. MySQL의 SUBSTRING / SUBSTR

MySQL은 `SUBSTRING` 또는 `SUBSTR` 모두 동일하게 동작합니다.

### ✔ 사용 예시

```sql
SELECT SUBSTRING('HELLO WORLD', 1, 5);   -- 'HELLO'
SELECT SUBSTRING('HELLO WORLD', 7);      -- 'WORLD'
```

### ✔ 음수 인덱스 지원

```sql
SELECT SUBSTRING('HELLO WORLD', -5, 3);  -- 'WOR'
```

---

# 4. PostgreSQL의 SUBSTRING

PostgreSQL은 표준 SQL 스타일을 따르며 함수형 문법을 많이 사용합니다.

```sql
SELECT SUBSTRING('HELLO WORLD' FROM 1 FOR 5);  -- 'HELLO'
```

### ✔ 끝까지 자르기

```sql
SELECT SUBSTRING('HELLO WORLD' FROM 7);        -- 'WORLD'
```

### ✔ MySQL/Oracle 스타일도 일부 버전에서 지원

```sql
SELECT SUBSTRING('HELLO WORLD', 1, 5);         -- 가능 (버전별 상이)
```

---

# 5. LENGTH – 길이 계산 (문자 수? 바이트 수?)

가장 헷갈리는 부분입니다.

---

# 🚨 핵심 결론: DB마다 의미가 다르다!

| DBMS           | LENGTH         | 문자 개수 | 바이트 수                  |
| -------------- | -------------- | ----- | ---------------------- |
| **Oracle**     | LENGTH = 문자 수  | ✔     | ✘ (`LENGTHB`로 계산)      |
| **MySQL**      | LENGTH = 바이트 수 | ✘     | ✔                      |
| **PostgreSQL** | LENGTH = 문자 수  | ✔     | ✘ (`OCTET_LENGTH`로 계산) |

---

# 6. Oracle의 LENGTH

Oracle은 **기본적으로 문자 수 기준**입니다.

```sql
SELECT LENGTH('가나')    -- 2
FROM dual;
```

바이트 계산은 별도 함수:

```sql
SELECT LENGTHB('가나')   -- 6 (UTF-8 기준)
FROM dual;
```

---

# 7. MySQL의 LENGTH / CHAR_LENGTH

MySQL은 크게 다릅니다.

### ✔ LENGTH → 바이트 수

```sql
SELECT LENGTH('ABC');      -- 3
SELECT LENGTH('가나');     -- 6  (UTF-8: 한글 1자 = 3바이트)
```

### ✔ CHAR_LENGTH → 문자 수

```sql
SELECT CHAR_LENGTH('가나');   -- 2
```

문자 수 기준 연산을 해야 할 때 반드시 `CHAR_LENGTH` 사용해야 합니다.

---

# 8. PostgreSQL의 LENGTH

PostgreSQL은 Oracle처럼 문자 수를 반환합니다.

```sql
SELECT LENGTH('가나');        -- 2
```

바이트 수는 `OCTET_LENGTH` 사용:

```sql
SELECT OCTET_LENGTH('가나');  -- 6
```

---

# 9. SUBSTR + LENGTH 조합: 실무에서 가장 자주 쓰는 패턴

## ① 문자열 끝에서 N글자 추출

### Oracle

```sql
SELECT SUBSTR(file_name, LENGTH(file_name) - 3) AS last4
FROM files;
```

### MySQL (음수 인덱스 지원)

```sql
SELECT SUBSTR(file_name, -4) AS last4
FROM files;
```

---

## ② 이메일 아이디/도메인 분리

```sql
SELECT
    SUBSTR(email, 1, INSTR(email, '@') - 1) AS user_id,
    SUBSTR(email, INSTR(email, '@') + 1) AS domain
FROM member;
```

---

## ③ 주민등록번호 앞 6자리 분리

```sql
SELECT SUBSTR(jumin_no, 1, 6) AS birth
FROM member;
```

---

## ④ 휴대폰번호 검사 (MySQL 기준)

```sql
SELECT *
FROM member
WHERE SUBSTR(phone, 1, 3) = '010'
  AND CHAR_LENGTH(phone) = 11;
```

---

# 10. SUBSTR / LENGTH 시험·면접에서 자주 나오는 포인트

### ✔ SUBSTR의 인덱스는 1부터 시작

(0 아님)

### ✔ MySQL LENGTH는 “문자 수”가 아니다

(UTF-8 한글 1자 = 3바이트)

### ✔ 공백 문자열 체크 시 LENGTH(TRIM()) 사용

NULL은 결과가 NULL이므로 주의

### ✔ 음수 인덱스는 Oracle/MySQL만 확실하게 지원

PostgreSQL은 제한적

---

# 11. 실전 예제: 문자열 함수 오류 방지 체크리스트

### 🔑 체크 1: 지금 필요한 기준은 “문자 수”인가, “바이트 수”인가?

* 이름/주소/이메일 길이 검사는 → **문자 수**
* 저장공간/패딩/바이트 제한 검사는 → **바이트 수**

---

### 🔑 체크 2: 다국어(한글) 환경이면 반드시 CHAR_LENGTH, LENGTHB, OCTET_LENGTH 중 하나 선택해야 한다.

---

### 🔑 체크 3: 생산일자나 파일명 패턴을 SUBSTR로 잘라낼 때는 위치 오프셋 실수 주의


