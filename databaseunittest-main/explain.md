# 🚀 EXPLAIN 

많은 개발자가 EXPLAIN 출력을 보며:

* “type=ref면 좋은 건가?”
* “rows=10은 많나 적나?”
* “Using filesort가 붙으면 뭐가 나쁜 거지?”

라는 수준으로만 이해합니다.
그러나 **실무에서는 EXPLAIN을 해석하는 능력이 SQL 성능의 80%를 결정**합니다.

데이터가 10만 건, 100만 건, 1억 건 넘어가면
인덱스 하나, 조건 하나 때문에
쿼리 속도가 **0.05초 → 12초**로 폭발적으로 변할 수 있습니다.

그리고 이런 문제는 90%가 **EXPLAIN만 제대로 보면 바로 해결**됩니다.

---

# 1️⃣ MySQL 옵티마이저는 어떻게 실행 계획을 만드는가?

MySQL의 옵티마이저는 다음 순서로 실행 계획을 만든다:

---

### ✔ Step 1. 통계(Statistics) 수집

각 컬럼에 대해 **카디널리티(유니크 비율)** 를 기반으로
“어떤 인덱스가 더 효율적인지” 판단한다.

예:

* email → 거의 모두 UNIQUE → 인덱스 사용 가치 매우 높음
* gender(M/F) → 2가지 → 인덱스 만들어도 무의미

---

### ✔ Step 2. 가능한 모든 인덱스 조합 탐색

EXPLAIN의 *possible_keys* 는
“옵티마이저가 고려한 모든 인덱스 후보”를 의미한다.

---

### ✔ Step 3. 비용 기반(Cost-Based) 최적 계획 선택

아래 비용을 모두 계산해서 가장 싼 계획을 선택:

* 인덱스 탐색 비용(B-tree depth)
* 예상 row 수
* join buffer 크기
* 정렬 비용
* 임시 테이블 비용
* random I/O 발생 가능성

이 최종 결과물이 EXPLAIN이다.

---

# 2️⃣ EXPLAIN 주요 컬럼 심층 분석

EXPLAIN의 핵심은 다음 6개다:

1. **type**
2. **key**
3. **rows**
4. **filtered**
5. **Extra**
6. **key_len**

이 6개만 정확히 읽을 수 있으면 **쿼리 병목의 90%는 바로 해결**된다.

---

# 3️⃣ type — 쿼리 성능의 등급표(★ 완전 심화 버전)

아래 표는 단순 번역이 아니라
**실제로 내부적으로 어떤 작업량이 발생하는지까지 포함한 분석**입니다.

---

## 🔥 type 등급표 (실무용 심화 해석)

| type       | 내부 동작                                    | 성능 해석     |
| ---------- | ---------------------------------------- | --------- |
| **system** | 테이블에 row가 1개뿐                            | ★★★★★ 최상  |
| **const**  | PK/UNIQUE로 단 1개의 row만 읽음 (인덱스 depth ≈ 3) | ★★★★★     |
| **eq_ref** | JOIN 시 PK 기반 매칭 → row당 1건 정확히 찾음         | ★★★★★     |
| **ref**    | 인덱스를 이용해 여러 row 탐색 (동등 비교)               | ★★★★☆     |
| **range**  | 인덱스 범위 검색 (>, <, BETWEEN, LIKE 'abc%')   | ★★★☆☆     |
| **index**  | 인덱스 전체 스캔 (Full Index Scan)              | ★★☆☆☆     |
| **ALL**    | 인덱스도 없이 Full Table Scan                  | ★☆☆☆☆, 최악 |

---

### 🧠 실무 핵심 포인트

* **ALL 이면 90%는 성능 개선 포인트 존재**
* **range/ref/eq_ref/const를 목표**
* 이메일, 전화번호, id는 고카디널리티 → 인덱스 매우 잘 맞음
* 성별, 상태(0/1), YES/NO는 로우카디널리티 → 인덱스 무력화

---

# 4️⃣ rows — 얼마나 스캔하는지(실제 성능 지표)

rows는 MySQL이 **예상하는 탐색 row 수**입니다.
실제 행이 아니라 “예상치”지만, 거의 정확합니다.

### ✔ rows가 10, 100, 1,000, 10,000, 100,000 정도로 증가하면 성능 체감이 급격히 증가

예:

* rows = 10 → 몇 밀리초
* rows = 10,000 → 수십~수백 ms
* rows = 1,000,000 → 초 단위 발생

---

### ✔ rows × filtered = 실제 읽는 row 수

예:

```
rows=10000, filtered=10% → 실제 읽는 row 1,000
```

---

# 5️⃣ key_len — 인덱스를 얼마나 활용했는지

예:

```
idx (email(100), age)
```

key_len이 100보다 작으면 email 일부만 사용한 것이다.

또는:

```
(city, age)
```

조건이 city만 포함되면 key_len은 city 길이만 사용됨.

이것이 바로 **복합 인덱스의 왼쪽 우선 원칙**.

---

# 6️⃣ Extra — 성능 병목의 70%를 결정하는 지표

Extra의 메시지는 대부분 성능 영향을 의미한다:

| Extra                           | 의미 (심화)             | 성능    |
| ------------------------------- | ------------------- | ----- |
| **Using index**                 | 커버링 인덱스 → 테이블 접근 없음 | 최고    |
| **Using where**                 | 조건 적합한 row만 반환      | 정상    |
| **Using index condition (ICP)** | 인덱스에서 일부 조건 필터      | 좋음    |
| **Using filesort**              | 메모리/디스크 정렬 발생       | 느림    |
| **Using temporary**             | 임시 테이블 생성           | 느림    |
| **Using join buffer**           | 인덱스 없는 JOIN 발생      | 매우 느림 |

✔ filesort + temporary 동시에 나오면 **빨간불**
✔ covering index 이면 최고의 성능

---

# 7️⃣ 실습 테이블 생성

```sql
CREATE TABLE Member (
    id      INT NOT NULL AUTO_INCREMENT,
    name    VARCHAR(50),
    email   VARCHAR(100),
    age     INT,
    city    VARCHAR(50),
    PRIMARY KEY(id)
);
```

---

# 8️⃣ 샘플 데이터 삽입

```sql
INSERT INTO Member (name, email, age, city) VALUES
('김철수', 'kim@test.com', 25, 'Seoul'),
('이영희', 'lee@test.com', 32, 'Busan'),
('박민수', 'minsu@test.com', 19, 'Seoul'),
('최가영', 'ga@test.com', 28, 'Daegu'),
('홍길동', 'hong@test.com', 45, 'Seoul');
```

---

# 9️⃣ 인덱스 생성

```sql
CREATE INDEX idx_email ON Member(email);
CREATE INDEX idx_city_age ON Member(city, age);
```

---

# 10️⃣ EXPLAIN 실습 예제

아래 결과는 **type뿐 아니라 key, key_len, rows, extra까지 완전 해석**합니다.

---

## ✔ 예제 1) PK 검색 — type=const

```sql
EXPLAIN SELECT * FROM Member WHERE id=1;
```

**실무 해석**

* type=const → 단일 row만 읽음
* key=PRIMARY
* rows=1
* Extra=Using index → 커버링 인덱스

⭐ 최상의 실행 계획

---

## ✔ 예제 2) UNIQUE 검색

```sql
EXPLAIN SELECT * FROM Member WHERE email='kim@test.com';
```

결과 해석:

* type=ref
* key=idx_email
* rows≈1
* Extra=Using index

→ 커버링 인덱스 덕분에 데이터 페이지 접근 없음 → 매우 빠름

---

## ✔ 예제 3) city 단일 조건

```sql
EXPLAIN SELECT * FROM Member WHERE city='Seoul';
```

해석:

* type=ref
* key=idx_city_age
* rows≈3

→ 서울이 3건이라는 통계를 이용해 rows를 예측

---

## ✔ 예제 4) 범위 검색(range)

```sql
EXPLAIN SELECT * FROM Member WHERE city='Seoul' AND age>=20;
```

* type=range
* key=idx_city_age
* key_len = city + age 길이
* Extra: Using index condition

→ IDX의 두 컬럼 모두 사용됨 → 아주 좋은 계획

---

## ✔ 예제 5) LIKE '%문자' Full Scan

```sql
EXPLAIN SELECT * FROM Member WHERE email LIKE '%test.com';
```

→ key=NULL
→ type=ALL
→ rows=5
→ Extra: Using where

**앞부분 wildcard는 절대 인덱스 사용 불가**

---

## ✔ 예제 6) LIKE '문자%' 인덱스 사용

```sql
EXPLAIN SELECT * FROM Member WHERE email LIKE 'kim%';
```

→ type=range
→ idx_email 사용
→ 빠름

---

## ✔ 예제 7) ORDER BY 인덱스 포함

```sql
EXPLAIN SELECT * FROM Member ORDER BY email;
```

* key=idx_email
* Extra: Using index

→ 정렬 비용(filesort) 없음 → 인덱스 정렬 이용

---

## ✔ 예제 8) ORDER BY 인덱스 없음 → filesort

```sql
EXPLAIN SELECT * FROM Member ORDER BY name;
```

→ key=NULL
→ type=ALL
→ Extra: Using filesort

---

## ✔ 예제 9) GROUP BY와 인덱스

```sql
EXPLAIN SELECT city, COUNT(*) FROM Member GROUP BY city;
```

→ idx_city_age 활용
→ Extra: Using index

---

## ✔ 예제 10) GROUP BY + filesort

```sql
EXPLAIN SELECT age, COUNT(*) FROM Member GROUP BY age;
```

→ Extra: Using temporary; Using filesort
→ 매우 느린 패턴

---

## ✔ 예제 11) JOIN + 인덱스 사용 여부

(JOIN 테이블은 생략)

---

## ✔ 예제 12) WHERE 조건에서 함수 사용

```sql
EXPLAIN SELECT * FROM Member WHERE LOWER(name)='김철수';
```

→ type=ALL
→ key=NULL
→ 인덱스 무효화

---

## ✔ 예제 13) FORCE INDEX

```sql
EXPLAIN SELECT * FROM Member FORCE INDEX(idx_city_age) WHERE city='Seoul';
```

---

## ✔ 예제 14) 커버링 인덱스

```sql
EXPLAIN SELECT email FROM Member WHERE email='kim@test.com';
```

→ Extra=Using index
→ 데이터 페이지(access) 없음

---

## ✔ 예제 15) EXPLAIN ANALYZE (실제 실행 시간 확인)

```sql
EXPLAIN ANALYZE
SELECT * FROM Member WHERE city='Seoul';
```

출력 예:

```
Rows examined: 3
Actual time: 0.061 ms
Index Lookup
```

---

# 11️⃣ EXPLAIN 전문가 실무 팁 20선

### ✔ 1) type=ALL이면 거의 항상 인덱스 문제

### ✔ 2) filesort + temporary 조합은 반드시 튜닝

### ✔ 3) LIKE '%abc'는 절대 인덱스 사용 불가

### ✔ 4) 복합 인덱스는 “왼쪽 컬럼”이 핵심

### ✔ 5) WHERE 함수 사용 금지 (LOWER, SUBSTR 등)

### ✔ 6) 정렬 컬럼에는 인덱스 강제

### ✔ 7) JOIN 컬럼은 반드시 인덱스 필요

### ✔ 8) GROUP BY는 인덱스가 있으면 10배 빠름

### ✔ 9) covering index는 최강의 성능

### ✔ 10) rows × filtered 로 실제 읽는 row 계산

### ✔ 11) key_len을 보고 인덱스 활용 정도 확인

### ✔ 12) 한 테이블에 인덱스 너무 많으면 DML 느려짐

### ✔ 13) OR 조건은 인덱스 무력화 가능

### ✔ 14) 다른 컬럼 계산식(where col+1=3)은 인덱스 사용 안함

### ✔ 15) EXPLAIN ANALYZE로 실제 실행 시간 체크

### ✔ 16) 옵티마이저는 비용 기반(CBO)

### ✔ 17) 실행 계획은 데이터량에 따라 계속 바뀔 수 있음

### ✔ 18) rows 예측이 틀리면 잘못된 실행계획이 선택됨

### ✔ 19) 지나치게 많은 인덱스는 오히려 성능 저하

### ✔ 20) SQL 성능 튜닝의 90%는 "어떤 인덱스를 만들지"에 달려 있음

---



