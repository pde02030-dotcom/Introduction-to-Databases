다중 조인의, **논리적인 실행 순서**는 이렇게 됩니다:

> **FROM + JOIN(들)** → (WHERE) → (GROUP BY) → (HAVING) → SELECT → (ORDER BY) → (LIMIT)



---

## 0. 다중 조인 쿼리 예시시 

```sql
SELECT
    s.name      AS student_name,
    c.title     AS course_title,
    ei.exam_name,
    sc.score
FROM Student s
JOIN Score sc      ON s.student_id = sc.student_id
JOIN ExamInfo ei   ON sc.exam_id   = ei.exam_id
JOIN Course c      ON ei.course_id = c.course_id;
```

등장하는 테이블과 별칭:

- `Student s` : 학생 정보
- `Score sc` : 학생별 시험 점수 (학생 + 시험 사이의 매핑)
- `ExamInfo ei` : 시험(과목) 정보
- `Course c` : 강좌(과정) 정보

---

## 1. 논리적인 실행 순서(교과서 관점)

SQL 표준 관점에서 **논리적**인 처리 순서는 아래처럼 생각하면 됩니다.

1. `FROM Student s`
2. `JOIN Score sc ON s.student_id = sc.student_id`
3. `JOIN ExamInfo ei ON sc.exam_id = ei.exam_id`
4. `JOIN Course c ON ei.course_id = c.course_id`
5. `SELECT ...` 로 필요한 컬럼만 뽑고 별칭 적용

각 단계에서 **중간 결과(Result Set)** 를 계속 만들어 나간다고 생각하시면 편합니다.

---

## 2. 단계별로 “중간 테이블”이 어떻게 생기는지

### (1) FROM Student s

```sql
FROM Student s
```

- DB는 먼저 `Student` 테이블을 읽습니다.
- 이 시점에서 **중간 결과**는 Student의 전체 행 집합입니다.

예를 들어 Student에 데이터가 이렇게 있다고 가정:

| student_id | name     | ... |
|-----------|----------|-----|
| 1         | 김철수   | ... |
| 2         | 이영희   | ... |
| 3         | 박민수   | ... |

→ 현재 중간 결과 R₁ = Student 의 모든 행

---

### (2) Student 와 Score를 JOIN

```sql
FROM Student s
JOIN Score sc ON s.student_id = sc.student_id
```

**논리적 의미**:

- `Student`의 각 행에 대해  
  `Score` 테이블에서 `sc.student_id = s.student_id` 가 같은 행만 붙입니다.
- 둘 다 매칭되는(=조인 조건을 만족하는) 행만 남는 **INNER JOIN** 입니다.

예시로 Score 테이블이 다음과 같다고 해볼게요:

| score_id | student_id | exam_id | score |
|----------|------------|---------|-------|
| 10       | 1          | 100     | 90    |
| 11       | 1          | 101     | 85    |
| 12       | 2          | 100     | 70    |

조인 결과 R₂ (Student ⨝ Score):

| s.student_id | s.name | sc.score_id | sc.exam_id | sc.score |
|--------------|--------|-------------|------------|----------|
| 1            | 김철수 | 10          | 100        | 90       |
| 1            | 김철수 | 11          | 101        | 85       |
| 2            | 이영희 | 12          | 100        | 70       |

- 김철수는 시험을 2개 봤으니 2행으로 “펼쳐진” 것  
- Score에 없는 학생(예: 박민수, student_id=3)은 조인 결과에 나타나지 않습니다. (INNER JOIN)

---

### (3) R₂ 와 ExamInfo 를 JOIN

```sql
JOIN ExamInfo ei ON sc.exam_id = ei.exam_id
```

- 방금 만든 중간 결과 R₂(학생+점수) 에  
- `ExamInfo` 테이블을 **시험 ID(exam_id)** 로 조인합니다.

ExamInfo 예시:

| exam_id | exam_name     | course_id |
|---------|---------------|-----------|
| 100     | 중간고사      | 1000      |
| 101     | 기말고사      | 1000      |

조인 결과 R₃ (R₂ ⨝ ExamInfo):

| s.student_id | s.name | sc.score | sc.exam_id | ei.exam_name | ei.course_id |
|--------------|--------|----------|------------|--------------|--------------|
| 1            | 김철수 | 90       | 100        | 중간고사     | 1000         |
| 1            | 김철수 | 85       | 101        | 기말고사     | 1000         |
| 2            | 이영희 | 70       | 100        | 중간고사     | 1000         |

이제 한 행(row)이 “학생 + 시험 + 점수” 정보를 모두 가지고 있게 됩니다.

---

### (4) R₃ 와 Course 를 JOIN

```sql
JOIN Course c ON ei.course_id = c.course_id
```

- ExamInfo 에 있는 `course_id` 를 이용해서
- 해당 시험이 속한 `Course` 를 붙입니다.

Course 예시:

| course_id | title           |
|-----------|-----------------|
| 1000      | 데이터베이스    |
| 2000      | 운영체제        |

조인 결과 R₄ (R₃ ⨝ Course):

| s.student_id | s.name | c.title        | ei.exam_name | sc.score |
|--------------|--------|----------------|--------------|---------|
| 1            | 김철수 | 데이터베이스   | 중간고사     | 90      |
| 1            | 김철수 | 데이터베이스   | 기말고사     | 85      |
| 2            | 이영희 | 데이터베이스   | 중간고사     | 70      |

이제 각 행이  
**“어느 학생이 어떤 과정(Course)의 어떤 시험을 몇 점(score) 받았는지”**  
완전히 표현하는 상태가 됩니다.

---

### (5) SELECT 절로 최종 컬럼 선택 & 별칭 적용

```sql
SELECT
    s.name      AS student_name,
    c.title     AS course_title,
    ei.exam_name,
    sc.score
```

- 지금까지의 중간 결과 R₄에서  
  필요한 컬럼만 골라서 **투영(projection)** 하는 단계입니다.
- 동시에 `AS student_name`, `AS course_title` 같은 **컬럼 별칭(alias)** 도 이 단계에서 적용됩니다.

최종 Result Set:

| student_name | course_title   | exam_name | score |
|--------------|----------------|-----------|-------|
| 김철수       | 데이터베이스   | 중간고사  | 90    |
| 김철수       | 데이터베이스   | 기말고사  | 85    |
| 이영희       | 데이터베이스   | 중간고사  | 70    |

이게 클라이언트(JDBC, 콘솔, DBeaver, Workbench 등)로 넘어가는 **SELECT 쿼리의 출력 결과(Result Set)** 입니다.

---

## 3. “진짜” 실행 순서: 옵티마이저 관점 한마디

위에서 설명한 건 **논리적인 순서(개념적)** 입니다.  
실제 DB 내부에서는 **옵티마이저(Optimizer)**가 다음을 고려해서 실행 순서를 바꿀 수도 있습니다.

- 어느 테이블이 더 작은지 (먼저 읽으면 좋음)
- 인덱스가 어디에 있는지
- 조인 순서를 바꿔도 결과가 바뀌지 않는지
- 카디널리티(각 조건에서 줄어드는 행 수)

그래서 실제로는:

- Score를 먼저 읽고 → Student 조인
- 혹은 ExamInfo, Course 순서를 바꿔서 조인

같은 방식으로 **물리적 실행 순서를 재조합**합니다.  
하지만 우리는 **논리적 관점**에서

> FROM → JOIN → JOIN → JOIN → SELECT

이렇게 이해하시는 게 가장 좋습니다.  
EXPLAIN으로 보면 그 “물리적 실행 순서”가 어떻게 바뀌는지 확인할 수 있고요.

---

## 4. 요약

- 쿼리의 **논리적 처리 순서**
  1. `FROM Student s`
  2. `JOIN Score sc ON ...`
  3. `JOIN ExamInfo ei ON ...`
  4. `JOIN Course c ON ...`
  5. `SELECT ...` (별칭 적용, 최종 컬럼만 선택)

- 각 단계에서 “중간 테이블”이 계속 확장되면서  
  마지막에 “학생 + 과정 + 시험 + 점수” 데이터를 한 줄에 하나씩 담은 Result Set이 만들어진다.

