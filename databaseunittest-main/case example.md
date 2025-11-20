# 🧪 **CASE 실습을 위한 예제 세트**

---

# 1️⃣ 실습용 테이블 생성

학생들의 성적/상태/출석 정보 등을 담은 테이블을 만듭니다.

```sql
CREATE TABLE StudentScore (
    id        INT NOT NULL AUTO_INCREMENT,
    name      VARCHAR(50) NOT NULL,
    subject   VARCHAR(20) NOT NULL,
    score     INT,
    attendance INT,              -- 출석률(%)
    status    VARCHAR(20),       -- 상태: active / leave / dropout
    PRIMARY KEY (id)
);
```

---

# 2️⃣ 샘플 데이터 삽입

```sql
INSERT INTO StudentScore (name, subject, score, attendance, status) VALUES
('김철수', '국어', 95, 98, 'active'),
('김철수', '영어', 67, 98, 'active'),
('김철수', '수학', 82, 98, 'active'),

('이영희', '국어', 58, 75, 'leave'),
('이영희', '영어', 88, 75, 'leave'),
('이영희', '수학', 73, 75, 'leave'),

('박민수', '국어', 40, 60, 'dropout'),
('박민수', '영어', 55, 60, 'dropout'),
('박민수', '수학', 61, 60, 'dropout'),

('최가영', '국어', 77, 92, 'active'),
('최가영', '영어', 83, 92, 'active'),
('최가영', '수학', 91, 92, 'active');
```

---

# 3️⃣ CASE 실습 예제 문제 + 해설 (총 10개)

---

## ✔ **1. 점수에 따라 합격/불합격 표시**

```sql
SELECT
    name,
    subject,
    score,
    CASE
        WHEN score >= 60 THEN '합격'
        ELSE '불합격'
    END AS pass_status
FROM StudentScore;
```

---

## ✔ **2. 점수 구간에 따라 등급(A~F) 부여**

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
FROM StudentScore;
```

---

## ✔ **3. 출석률로 상태 표시 (90% 이상: 우수 / 70% 이상: 정상 / 그 외: 경고)**

```sql
SELECT
    name,
    attendance,
    CASE
        WHEN attendance >= 90 THEN '우수'
        WHEN attendance >= 70 THEN '정상'
        ELSE '경고'
    END AS attendance_level
FROM StudentScore
GROUP BY name;
```

> 동일 학생이 여러 과목이 있으므로 `GROUP BY name`을 통해 학생별로 하나씩 보고 싶을 때 사용

---

## ✔ **4. 학생 상태(status 컬럼)를 한국어로 변환**

```sql
SELECT
    name,
    status,
    CASE status
        WHEN 'active' THEN '재학중'
        WHEN 'leave'  THEN '휴학'
        WHEN 'dropout' THEN '제적'
        ELSE '미정'
    END AS status_kor
FROM StudentScore
GROUP BY name;
```

---

## ✔ **5. 특정 과목(수학)에 한해 추가 규칙 적용 → 중첩 CASE**

“수학에서 90점 이상이면 S 등급”이라는 특별 규칙을 추가합니다.

```sql
SELECT
    name,
    subject,
    score,
    CASE 
        WHEN subject = '수학' THEN
            CASE
                WHEN score >= 90 THEN 'S'
                WHEN score >= 80 THEN 'A'
                WHEN score >= 70 THEN 'B'
                WHEN score >= 60 THEN 'C'
                ELSE 'D'
            END
        ELSE
            CASE
                WHEN score >= 90 THEN 'A'
                WHEN score >= 80 THEN 'B'
                WHEN score >= 70 THEN 'C'
                WHEN score >= 60 THEN 'D'
                ELSE 'F'
            END
    END AS grade
FROM StudentScore;
```

---

## ✔ **6. 출석률과 점수를 함께 고려한 위험 학생 탐지**

출석률 < 70 OR 점수 < 50 → 위험학생

```sql
SELECT
    name,
    subject,
    score,
    attendance,
    CASE
        WHEN attendance < 70 OR score < 50 THEN '위험학생'
        ELSE '정상'
    END AS risk_level
FROM StudentScore;
```

---

## ✔ **7. ORDER BY에서 CASE 사용 — 제적(dropout) 학생은 맨 뒤로 정렬**

```sql
SELECT 
    name, subject, status
FROM StudentScore
ORDER BY 
    CASE 
        WHEN status = 'dropout' THEN 1 
        ELSE 0 
    END,
    name ASC;
```

---

## ✔ **8. GROUP BY + CASE 로 등급별 학생 수**

```sql
SELECT
    CASE
        WHEN score >= 90 THEN 'A'
        WHEN score >= 80 THEN 'B'
        WHEN score >= 70 THEN 'C'
        WHEN score >= 60 THEN 'D'
        ELSE 'F'
    END AS grade,
    COUNT(*) AS cnt
FROM StudentScore
GROUP BY grade
ORDER BY grade;
```

---

## ✔ **9. NULL 값 처리 (트리키한 실전 패턴)**

먼저 하나의 NULL 데이터 삽입:

```sql
INSERT INTO StudentScore (name, subject, score, attendance, status)
VALUES ('테스트', '국어', NULL, 80, 'active');
```

CASE로 NULL 스코어 처리:

```sql
SELECT
    name,
    subject,
    score,
    CASE 
        WHEN score IS NULL THEN '미입력'
        WHEN score >= 60 THEN '합격'
        ELSE '불합격'
    END AS pass_status
FROM StudentScore
WHERE name = '테스트';
```

---

## ✔ **10. UPDATE 문에 CASE 사용 — 조건에 따라 일괄 점수 보정**

합격자(score ≥ 60)는 +3점, 불합격자는 -1점 조정

```sql
UPDATE StudentScore
SET score = score +
    CASE
        WHEN score >= 60 THEN 3
        ELSE -1
    END;
```

조회로 결과 확인:

```sql
SELECT name, subject, score FROM StudentScore;
```

