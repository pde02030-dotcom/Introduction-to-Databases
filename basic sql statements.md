# 📌 Create
```sql
CREATE TABLE student (
    student_id   INT NOT NULL AUTO_INCREMENT,
    name         VARCHAR(50) NOT NULL,
    email        VARCHAR(100) UNIQUE,
    phone        VARCHAR(20),
    reg_date     TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (student_id)
);
```

# 📌 Insert
```sql
INSERT INTO student (name, email, phone) VALUES
('홍길동', 'hong1@example.com', '010-1111-1111'),
('김철수', 'kimcs@example.com', '010-2222-2222'),
('이영희', 'leeyh@example.com', '010-3333-3333'),
('박민수', 'parkms@example.com', '010-4444-4444'),
('최지현', 'choijh@example.com', '010-5555-5555'),
('정수빈', 'jungsb@example.com', '010-6666-6666'),
('한지우', 'hanjw@example.com', '010-7777-7777'),
('신예림', 'shinyerim@example.com', '010-8888-8888'),
('오도현', 'odohyun@example.com', '010-9999-9999'),
('서성원', 'seosw@example.com', '010-0000-0000'),
('윤아람', 'yoonar@example.com', '010-1212-3434'),
('강다혜', 'kangdh@example.com', '010-5656-7878'),
('문태현', 'moonhy@example.com', '010-9898-7676'),
('배지호', 'baejiho@example.com', '010-4545-9999'),
('조은별', 'joeunbyul@example.com', '010-2323-4545');
```


# 📌 1️⃣ SELECT 기본 조회

```sql
-- 모든 학생 조회
SELECT * FROM student;
```

```sql
-- 이름만 조회
SELECT name FROM student;
```

```sql
-- 이름, 이메일, 등록일 조회
SELECT name, email, reg_date FROM student;
```

---

# 📌 2️⃣ WHERE 조건 조회

```sql
-- 이름이 '홍길동'인 학생 조회
SELECT * FROM student
WHERE name = '홍길동';
```

```sql
-- 전화번호가 '010-4444-4444'인 학생
SELECT * FROM student
WHERE phone = '010-4444-4444';
```

```sql
-- 이메일 도메인이 example.com 인 학생
SELECT * FROM student
WHERE email LIKE '%@example.com';
```

---

# 📌 3️⃣ LIKE 검색

```sql
-- '김'으로 시작하는 학생
SELECT * FROM student
WHERE name LIKE '김%';
```

```sql
-- 전화번호에 '23'이 포함된 학생
SELECT * FROM student
WHERE phone LIKE '%23%';
```

---

# 📌 4️⃣ ORDER BY 정렬

```sql
-- 이름 오름차순 정렬
SELECT * FROM student
ORDER BY name ASC;
```

```sql
-- 최근 등록한 순서 (reg_date 최신순)
SELECT * FROM student
ORDER BY reg_date DESC;
```

---

# 📌 5️⃣ LIMIT (페이징)

```sql
-- 상위 5명 조회
SELECT * FROM student
ORDER BY student_id
LIMIT 5;
```

```sql
-- 6번째~10번째 학생 조회 (페이징)
SELECT * FROM student
ORDER BY student_id
LIMIT 5 OFFSET 5;
```

---

# 📌 6️⃣ UPDATE 기본

```sql
-- 특정 학생 전화번호 수정
UPDATE student
SET phone = '010-1234-5678'
WHERE name = '홍길동';
```

```sql
-- 이메일 변경
UPDATE student
SET email = 'hong_new@example.com'
WHERE student_id = 1;
```

---

# 📌 7️⃣ UPDATE + 조건

```sql
-- 전화번호가 없는 학생에게 기본 번호 부여
UPDATE student
SET phone = '010-0000-0000'
WHERE phone IS NULL;
```

---

# 📌 8️⃣ DELETE

```sql
-- 특정 학생 삭제
DELETE FROM student
WHERE name = '배지호';
```

```sql
-- 이메일 도메인이 example.com이 아닌 학생 삭제
DELETE FROM student
WHERE email NOT LIKE '%@example.com';
```

---

# 📌 9️⃣ IN / NOT IN

```sql
-- 이름이 특정 목록에 있는 학생
SELECT * FROM student
WHERE name IN ('홍길동', '김철수', '이영희');
```

```sql
-- 특정 학생 제외
SELECT * FROM student
WHERE name NOT IN ('홍길동', '김철수');
```

---

# 📌 🔟 BETWEEN

```sql
-- student_id가 3~7 범위인 학생
SELECT * FROM student
WHERE student_id BETWEEN 3 AND 7;
```

---

# 📌 1️⃣1️⃣ COUNT / GROUP BY 기본 집계

현재 student 테이블은 집계할 컬럼이 적지만, 기본 예제로 활용 가능합니다.

```sql
-- 전체 학생 수
SELECT COUNT(*) AS total_student
FROM student;
```

```sql
-- 이메일 도메인별 학생 수
SELECT 
    SUBSTRING_INDEX(email, '@', -1) AS domain,
    COUNT(*) AS domain_count
FROM student
GROUP BY domain;
```




