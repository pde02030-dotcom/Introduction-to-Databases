# 🚀 INSTR 함수 완전 정복

`INSTR()` 함수는 SQL에서 문자열 안에 특정 문자열이 **어떤 위치에 존재하는지** 찾는 함수입니다.
하지만 실무에서는 단순 검색이 아니라:

* 문자열 파싱
* 로그 분석
* URL 구조 해석
* 코드/패턴 분석
* ETL 데이터 정제
* LIKE 대비 성능 차이
* 전문화된 텍스트 인덱스 활용 여부

등에서 매우 중요한 역할을 합니다.

---

# 1️⃣ INSTR 함수란?

INSTR는 다음을 반환합니다:

> **“주어진 문자열에서 특정 문자열이 나타나는 첫 번째 위치(1-based index)”**

즉:

```sql
INSTR('abcdef', 'cd')  → 3
```

✔ 찾는 문자열이 없다면 → 0
✔ 위치는 1부터 시작 (MySQL/Oracle 동일)

---

# 2️⃣ 기본 사용법

```
INSTR(string, substring)
```

예:

```sql
SELECT INSTR('Hello SQL World', 'SQL');
```

결과:

```
7
```

문자열 `"Hello "` = 6글자
→ SQL의 시작은 7번째.

---

# 3️⃣ 실습용 테이블 생성

```sql
CREATE TABLE Articles (
    id        INT AUTO_INCREMENT PRIMARY KEY,
    title     VARCHAR(200),
    content   TEXT
);
```

---

# 4️⃣ 샘플 데이터 삽입

```sql
INSERT INTO Articles (title, content) VALUES
('SQL 튜토리얼', '이 글에서는 MySQL의 INSTR 함수 사용법을 설명합니다.'),
('Docker 가이드', '컨테이너 기반 배포에 대해 설명합니다.'),
('웹 개발', 'JavaScript에서 문자열 검색은 indexOf 함수를 사용합니다.'),
('DB Index', 'INSTR 함수와 LIKE의 성능 차이를 다룹니다.');
```

---

# 5️⃣ 실습 예제 + 결과

---

## ✔ 예제 1) title 컬럼에서 'SQL' 위치 찾기

```sql
SELECT id, title, INSTR(title, 'SQL') AS pos
FROM Articles;
```

### 결과:

| id | title      | pos |
| -- | ---------- | --- |
| 1  | SQL 튜토리얼   | 1   |
| 2  | Docker 가이드 | 0   |
| 3  | 웹 개발       | 0   |
| 4  | DB Index   | 0   |

---

## ✔ 예제 2) content 안에서 '함수'가 존재하는 위치

```sql
SELECT id, INSTR(content, '함수') AS pos
FROM Articles;
```

### 결과 예:

| id | pos |
| -- | --- |
| 1  | 17  |
| 2  | 0   |
| 3  | 0   |
| 4  | 8   |

---

## ✔ 예제 3) INSTR을 조건식으로 활용하기 (WHERE)

특정 단어 포함된 row만 찾기:

```sql
SELECT *
FROM Articles
WHERE INSTR(content, 'INSTR') > 0;
```

결과:

* 1번 row
* 4번 row

(이 두 row는 content 내에 "INSTR" 문자열 포함)

---

# 6️⃣ INSTR vs LIKE — 성능 차이는?

| 기능               | 특징                             |
| ---------------- | ------------------------------ |
| **INSTR**        | 내부적으로 substring search 알고리즘 사용 |
| **LIKE '%str%'** | Full scan 필요, 인덱스 거의 못 씀       |
| **LIKE 'str%'**  | 인덱스 사용 가능(좌측 고정)               |

중요:

### 📌 INSTR('text', 'str') 는

`LIKE '%str%'` 패턴과 **동일한 의미**이다.

즉, INSTR도 대부분 인덱스를 사용할 수 없다.
(왼쪽 일치가 아니기 때문에)

---

# 7️⃣ INSTR의 내부 동작 방식 (전문가 설명)

DBMS는 보통 문자열 검색에 아래 알고리즘을 사용한다:

* Naive Search
* Boyer–Moore
* Boyer–Moore–Horspool
* Knuth–Morris–Pratt(KMP)

MySQL의 SQL 레벨 INSTR는 내부적으로 효율적인 검색 알고리즘을 사용하지만
**텍스트 인덱스를 사용하지 않는 한 테이블 full scan이 발생**한다.

즉,
**대량 데이터에서는 INSTR 남용 → 성능 저하**

---

# 8️⃣ INSTR 성능 최적화 전략

### ✔ 1) 가능한 LIKE 'prefix%' 형태로 바꾼다

(INDEX 사용 가능)

### ✔ 2) 전문 검색 인덱스(Full-text index) 사용

MySQL:

```sql
ALTER TABLE Articles ADD FULLTEXT(content);
```

그 후:

```sql
SELECT * FROM Articles WHERE MATCH(content) AGAINST ('INSTR');
```

→ INSTR보다 10~100배 빠름

### ✔ 3) JSON 검색으로 치환

텍스트 특정 패턴 기반이라면 JSON 구조로 저장 후 JSON_QUERY 활용

### ✔ 4) 튜닝 포인트

* INSTR은 문자열 길이에 비례하여 CPU 사용
* 긴 TEXT 컬럼일수록 비용 증가

---

# 9️⃣ INSTR과 SUBSTR 조합 활용

문자열 파싱에서 자주 사용됨.

예) 이메일 아이디 추출:

```sql
SELECT SUBSTR(email, 1, INSTR(email, '@') - 1)
FROM Member;
```

예) URL에서 도메인 추출:

```sql
SELECT SUBSTR(url, 1, INSTR(url, '/', 1, 3) - 1)
```

---

# 🔟 Oracle vs MySQL INSTR 차이

### ✔ MySQL

`INSTR(str, substr)` — 2개의 인자만 허용
문자 찾으면 위치 반환, 못 찾으면 0

---

### ✔ Oracle

인자가 4개까지 가능

```
INSTR(string, substring, start_position, nth_occurrence)
```

예:

```sql
SELECT INSTR('a,b,c,d', ',', 1, 2) FROM dual;
```

→ 두 번째 콤마 위치 반환

MySQL에는 없는 강력한 기능.

---

# 1️⃣1️⃣ 실무에서 INSTR을 언제 사용하는가?

* 문자열 파싱
* 특정 패턴 포함 여부 체크
* 로그/메시지 분석
* URL, Path, 코드 분석
* CSV-like 문자열 처리
* ETL 과정에서 데이터 정제

정규표현식보다 가벼워서
조건 검색에서 자주 쓰인다.

---

# ✨ 최종 요약

> **INSTR은 문자열 내에서 특정 문자열이 위치한 인덱스를 반환하는 함수이며,
> LIKE '%str%'와 동일한 패턴 검색으로 대부분 인덱스를 사용할 수 없기 때문에
> 대량 데이터에서는 성능 주의가 필요하다.
> SUBSTR과 조합해서 문자열 파싱 시 매우 유용하다.**

