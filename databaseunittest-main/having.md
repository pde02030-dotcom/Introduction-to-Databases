

# 🚀 SQL HAVING

SQL에서 초보자가 가장 헷갈리는 문법 1순위는
**“WHERE와 HAVING의 차이”** 입니다.

HAVING 절은 단순히 “집계 조건을 걸 때 사용한다” 정도로만 알려져 있지만,
실무에서는 **데이터 품질 검증, 통계 필터링, 구간 분석, 이상치 제거** 등
고급 데이터 분석에 반드시 필요한 핵심 문법입니다.

---

# 1️⃣ HAVING 절이란?

HAVING 은 **GROUP BY로 집계(Aggregation)한 결과에 조건을 적용하는 절**이다.

핵심:

* WHERE: **그룹핑 이전** 개별 로우에 조건 적용
* HAVING: **그룹핑 이후** 요약된 결과에 조건 적용

예:

```
'평균 점수가 80 이상인 과목만 조회'
'학생별 총점이 200점 이상인 학생'
'에러 로그 수가 10건 이상인 사용자'
```

이는 WHERE 로는 절대 불가능하다.
HAVING 으로만 가능한 조건이다.

---

# 2️⃣ HAVING과 WHERE 차이 (정말 중요)

| 항목       | WHERE     | HAVING        |
| -------- | --------- | ------------- |
| 적용 시점    | 그룹핑 이전    | 그룹핑 이후        |
| 적용 대상    | 개별 행(Row) | 그룹(Row Group) |
| 집계 함수 사용 | ❌ 사용 불가   | ✔ 사용 가능       |
| 성능       | 더 좋음      | 그룹 후 필터 → 비용↑ |
| 사용 목적    | 필터링       | 집계된 결과 필터링    |

---

# 3️⃣ SQL 실행 순서 (HAVING 위치 이해 핵심)

SQL 내부 실행 순서:

1. FROM
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. ORDER BY

즉,
**HAVING은 GROUP BY 다음에 실행되므로 집계 결과를 대상으로만 필터링 가능**하다.

---

# 4️⃣ 실습용 테이블 생성

GROUP BY 실습과 같은 구조를 사용합니다.

```sql
CREATE TABLE StudentScore (
    id        INT NOT NULL AUTO_INCREMENT,
    name      VARCHAR(50),
    subject   VARCHAR(20),
    score     INT,
    attendance INT,
    PRIMARY KEY (id)
);
```

---

# 5️⃣ 샘플 데이터 삽입

```sql
INSERT INTO StudentScore (name, subject, score, attendance) VALUES
('김철수', '국어', 95, 98),
('김철수', '영어', 67, 98),
('김철수', '수학', 82, 98),

('이영희', '국어', 58, 75),
('이영희', '영어', 88, 75),
('이영희', '수학', 73, 75),

('박민수', '국어', 40, 60),
('박민수', '영어', 55, 60),
('박민수', '수학', 61, 60),

('최가영', '국어', 77, 92),
('최가영', '영어', 83, 92),
('최가영', '수학', 91, 92);
```

---

# 6️⃣ HAVING 실습 예제

---

## ✔ 예제 1) 과목별 평균 점수가 70 이상인 과목

```sql
SELECT subject, AVG(score) AS avg_score
FROM StudentScore
GROUP BY subject
HAVING AVG(score) >= 70;
```

### 결과

| subject | avg_score |
| ------- | --------- |
| 영어      | 73.25     |
| 수학      | 76.75     |

---

## ✔ 예제 2) 학생별 총점이 200점 이상인 학생

```sql
SELECT name, SUM(score) AS total_score
FROM StudentScore
GROUP BY name
HAVING SUM(score) >= 200;
```

### 결과

| name | total_score |
| ---- | ----------- |
| 김철수  | 244         |
| 이영희  | 219         |
| 최가영  | 251         |

---

## ✔ 예제 3) 학생별 최고 점수가 90 이상

```sql
SELECT name, MAX(score) AS max_score
FROM StudentScore
GROUP BY name
HAVING MAX(score) >= 90;
```

### 결과

| name | max_score |
| ---- | --------- |
| 김철수  | 95        |
| 최가영  | 91        |

---

## ✔ 예제 4) 과목별 70점 미만 학생이 2명 이상인 과목

```sql
SELECT subject,
       SUM(CASE WHEN score < 70 THEN 1 ELSE 0 END) AS under70
FROM StudentScore
GROUP BY subject
HAVING SUM(CASE WHEN score < 70 THEN 1 ELSE 0 END) >= 2;
```

### 결과

| subject | under70 |
| ------- | ------- |
| 국어      | 2       |

---

## ✔ 예제 5) 학생별 출석률 평균이 80% 이상

```sql
SELECT name, AVG(attendance) AS avg_att
FROM StudentScore
GROUP BY name
HAVING AVG(attendance) >= 80;
```

### 결과

| name | avg_att |
| ---- | ------- |
| 김철수  | 98      |
| 최가영  | 92      |

---

## ✔ 예제 6) 과목별 최고 점수와 최저 점수 차이가 25 이상인 과목

```sql
SELECT subject,
       MAX(score) - MIN(score) AS diff
FROM StudentScore
GROUP BY subject
HAVING diff >= 25;
```

### 결과

| subject | diff |
| ------- | ---- |
| 국어      | 55   |
| 영어      | 33   |

---

## ✔ 예제 7) 학생별 평균 점수가 전체 평균보다 높은 학생

먼저 전체 평균:

```sql
SELECT AVG(score) FROM StudentScore;
```

결과: **AVG(score) = 72.44**

이제 HAVING으로 비교:

```sql
SELECT name, AVG(score) AS avg_score
FROM StudentScore
GROUP BY name
HAVING AVG(score) > (SELECT AVG(score) FROM StudentScore);
```

### 결과

| name | avg_score |
| ---- | --------- |
| 김철수  | 81.33     |
| 최가영  | 83.66     |

---

## ✔ 예제 8) 3개 과목 모두 60점 이상인 학생

```sql
SELECT name,
       MIN(score) AS min_score
FROM StudentScore
GROUP BY name
HAVING MIN(score) >= 60;
```

### 결과

| name | min_score |
| ---- | --------- |
| 김철수  | 67        |
| 이영희  | 58 ❌ 불합격  |

최종 결과:

| name | min_score |
| ---- | --------- |
| 김철수  | 67        |
| 최가영  | 77        |

---

## ✔ 예제 9) 과목별 학생 수가 4명 미만인 과목

```sql
SELECT subject, COUNT(*) AS cnt
FROM StudentScore
GROUP BY subject
HAVING COUNT(*) < 4;
```

### 결과

없음 (모두 4명)

---

## ✔ 예제 10) 평균 점수가 75~85 사이인 과목

```sql
SELECT subject, AVG(score) AS avg_score
FROM StudentScore
GROUP BY subject
HAVING AVG(score) BETWEEN 75 AND 85;
```

### 결과

| subject | avg_score |
| ------- | --------- |
| 수학      | 76.75     |

---

## ✔ 예제 11) 출석률이 70 미만인 학생 수가 1명 이상인 과목

```sql
SELECT subject,
       SUM(CASE WHEN attendance < 70 THEN 1 ELSE 0 END) AS low_att
FROM StudentScore
GROUP BY subject
HAVING low_att >= 1;
```

### 결과

| subject | low_att |
| ------- | ------- |
| 국어      | 1       |
| 영어      | 1       |
| 수학      | 1       |

---

## ✔ 예제 12) 학생별 점수 합계가 가장 높은 학생 1명만 조회

(HAVING + 서브쿼리)

```sql
SELECT name, SUM(score) AS total_score
FROM StudentScore
GROUP BY name
HAVING SUM(score) = (
    SELECT MAX(t.total)
    FROM (
        SELECT SUM(score) AS total
        FROM StudentScore
        GROUP BY name
    ) t
);
```

### 결과

| name | total_score |
| ---- | ----------- |
| 최가영  | 251         |

---

## ✔ 예제 13) 과목별 점수 평균을 기준으로 상/중/하 분류 (HAVING 활용)

```sql
SELECT
    subject,
    AVG(score) AS avg_score,
    CASE
        WHEN AVG(score) >= 80 THEN '상'
        WHEN AVG(score) >= 70 THEN '중'
        ELSE '하'
    END AS level
FROM StudentScore
GROUP BY subject
HAVING AVG(score) >= 60;
```

---

## ✔ 예제 14) ROLLUP + HAVING으로 전체 합계 제외

```sql
SELECT subject, SUM(score) AS total_score
FROM StudentScore
GROUP BY subject WITH ROLLUP
HAVING subject IS NOT NULL;
```

---

## ✔ 예제 15) HAVING으로 데이터 품질 검증

(user가 과목을 3개 들었는지 점검)

```sql
SELECT name, COUNT(*) AS cnt
FROM StudentScore
GROUP BY name
HAVING COUNT(*) <> 3;
```

### 결과

없음 → 모두 3개 과목 이수

---

# 7️⃣ HAVING 전문가 팁

---

### ✔ 1) HAVING에는 집계 함수가 반드시 필요하진 않다

MySQL에서는 아래도 허용됨:

```sql
HAVING subject = '국어';
```

하지만 권장되지 않음 → WHERE로 처리하는 것이 맞다.

---

### ✔ 2) WHERE를 먼저 쓰고 HAVING은 최소화해야 성능이 좋다

WHERE → GROUP BY → HAVING
순서 때문에 HAVING은 항상 더 느림.

---

### ✔ 3) HAVING은 GROUP BY 없이도 단독으로 사용 가능

(그러나 거의 사용하지 않음)

```sql
SELECT score
FROM StudentScore
HAVING score > 80;
```

---

### ✔ 4) HAVING은 “집계 이후 필터링”이란 원칙만 확실히 기억하면 오류가 없음

---
