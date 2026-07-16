---
tags:
  - cs/certificate
  - certificate/정보처리기사
  - exam/practical
  - type/note
area: SQL 응용
priority: 최우선
created: 2026-07-16
updated: 2026-07-16
---

# SQL 응용

## 공부 전략

- SQL은 단답형과 실행 결과 추론이 모두 나오므로 문법 암기와 손풀이를 같이 한다.
- `SELECT 실행 순서`, `JOIN`, `GROUP BY/HAVING`, `서브쿼리`, `정규화`, `트랜잭션`을 최우선으로 본다.
- 문제를 풀 때는 항상 테이블을 작게 그려서 조건, 그룹, 정렬 순서대로 지운다.

## 핵심 키워드

- DDL, DML, DCL, TCL
- SELECT
- WHERE
- GROUP BY
- HAVING
- ORDER BY
- JOIN
- 서브쿼리
- 집합 연산자
- 제약조건
- 정규화
- 인덱스
- 트랜잭션

## 이론 정리

### SQL 분류

| 분류 | 의미 | 주요 명령어 |
|---|---|---|
| DDL | 데이터 정의어. 테이블, 뷰, 인덱스 같은 객체를 정의한다. | `CREATE`, `ALTER`, `DROP`, `TRUNCATE` |
| DML | 데이터 조작어. 테이블의 행을 조회, 삽입, 수정, 삭제한다. | `SELECT`, `INSERT`, `UPDATE`, `DELETE` |
| DCL | 데이터 제어어. 권한을 부여하거나 회수한다. | `GRANT`, `REVOKE` |
| TCL | 트랜잭션 제어어. 작업 단위를 확정하거나 되돌린다. | `COMMIT`, `ROLLBACK`, `SAVEPOINT` |

### SELECT 기본 구조

```sql
SELECT 컬럼명
FROM 테이블명
WHERE 조건
GROUP BY 그룹기준컬럼
HAVING 그룹조건
ORDER BY 정렬기준컬럼;
```

실행 순서는 문법 순서와 다르다.

1. `FROM`: 조회할 테이블을 정한다.
2. `WHERE`: 행 단위 조건을 먼저 거른다.
3. `GROUP BY`: 남은 행을 그룹으로 묶는다.
4. `HAVING`: 그룹에 대한 조건을 거른다.
5. `SELECT`: 출력할 컬럼이나 계산식을 정한다.
6. `ORDER BY`: 최종 결과를 정렬한다.

### WHERE와 HAVING

- `WHERE`: 그룹화 전, 개별 행에 적용한다.
- `HAVING`: 그룹화 후, 집계 결과에 적용한다.

```sql
SELECT dept, COUNT(*) AS cnt
FROM employee
WHERE salary >= 3000
GROUP BY dept
HAVING COUNT(*) >= 2;
```

위 SQL은 급여가 3000 이상인 행만 먼저 남기고, 부서별로 묶은 뒤, 인원이 2명 이상인 부서만 출력한다.

### 주요 비교/논리 연산자

| 표현 | 의미 |
|---|---|
| `=` | 같다 |
| `<>`, `!=` | 같지 않다 |
| `BETWEEN A AND B` | A 이상 B 이하 |
| `IN (A, B, C)` | 목록 중 하나와 일치 |
| `LIKE` | 패턴 비교 |
| `IS NULL` | NULL 값 확인 |
| `AND` | 두 조건이 모두 참 |
| `OR` | 둘 중 하나 이상 참 |
| `NOT` | 조건 부정 |

`NULL`은 값이 없음을 뜻하므로 `= NULL`이 아니라 `IS NULL`로 비교한다.

### LIKE 패턴

| 패턴 | 의미 |
|---|---|
| `%` | 0개 이상의 임의 문자열 |
| `_` | 임의의 한 문자 |

```sql
SELECT *
FROM student
WHERE name LIKE '김%';
```

이 SQL은 이름이 `김`으로 시작하는 행을 조회한다.

### 집계 함수

| 함수 | 의미 |
|---|---|
| `COUNT(*)` | 전체 행 수 |
| `COUNT(컬럼)` | NULL을 제외한 컬럼 값 개수 |
| `SUM(컬럼)` | 합계 |
| `AVG(컬럼)` | 평균 |
| `MAX(컬럼)` | 최댓값 |
| `MIN(컬럼)` | 최솟값 |

`COUNT(*)`는 NULL 여부와 무관하게 행 자체를 센다. `COUNT(컬럼)`은 해당 컬럼이 NULL인 행을 제외한다.

### JOIN

| 종류 | 의미 |
|---|---|
| `INNER JOIN` | 두 테이블에서 조인 조건이 일치하는 행만 출력 |
| `LEFT OUTER JOIN` | 왼쪽 테이블은 모두 출력하고, 오른쪽은 일치하는 값만 붙임 |
| `RIGHT OUTER JOIN` | 오른쪽 테이블은 모두 출력하고, 왼쪽은 일치하는 값만 붙임 |
| `FULL OUTER JOIN` | 양쪽 테이블의 모든 행을 출력 |
| `CROSS JOIN` | 가능한 모든 조합을 출력 |
| `SELF JOIN` | 같은 테이블을 자기 자신과 조인 |

```sql
SELECT e.name, d.dept_name
FROM employee e
INNER JOIN department d
ON e.dept_id = d.dept_id;
```

조인 문제는 먼저 기준 테이블을 정하고, `ON` 조건이 맞는 행만 연결한다. 외부 조인은 매칭되지 않는 쪽에 NULL이 생길 수 있다.

### 서브쿼리

서브쿼리는 SQL 내부에 포함된 또 다른 SQL이다.

| 종류 | 설명 |
|---|---|
| 단일 행 서브쿼리 | 결과가 한 행이다. `=`, `>`, `<` 같은 비교 연산자 사용 |
| 다중 행 서브쿼리 | 결과가 여러 행이다. `IN`, `ANY`, `ALL`, `EXISTS` 사용 |
| 스칼라 서브쿼리 | 하나의 값처럼 사용되는 서브쿼리 |
| 상관 서브쿼리 | 바깥 쿼리의 값을 참조하는 서브쿼리 |

```sql
SELECT name
FROM employee
WHERE salary > (
  SELECT AVG(salary)
  FROM employee
);
```

이 SQL은 전체 평균 급여보다 급여가 높은 직원을 조회한다.

### 집합 연산자

| 연산자 | 의미 | 중복 |
|---|---|---|
| `UNION` | 합집합 | 제거 |
| `UNION ALL` | 합집합 | 유지 |
| `INTERSECT` | 교집합 | 제거 |
| `MINUS`, `EXCEPT` | 차집합 | 제거 |

집합 연산자는 양쪽 SELECT의 컬럼 개수와 데이터 타입이 맞아야 한다.

### DDL

```sql
CREATE TABLE student (
  student_id INT PRIMARY KEY,
  name VARCHAR(20) NOT NULL,
  dept VARCHAR(20),
  score INT CHECK (score BETWEEN 0 AND 100)
);
```

- `CREATE`: 객체 생성
- `ALTER`: 객체 구조 변경
- `DROP`: 객체 삭제
- `TRUNCATE`: 테이블의 모든 행 삭제, 구조는 유지

`DROP`은 테이블 구조까지 삭제하고, `TRUNCATE`는 구조를 남긴 채 데이터만 빠르게 삭제한다.

### 제약조건

| 제약조건 | 의미 |
|---|---|
| `PRIMARY KEY` | 기본키. 행을 유일하게 식별하며 NULL 불가 |
| `FOREIGN KEY` | 외래키. 다른 테이블의 기본키를 참조 |
| `UNIQUE` | 중복 불가 |
| `NOT NULL` | NULL 불가 |
| `CHECK` | 조건을 만족하는 값만 허용 |
| `DEFAULT` | 값이 없을 때 기본값 입력 |

### DML

```sql
INSERT INTO student(student_id, name, dept)
VALUES (1, 'Kim', 'CS');

UPDATE student
SET dept = 'AI'
WHERE student_id = 1;

DELETE FROM student
WHERE student_id = 1;
```

`UPDATE`와 `DELETE`는 `WHERE` 조건이 없으면 전체 행에 적용될 수 있다.

### DCL

```sql
GRANT SELECT, INSERT ON student TO user1;
REVOKE INSERT ON student FROM user1;
```

- `GRANT`: 권한 부여
- `REVOKE`: 권한 회수

### 트랜잭션

트랜잭션은 데이터베이스에서 하나의 논리적 작업 단위이다.

| 특성 | 의미 |
|---|---|
| Atomicity | 원자성. 모두 수행되거나 모두 수행되지 않아야 한다. |
| Consistency | 일관성. 수행 전후 데이터가 제약조건을 만족해야 한다. |
| Isolation | 격리성. 동시에 실행되는 트랜잭션이 서로 간섭하지 않아야 한다. |
| Durability | 지속성. 완료된 결과는 장애가 발생해도 보존되어야 한다. |

```sql
SAVEPOINT s1;
ROLLBACK TO s1;
COMMIT;
```

- `COMMIT`: 변경 내용을 확정
- `ROLLBACK`: 변경 내용을 취소
- `SAVEPOINT`: 되돌아갈 중간 지점 설정

### 정규화

정규화는 데이터 중복과 이상 현상을 줄이기 위해 테이블을 분해하는 과정이다.

| 단계 | 핵심 |
|---|---|
| 1NF | 원자값이 아닌 반복 속성 제거 |
| 2NF | 부분 함수 종속 제거 |
| 3NF | 이행 함수 종속 제거 |
| BCNF | 결정자가 후보키가 아니면 분해 |

이상 현상은 삽입 이상, 삭제 이상, 갱신 이상으로 구분한다.

### 인덱스

인덱스는 검색 속도를 높이기 위한 자료구조이다.

- 장점: 조회 성능 향상, 정렬/검색 비용 감소
- 단점: 저장 공간 추가 사용, 삽입/수정/삭제 시 인덱스 갱신 비용 발생

인덱스는 조회가 많은 컬럼에 유리하고, 변경이 잦은 컬럼에는 부담이 될 수 있다.

### 뷰

뷰는 하나 이상의 테이블에서 유도된 가상의 테이블이다.

- 장점: 보안성 향상, 복잡한 질의 단순화, 논리적 독립성 제공
- 단점: 뷰 정의에 따라 삽입/수정/삭제가 제한될 수 있다.

## 기출 포인트

- [ ] `SELECT` 실행 순서로 결과 추론
- [ ] `WHERE`와 `HAVING` 차이
- [ ] `COUNT(*)`와 `COUNT(컬럼)` 차이
- [ ] `INNER JOIN`과 외부 조인 결과 비교
- [ ] 서브쿼리 결과가 단일 행인지 다중 행인지 판단
- [ ] `UNION`과 `UNION ALL`의 중복 처리 차이
- [ ] `DROP`, `TRUNCATE`, `DELETE` 차이
- [ ] 기본키, 외래키, UNIQUE, CHECK 제약조건 구분
- [ ] 정규화 단계별 제거 대상 암기
- [ ] 트랜잭션 ACID와 `COMMIT`/`ROLLBACK`

## 오답 노트

| 문제 키워드 | 헷갈린 개념 | 정답 근거 | 다시 볼 것 |
|---|---|---|---|
|  |  |  |  |

## 회독 체크

- [ ] 1회독
- [ ] 2회독
- [ ] 기출 오답 반영
