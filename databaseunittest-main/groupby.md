# 🚀 SQL GROUP BY

---

# 1️⃣ GROUP BY란?

GROUP BY는 **레코드(row)를 특정 기준으로 묶은 뒤, 집계 함수를 적용하여 요약된 결과를 반환**하는 구문이다.

예:

```
과목별 평균 점수  
과목별 인원수  
학생별 총점  
로그 레벨별 카운트  
날짜별 접속자 수
```

---

# 2️⃣ GROUP BY + 집계 함수(Aggregate Function)

GROUP BY 는 반드시 집계 함수와 함께 사용된다.

✔ COUNT()
✔ SUM()
✔ AVG()
✔ MAX()
✔ MIN()

---

# 3️⃣ GROUP BY의 중요한 규칙 (매우 중요)

### ✔ SELECT 절에 나오는 컬럼은 아래 둘 중 하나여야 한다

1. GROUP BY 절에 포함된 컬럼
2. 집계 함수로 감싸진 컬럼

아래는 오류 예시:

```sql
SELECT subject, name, AVG(score)
FROM Score
GROUP BY subject;
```

→ subject는 그룹 기준이지만 name은 그룹 기준이 아니므로 오류 발생.

---

# 4️⃣ WHERE → GROUP BY → HAVING → ORDER BY 실행 순서

SQL 실행 순서는 다음과 같다:

1. **FROM**
2. **WHERE**
3. **GROUP BY**
4. **HAVING**
5. **SELECT**
6. **ORDER BY**

따라서 HAVING은 **집계된 이후** 조건을 걸 때 사용한다.

---

# 5️⃣ 실습용 테이블 생성

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

# 6️⃣ 샘플 데이터 삽입

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

# 7️⃣ GROUP BY 실습 예제

---

## ✔ 예제 1) 과목별 평균 점수

```sql
SELECT subject, AVG(score) AS avg_score
FROM StudentScore
GROUP BY subject;
```

### 결과

| subject | avg_score               |
| ------- | ----------------------- |
| 국어      | (95+58+40+77)/4 = 67.5  |
| 영어      | (67+88+55+83)/4 = 73.25 |
| 수학      | (82+73+61+91)/4 = 76.75 |

---

## ✔ 예제 2) 학생별 총점

```sql
SELECT name, SUM(score) AS total_score
FROM StudentScore
GROUP BY name;
```

### 결과

| name | total_score |
| ---- | ----------- |
| 김철수  | 244         |
| 이영희  | 219         |
| 박민수  | 156         |
| 최가영  | 251         |

---

## ✔ 예제 3) 학생별 평균 점수

```sql
SELECT name, AVG(score) AS avg_score
FROM StudentScore
GROUP BY name;
```

---

## ✔ 예제 4) 과목별 최고 점수

```sql
SELECT subject, MAX(score) AS max_score
FROM StudentScore
GROUP BY subject;
```

### 결과

| subject | max_score |
| ------- | --------- |
| 국어      | 95        |
| 영어      | 88        |
| 수학      | 91        |

---

## ✔ 예제 5) 출석률(attendance) 구간별 인원 수

(출석률을 구간으로 CASE 처리하여 GROUP BY)

```sql
SELECT 
    CASE 
        WHEN attendance >= 90 THEN '우수'
        WHEN attendance >= 70 THEN '보통'
        ELSE '부족'
    END AS attendance_level,
    COUNT(*) AS cnt
FROM StudentScore
GROUP BY attendance_level;
```

### 결과

| attendance_level | cnt |
| ---------------- | --- |
| 우수               | 3   |
| 보통               | 3   |
| 부족               | 3   |

---

## ✔ 예제 6) 과목별 학생 수

```sql
SELECT subject, COUNT(*) AS cnt
FROM StudentScore
GROUP BY subject;
```

---

## ✔ 예제 7) 이름 + 과목 그룹핑은 허용되나 “이름만” 조회는 불가

```sql
SELECT name, subject, AVG(score)
FROM StudentScore
GROUP BY name, subject;
```

### (정상)

---

## ✔ 예제 8) 평균 점수가 80 이상인 과목만 조회 (HAVING)

```sql
SELECT subject, AVG(score) AS avg_score
FROM StudentScore
GROUP BY subject
HAVING AVG(score) >= 70;
```

> HAVING 은 집계된 결과에 조건을 거는 절이다.

---

## ✔ 예제 9) 학생별 최고 점수가 90 이상인 학생

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

## ✔ 예제 10) 과목별 점수 합계 내림차순 정렬

```sql
SELECT subject, SUM(score) AS total
FROM StudentScore
GROUP BY subject
ORDER BY total DESC;
```

---

## ✔ 예제 11) 출석률 평균이 가장 높은 과목

```sql
SELECT subject, AVG(attendance) AS avg_att
FROM StudentScore
GROUP BY subject
ORDER BY avg_att DESC;
```

---

## ✔ 예제 12) 학생별 “점수 편차(최고-최저)” 계산

```sql
SELECT
    name,
    MAX(score) - MIN(score) AS diff
FROM StudentScore
GROUP BY name;
```

### 결과

| name | diff |
| ---- | ---- |
| 김철수  | 28   |
| 이영희  | 30   |
| 박민수  | 21   |
| 최가영  | 14   |

---

## ✔ 예제 13) 과목별 점수 분포(70 이상 / 미만)

```sql
SELECT
    subject,
    SUM(CASE WHEN score >= 70 THEN 1 ELSE 0 END) AS above_70,
    SUM(CASE WHEN score < 70 THEN 1 ELSE 0 END) AS below_70
FROM StudentScore
GROUP BY subject;
```

---

## ✔ 예제 14) 학생별 전체 데이터 건수와 평균 출석률

```sql
SELECT
    name,
    COUNT(*) AS cnt,
    AVG(attendance) AS avg_att
FROM StudentScore
GROUP BY name;
```

---

## ✔ 예제 15) GROUP BY 확장 문법 ROLLUP 예제

(총합 표시)

```sql
SELECT subject, SUM(score) AS total_score
FROM StudentScore
GROUP BY subject WITH ROLLUP;
```

### 결과

| subject | total_score |
| ------- | ----------- |
| 국어      | 270         |
| 영어      | 293         |
| 수학      | 307         |
| NULL    | 870 (전체 합)  |

ROLLUP 은 마지막 행에 전체 합계를 자동 생성한다.

---

# 8️⃣ 전문가용 GROUP BY 성능 팁

---

### ✔ 1) WHERE 절은 그룹핑 이전에 실행된다

→ WHERE 절에서는 집계 함수 사용 불가능

---

### ✔ 2) HAVING은 반드시 “집계된 후 필터링”

→ 성능 위해 WHERE 우선, HAVING 최소화

---

### ✔ 3) GROUP BY 는 기본적으로 정렬(Sort) 작업 필요

→ 대량 데이터에서는 CPU 사용량 증가
→ 필요한 컬럼에 인덱스 고려해야 함

---

### ✔ 4) SELECT 절의 컬럼 규칙은 매우 중요

GROUP BY에 없는 컬럼은 반드시 **집계 함수로 감싸야 한다**

---

### ✔ 5) GROUPING SETS / ROLLUP / CUBE 는 BI/통계용으로 강력

---
