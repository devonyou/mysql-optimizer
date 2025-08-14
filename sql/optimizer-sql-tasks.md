# SQL

## 난이도 ★☆☆☆☆

1. 각 사원(employee)의 이름과 소속 부서(department) 이름을 출력하라.

```sql
SELECT e.name AS empl_name, d.name AS dept_name
FROM employee e
INNER JOIN department d
ON e.dept_id = d.id;
```

2. 프로젝트(project) 이름과 해당 프로젝트의 리더(leader)의 이름을 출력하라.

```sql
SELECT p.name, e.name AS empl_name
FROM project p
INNER JOIN employee e
ON p.leader_id = e.id
```

3. 사원과 그 사원이 참여한 프로젝트명을 출력하라 (works_on 조인).

```sql
SELECT e.name, p.name AS project_name
FROM works_on w
INNER JOIN employee e
ON w.empl_id = e.id
INNER JOIN project p
ON w.proj_id = p.id
```

4. 특정 부서(dept_id=3)에 속한 사원의 목록과 급여를 출력하라.

```sql
SELECT name, salary
FROM employee
WHERE dept_id = 3;

```

5. 프로젝트가 시작한 날짜가 2023년 이후인 프로젝트와 해당 프로젝트에 참여한 사원의 이름을 출력하라.

```sql
SELECT p.id, p.name AS project_name, e.name AS employee_name
FROM project p
INNER JOIN works_on w
ON w.proj_id = p.id
INNER JOIN employee e
ON e.id = w.empl_id
WHERE p.start_date >= '2023-01-01';
```

## 난이도 ★★☆☆☆

6. 각 부서별로 평균 급여를 계산하고 출력하라.

```sql
SELECT e.dept_id, avg(e.salary) AS avg_salary
FROM employee e
GROUP BY e.dept_id;
```

7. 가장 최근에 시작한 프로젝트 정보를 출력하라 (서브쿼리 사용).

```sql
SELECT *
FROM project
WHERE start_date = (
    SELECT MAX(start_date)
    FROM project
);
```

8. 프로젝트에 참여한 사원 수가 10명 이상인 프로젝트만 출력하라 (집계 + HAVING).

```sql
SELECT p.name, t.cnt
FROM (
	SELECT w.proj_id, count(*) AS  cnt
	FROM works_on w
	GROUP BY w.proj_id
	HAVING count(*) >=10
) AS t
LEFT JOIN project p
ON p.id = t.proj_id
```

9. 부서별 최고 급여를 받는 사원의 이름과 급여를 출력하라 (서브쿼리 또는 조인).

```sql
SELECT e.name, e.salary, e.dept_id
FROM employee e
INNER JOIN (
	SELECT dept_id, max(salary) AS max_salary
	FROM employee e
	GROUP BY dept_id
) AS em
ON em.dept_id = e.dept_id
AND em.max_salary = e.salary;
```

10. 특정 사원이 참여한 프로젝트 중 종료일이 지난 프로젝트만 출력하라 (서브쿼리, 날짜 조건).

```sql
SELECT *
FROM project p
INNER JOIN (
	SELECT proj_id
	FROM works_on
	WHERE empl_id = 3
) AS w
ON w.proj_id = p.id
WHERE p.end_date < now()
AND p.end_date IS NOT NULL;


SELECT *
FROM project p
INNER JOIN works_on w
ON w.proj_id = p.id
WHERE w.empl_id = 3
AND p.end_date < now()
```

## 난이도 ★★★☆☆

11. 각 프로젝트별 사원들의 급여 순위를 출력하라 (윈도우 함수 사용).

```sql
SELECT
    e.id,
    e.name,
    e.salary,
    ROW_NUMBER() OVER (PARTITION BY w.proj_id ORDER BY e.salary DESC) AS rank_no,
    w.proj_id
FROM employee AS e
INNER JOIN works_on AS w
ON e.id = w.empl_id
WHERE w.proj_id IN (29, 30);
```

12. 각 사원의 프로젝트 참여 횟수와 전체 프로젝트 수 대비 참여 비율을 계산하라.

```sql
WITH total_project AS (
	SELECT COUNT(DISTINCT proj_id) AS total_count
    FROM works_on
)
SELECT
	e.id,
	e.name,
	COUNT(w.proj_id) AS proj_cnt,
	ROUND(
		COUNT(w.proj_id) * 100 / t.total_count,
		2
	) AS participation_rate
FROM employee AS e
INNER JOIN works_on AS W
ON w.empl_id = e.id
CROSS JOIN total_project AS t
GROUP BY e.id, t.total_count;
```

13. 프로젝트별로 리더를 제외한 사원들의 평균 급여를 구하라.

```sql
SELECT w.proj_id, FLOOR(AVG(salary)) AS avg_salary
FROM employee AS e
INNER JOIN works_on AS w
ON w.empl_id = e.id
WHERE EXISTS (
	SELECT *
	FROM project AS p
	WHERE p.id = w.proj_id
	AND (p.leader_id != w.empl_id OR p.leader_id IS null)
)
GROUP BY w.proj_id;

SELECT p.id, FLOOR(AVG(e.salary))
FROM employee AS e
INNER JOIN works_on AS w
ON w.empl_id = e.id
INNER JOIN project AS p
ON p.id = w.proj_id
WHERE e.id <> p.leader_id OR p.leader_id IS NULL
GROUP BY p.id;

```

14. 가장 많이 참여한 프로젝트 상위 5개를 출력하라 (집계 + ORDER BY + LIMIT).

```sql
SELECT proj_id, COUNT(*) AS cnt
FROM works_on AS w
GROUP BY w.proj_id
ORDER BY cnt DESC, proj_id ASC
LIMIT 0, 5;
```

15. 각 부서별로 프로젝트 리더를 맡은 사원이 있는지 여부를 출력하라 (조건 포함 JOIN).

```sql
SELECT
	d.id,
	d.name,
	CASE
		WHEN t.leader_id IS NOT NULL THEN 'O'
		ELSE 'X'
	END AS has_project_leader
FROM department AS d
LEFT JOIN (
	SELECT DISTINCT p.leader_id, e.dept_id
	FROM project AS p
	INNER JOIN employee AS e
	ON e.id = p.leader_id
	WHERE p.leader_id IS NOT NULL
) AS t
ON t.dept_id = d.id;
```

## 난이도 ★★★★☆

16. 동일한 부서에 속한 사원들 중에서 급여가 평균 이상인 사원의 이름과 급여를 출력하라 (중첩 서브쿼리).

```sql
// 1
SELECT e.id, e.name, e.salary
FROM employee AS e
WHERE e.salary > (
	SELECT FLOOR(AVG(salary)) AS avg_salary
	FROM employee AS a
	WHERE a.dept_id = e.dept_id
);

// 2
SELECT e.name, e.salary, a.avg_salary
FROM employee e
LEFT JOIN (
	SELECT dept_id, FLOOR(AVG(salary)) AS avg_salary
	FROM employee
	GROUP BY dept_id
) AS a
ON a.dept_id = e.dept_id
WHERE e.salary > avg_salary;

// 3
SELECT id, name, salary, dept_id, dept_avg_salary
FROM (
    SELECT
      id,
      name,
      salary,
      dept_id,
      AVG(salary) OVER (PARTITION BY dept_id) AS dept_avg_salary
    FROM employee
) AS sub
WHERE salary >= dept_avg_salary;
```

17. 사원이 속한 부서의 평균 급여보다 높은 급여를 받으면서 프로젝트에 참여하지 않은 사원 목록을 출력하라 (NOT EXISTS).

```sql
SELECT *
FROM employee AS e
LEFT JOIN (
	SELECT dept_id, ROUND(AVG(salary)) avg_salary
	FROM employee
	GROUP BY dept_id
) AS a
ON a.dept_id = e.dept_id
WHERE NOT EXISTS (
	SELECT empl_id
	FROM works_on AS w
	WHERE w.empl_id = e.id
)
AND e.salary > a.avg_salary;
```

18. 사원별로 자신의 부서 내 급여 순위를 출력하라 (자기조인 또는 윈도우 함수).

```sql
// 1. 윈도우 함수
SELECT
	e.id,
	e.name,
	e.dept_id,
	e.salary,
	RANK() OVER (PARTITION BY e.dept_id ORDER BY e.salary DESC) AS salary_rank
FROM employee AS e
ORDER BY dept_id ASC, salary_rank ASC;

// 2. 자기조인
SELECT
	e.id,
	e.name,
	e.dept_id,
	e.salary,
	COUNT(DISTINCT e2.salary) AS rank_salary
FROM employee AS e
LEFT JOIN employee AS e2
ON e2.dept_id = e.dept_id
AND e2.salary >= e.salary
GROUP BY e.id, e.dept_id, e.salary
ORDER BY e.dept_id, e.salary;

```

19. 프로젝트별로 참여자들의 총 급여 합계와 그 중 최고 급여 사원의 이름을 함께 출력하라.

```sql
// 1
SELECT *
FROM (
	SELECT
	    w.proj_id,
	    e.name,
	    e.salary,
	    SUM(e.salary) OVER (PARTITION BY w.proj_id) AS total_salary,
	    ROW_NUMBER() OVER (PARTITION BY w.proj_id ORDER BY e.salary DESC) AS rn
	FROM works_on w
	JOIN employee e ON e.id = w.empl_id
	WHERE w.proj_id IS NOT null
) AS r
WHERE r.rn = 1
ORDER BY proj_id

// 2
EXPLAIN
WITH total_salaries AS (
    SELECT
        w.proj_id,
        SUM(e.salary) AS total_salary
    FROM works_on w
    JOIN employee e ON e.id = w.empl_id
    GROUP BY w.proj_id
),
max_salaries AS (
    SELECT
        w.proj_id,
        MAX(e.salary) AS max_salary
    FROM works_on w
    JOIN employee e ON e.id = w.empl_id
    GROUP BY w.proj_id
)
SELECT
    ts.proj_id,
    ts.total_salary,
    e.name AS top_salary_employee,
    ms.max_salary AS top_salary
FROM total_salaries ts
JOIN max_salaries ms ON ts.proj_id = ms.proj_id
JOIN works_on w ON w.proj_id = ms.proj_id
JOIN employee e ON e.id = w.empl_id AND e.salary = ms.max_salary
ORDER BY ts.proj_id;


```
