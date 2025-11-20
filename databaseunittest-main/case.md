# 🎯 **SQL CASE**

SQL을 다루다 보면 “조건에 따라 값을 다르게 처리”해야 하는 상황이 반드시 등장합니다.
이때 **CASE 표현식(CASE Expression)** 은 SQL에서 제공하는 **가장 강력한 조건 로직**입니다.

다음 글에서는 다음을 깊이 있게 다룹니다:

* CASE 문 구조(검색형 / 단순형)
* NULL 처리 전략
* GROUP BY, ORDER BY에서의 활용
* JOIN 조건에서 사용하는 고급 패턴
* 트랜잭션과 함께 사용 시 주의점
* 실제 테이블 생성 + 데이터 삽입 후 실습 가능한 예제

---

# 1️⃣ CASE 문은 무엇인가?

CASE 는 SQL의 조건 분기문(condition expression)으로,
프로그래밍 언어의 `if-else`와 거의 동일한 역할을 수행합니다.

✔ **결과는 단 하나의 값이 된다.**
✔ SELECT, WHERE, GROUP BY, ORDER BY 어디든 사용 가능하다.
✔ 복잡한 로직을 SQL 한 줄로 처리할 수 있어 성능적·구조적으로 매우 유리하다.

---

# 2️⃣ CASE 문 구조

## 🔹 2-1. 단순형(SIMPLE CASE)

```sql
CASE 컬럼
    WHEN 값1 THEN 결과1
    WHEN 값2 THEN 결과2
    ELSE 기본결과
END
```

📌 특정 컬럼의 값이 정해진 범주일 때 사용
예: 점수 → 등급 매핑, 상태코드 → 한글 변환 등.

---

## 🔹 2-2. 검색형(SEARCHED CASE)

```sql
CASE 
    WHEN 조건식1 THEN 결과1
    WHEN 조건식2 THEN 결과2
    ELSE 기본결과
END
```

📌 더 많이 사용하는 방식
📌 범위 조건(≥, ≤) 또는 복합 조건이 필요한 경우 필수

---

# 3️⃣ 실습용 테이블 생성

아래는 **CASE 문 실습을 위한 테이블**입니다.

## 📌 3-1. 학생 성적 테이블 생성

```sql
CREATE TABLE Score (
    id        INT PRIMARY KEY AUTO_INCREMENT,
    name      VARCHAR(50),
    subject   VARCHAR(20),
    score     INT
);
```

## 📌 3-2. 샘플 데이터 삽입

```sql
INSERT INTO Score (name, subject, score) VALUES
('김철수', '국어', 95),
('김철수', '영어', 67),
('김철수', '수학', 82),
('이영희', '국어', 58),
('이영희', '영어', 88),
('이영희', '수학', 73),
('박민수', '국어', 40),
('박민수', '영어', 55),
('박민수', '수학', 61);
```

---

# 4️⃣ 단순형 CASE 실전 예제

## ✔ 점수 → 합격/불합격

```sql
SELECT 
    name,
    subject,
    score,
    CASE
        WHEN score >= 60 THEN '합격'
        ELSE '불합격'
    END AS pass_status
FROM Score;
```

---

# 5️⃣ 검색형 CASE – 전문가들이 자주 사용하는 패턴

## ✔ 점수 → 등급 분류

```sql
SELECT 
    name,
    subject,
    score,
    CASE
        WHEN score >= 90 THEN 'A'
        WHEN score >= 80 THEN 'B'
        WHEN score >= 70 THEN 'C'
        WHEN score >= 60 THEN 'D'
        ELSE 'F'
    END AS grade
FROM Score;
```

📌 조건이 위에서 아래로 평가되므로 **조건 순서가 매우 중요**합니다.

---

# 6️⃣ GROUP BY와 함께 사용하기

CASE는 종종 집계와 함께 쓰이며, 통계 리포트에서 필수 기술입니다.

## ✔ “과목별 등급별 인원수 집계”

```sql
SELECT 
    subject,
    CASE
        WHEN score >= 90 THEN 'A'
        WHEN score >= 80 THEN 'B'
        WHEN score >= 70 THEN 'C'
        WHEN score >= 60 THEN 'D'
        ELSE 'F'
    END AS grade,
    COUNT(*) AS cnt
FROM Score
GROUP BY subject, grade;
```

---

# 7️⃣ ORDER BY에서 CASE 활용

정렬 기준을 조건에 따라 결정할 수 있습니다.

## ✔ 합격자 먼저, 불합격자는 뒤로 정렬

```sql
SELECT 
    name, subject, score
FROM Score
ORDER BY
    CASE WHEN score >= 60 THEN 0 ELSE 1 END,
    score DESC;
```

---

# 8️⃣ WHERE 절에서 CASE 사용 — 주의해야 할 점

CASE는 WHERE 안에서도 가능하지만, 조건 완전 제어가 필요할 때만 사용해야 합니다.

✔ 비추천 예 (느리고 가독성 ↓):

```sql
WHERE CASE WHEN score >= 60 THEN 1 ELSE 0 END = 1
```

✔ 추천 예 (보다 명확):

```sql
WHERE score >= 60
```

📌 WHERE에 CASE 쓰는 것은 **조건 분기 자체가 필요할 때만** 하시는 것이 좋습니다.

---

# 9️⃣ JOIN에서 CASE — 고급 실무 패턴

## ✔ 문자열 상태 코드 → 표준화된 값으로 JOIN

```sql
SELECT s.*, c.category_name
FROM Score s
LEFT JOIN Category c
    ON c.code = CASE 
                    WHEN s.subject IN ('국어', '영어', '수학') THEN 'LANG'
                    ELSE 'ETC'
                END;
```

📌 데이터 품질이 떨어지는 환경에서 많이 쓰는 실전 패턴입니다.

---

# 🔟 NULL 처리 전략과 CASE

CASE 는 NULL을 그대로 비교하면 **항상 false**입니다.

예:

```sql
CASE
    WHEN col = NULL THEN 'X'   -- 항상 false
```

✔ 올바른 방법

```sql
WHEN col IS NULL THEN 'NULL 값'
```

✔ COALESCE 와 함께 사용

```sql
CASE WHEN COALESCE(col, 0) = 0 THEN '미입력'
     ELSE '입력됨'
END
```

---

# 1️⃣1️⃣ 트랜잭션에서의 CASE 활용

CASE 는 **트랜잭션의 Atomicity(원자성)** 에 영향을 주지 않습니다.
다만 조건에 따라 서로 다른 데이터 조작을 할 때 자주 사용됩니다.

예: 합격자 점수만 +5점 보정

```sql
UPDATE Score
SET score = score + 
    CASE WHEN score >= 60 THEN 5 ELSE 0 END;
```

📌 전체가 하나의 UPDATE 로 처리되므로 **원자성(atomic)** 이 보장됩니다.

---

# 1️⃣2️⃣ 고급 패턴 — 복합 CASE 중첩

실무에서 자주 등장합니다.

```sql
SELECT 
    name,
    subject,
    CASE
        WHEN score >= 90 THEN 'A'
        WHEN score >= 80 THEN 
            CASE 
                WHEN subject = '수학' THEN 'B+'
                ELSE 'B'
            END
        ELSE 'C 이하'
    END AS grade
FROM Score;
```

---

# 1️⃣3️⃣ 현업에서 자주 하는 실수 Best 3

| 실수          | 설명                          | 해결              |
| ----------- | --------------------------- | --------------- |
| CASE 순서 실수  | 앞 조건이 이미 true인데 아래를 계산하는 오해 | 위→아래 순차적 평가 기억  |
| ELSE 생략     | 예상하지 못한 값이 NULL 처리됨         | 가능한 ELSE 반드시 넣기 |
| WHERE에서 과사용 | 불필요하게 복잡                    | WHERE는 단순하게 유지  |

