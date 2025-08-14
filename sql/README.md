### 1. Department (부서)

-   `id`: 부서 고유 ID (Primary Key)
-   `name`: 부서명
-   `reader_id`: 부서 책임자 ID (Foreign Key)

### 2. Employee (직원)

-   `id`: 직원 고유 ID (Primary Key)
-   `name`: 직원명
-   `age`: 나이
-   `gender`: 성별 (M/F)
-   `salary`: 급여
-   `dept_id`: 소속 부서 ID (Foreign Key)
-   `created_at`: 생성일

### 3. Project (프로젝트)

-   `id`: 프로젝트 고유 ID (Primary Key)
-   `name`: 프로젝트명
-   `leader_id`: 프로젝트 리더 ID (Foreign Key)
-   `start_date`: 시작일
-   `end_date`: 종료일

### 4. Works_on (프로젝트 참여)

-   `empl_id`: 직원 ID (Foreign Key)
-   `proj_id`: 프로젝트 ID (Foreign Key)
-   복합 Primary Key (empl_id, proj_id)

---

## DDL (Data Definition Language)

### 테이블 생성

```sql
-- 1. Department 테이블 생성
CREATE TABLE `department` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(100) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `leader_id` int DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=16 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci

-- 2. Employee 테이블 생성
CREATE TABLE `employee` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `age` int DEFAULT NULL,
  `gender` enum('M','F') CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `salary` decimal(10,2) NOT NULL,
  `dept_id` int DEFAULT NULL,
  `created_at` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `fk_employee_department` (`dept_id`),
  CONSTRAINT `fk_employee_department` FOREIGN KEY (`dept_id`) REFERENCES `department` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1048561 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci

-- 3. Project 테이블 생성
CREATE TABLE `project` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(100) COLLATE utf8mb4_unicode_ci NOT NULL,
  `leader_id` int DEFAULT NULL,
  `start_date` date NOT NULL,
  `end_date` date DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci

-- 4. Works_on 테이블 생성
CREATE TABLE `works_on` (
  `empl_id` int NOT NULL,
  `proj_id` int NOT NULL,
  PRIMARY KEY (`empl_id`,`proj_id`),
  KEY `fk_works_on_project` (`proj_id`),
  CONSTRAINT `fk_works_on_employee` FOREIGN KEY (`empl_id`) REFERENCES `employee` (`id`),
  CONSTRAINT `fk_works_on_project` FOREIGN KEY (`proj_id`) REFERENCES `project` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
```

### 인덱스 생성

```sql

```

### 외래키 제약조건 추가

```sql
# project
ALTER TABLE project ADD CONSTRAINT fk_project_employee FOREIGN KEY (leader_id) REFERENCES employee(id);
```

---

## DML (Data Manipulation Language)

### 더미데이터 삽입

```sql
# 높은 재귀 횟수를 허용하도록 설정
SET SESSION cte_max_recursion_depth = 2000000;
```

#### 1. Department 테이블 데이터 삽입

```sql
INSERT INTO department (name)
WITH RECURSIVE cte (n) AS
(
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte WHERE n < 10
)
SELECT
    CASE
        WHEN n % 10 = 1 THEN 'Engineering'
        WHEN n % 10 = 2 THEN 'Marketing'
        WHEN n % 10 = 3 THEN 'Sales'
        WHEN n % 10 = 4 THEN 'Finance'
        WHEN n % 10 = 5 THEN 'HR'
        WHEN n % 10 = 6 THEN 'Operations'
        WHEN n % 10 = 7 THEN 'IT'
        WHEN n % 10 = 8 THEN 'Customer Service'
        WHEN n % 10 = 9 THEN 'Research and Development'
        ELSE 'Product Management'
    END AS department
FROM cte;
```

#### 2. Employee 테이블 데이터 삽입

```sql
INSERT INTO employee (name, age, gender, salary, dept_id, created_at)
WITH RECURSIVE cte (n) AS
(
  SELECT 1
  UNION ALL
  SELECT n + 1 FROM cte WHERE n < 1000000
)
SELECT
	CONCAT('User', LPAD(n, 7, '0')) AS name,
    FLOOR(1 + (RAND() * 70)) AS age,
    CASE WHEN RAND() < 0.5 THEN 'M' ELSE 'F' END AS gender,
	FLOOR(100000 + (RAND() * 9900000)) AS salary,
	FLOOR(1 + (RAND() * 10)) AS dept_id,
    TIMESTAMP(DATE_SUB(NOW(), INTERVAL FLOOR(RAND() * 3650) DAY) + INTERVAL FLOOR(RAND() * 86400) SECOND) AS created_at
FROM cte;
```

#### 3. Project 테이블 데이터 삽입

```sql
INSERT INTO project (name, leader_id, start_date, end_date)
WITH RECURSIVE cte (n) AS (
    SELECT 1
    UNION ALL
    SELECT n + 1 FROM cte WHERE n < 2000
)
SELECT
    ELT(
        FLOOR(1 + (RAND() * 20)),
        '웹사이트 리뉴얼',
        '모바일 앱 개발',
        'AI 챗봇 구축',
        '데이터베이스 마이그레이션',
        '보안 시스템 강화',
        '고객 관리 시스템',
        '마케팅 자동화',
        '품질 관리 시스템',
        '인사 관리 시스템',
        '재무 분석 도구',
        '디자인 시스템 구축',
        'API 개발',
        '클라우드 마이그레이션',
        '모니터링 시스템',
        '사용자 인증 시스템',
        '결제 시스템 개발',
        '알림 시스템',
        '리포트 생성 도구',
        '데이터 시각화',
        '통합 관리 대시보드'
    ) AS name,
    FLOOR(1 + (RAND() * 1000000)) AS leader_id,
    DATE_SUB(CURDATE(), INTERVAL FLOOR(RAND() * 1825) DAY) AS start_date,
    CASE
        WHEN RAND() < 0.2 THEN NULL
        ELSE DATE_ADD(
            DATE_SUB(CURDATE(), INTERVAL FLOOR(RAND() * 1825) DAY),
            INTERVAL FLOOR(RAND() * 365) DAY
        )
    END AS end_date
FROM cte;

```

#### 4. Works_on 테이블 데이터 삽입

```sql
INSERT INTO works_on (empl_id, proj_id)
WITH RECURSIVE cte AS (
    SELECT 1 AS empl_id, 1 AS proj_id
    UNION ALL
    SELECT
        CASE WHEN proj_id < 2000 THEN empl_id ELSE empl_id + 1 END,
        CASE WHEN proj_id < 2000 THEN proj_id + 1 ELSE 1 END
    FROM cte
    WHERE (empl_id - 1) * 2000 + proj_id < 2000000
)
SELECT empl_id, proj_id
FROM cte;
```

---

## DCL (Data Control Language)

### 사용자 권한 관리

```sql
-- 데이터베이스 생성 (필요시)
CREATE DATABASE company_db;

-- 사용자 생성
CREATE USER 'developer'@'localhost' IDENTIFIED BY 'password123';
CREATE USER 'analyst'@'localhost' IDENTIFIED BY 'password456';
CREATE USER 'manager'@'localhost' IDENTIFIED BY 'password789';

-- 개발자 권한 부여 (모든 권한)
GRANT ALL PRIVILEGES ON company_db.* TO 'developer'@'localhost';

-- 분석가 권한 부여 (조회만 가능)
GRANT SELECT ON company_db.* TO 'analyst'@'localhost';

-- 매니저 권한 부여 (조회, 수정 가능)
GRANT SELECT, INSERT, UPDATE ON company_db.* TO 'manager'@'localhost';

-- 특정 테이블에 대한 권한 부여
GRANT SELECT, INSERT ON company_db.employee TO 'analyst'@'localhost';
GRANT SELECT ON company_db.department TO 'analyst'@'localhost';

-- 권한 확인
SHOW GRANTS FOR 'developer'@'localhost';

-- 권한 회수
REVOKE INSERT, UPDATE ON company_db.* FROM 'manager'@'localhost';

-- 권한 적용
FLUSH PRIVILEGES;
```

### 데이터베이스 보안 설정

```sql
-- 읽기 전용 사용자 생성
CREATE USER 'readonly_user'@'localhost' IDENTIFIED BY 'readonly_pass';
GRANT SELECT ON company_db.* TO 'readonly_user'@'localhost';

-- 백업용 사용자 생성
CREATE USER 'backup_user'@'localhost' IDENTIFIED BY 'backup_pass';
GRANT SELECT, LOCK TABLES ON company_db.* TO 'backup_user'@'localhost';

-- 관리자 권한 설정
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION;
```
