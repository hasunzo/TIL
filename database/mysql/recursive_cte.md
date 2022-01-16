# 재귀적 CTE (Recursive CTE)
```mysql
with recursive cte (no) as (
	select 1    -- 비 재귀적 파트
	union all 
	select (no+1) from cte where no < 5 -- 재귀적 파트
)
select * from cte;
no
1
2
3
4
5
```
- 재귀적 CTE 쿼리는 비 재귀적 쿼리 파트와 재귀적 파트로 구분되며, 이 둘을 UNION(UNION DISTINCT) 또는 UNION ALL로 연결하는 형태로
반드시 쿼리를 작성해야 한다.
- 비 재귀적 파트는 처음 한 번만 실행되지만 재귀적 파트는 쿼리 결과가 없을 때까지 반복 실행된다.

#### 위 예제 쿼리가 작동하는 방법
1. CTE 쿼리의 비 재귀적 파트의 쿼리를 실행
2. 1번의 결과를 이용해 cte라는 이름의 임시 테이블 생성
3. 1번의 결과를 cte라는 임시 테이블에 저장
4. 1번 결과를 입력으로 사용해 CTE 쿼리의 재귀적 파트의 쿼리를 실행
5. 4번의 결과를 cte라는 임시 테이블에 저장(이때 UNION 또는 UNION DISTINCT의 경우 중복 제거를 실행)
6. 전 단계의 결과를 입력으로 사용해 CTE 쿼리의 재귀적 파트 쿼리를 실행
7. 6번 단계에서 쿼리 결과가 없으면 CTE 쿼리를 종료
8. 6번 결과를 cte라는 임시 테이블에 저장
9. 6번으로 돌아가서 반복 실행

> 1번 과정에서 CTE 임시 테이블의 구조가 결정된다.
- CTE 임시 테이블의 구조 (테이블의 칼럼명과 칼럼의 데이터 타입)는 CTE 쿼리의 비 재귀적 쿼리 파트의 결과로 결정된다.
- 예를 들어, 비 재귀적 파트의 결과와 재귀적 파트의 결과에서 칼럼 개수나 칼럼의 타입, 칼럼 이름이 서로 다른 경우 MySQL 서버는 비 재귀적 파트에 정의된 결과를 사용한다.
- CTE의 비 재귀적 쿼리 파트는 초기 데이터와 임시 테이블의 구조를 준비하고, 재귀적 쿼리 파트에서는 이후 데이터를 생성해내는 역할을 한다.

> 재귀적 쿼리 파트를 실행할 때는 지금까지의 모든 단계에서 만들어진 결과 셋이 아니라 직전 단계의 결과만 재귀 쿼리의 입력으로 사용된다.
- 예제 쿼리에서 재귀적 쿼리 파트는 "SELECT (no + 1) FROM cte WHERE no < 5"로 작성 되어 있음.
- 이 문장이 cte 테이블의 모든 레코드를 조회하는 것이 아니라 직전 단계에서 만들어진 결과만을 참조해서 쿼리가 실행됨.
![Desktop/TIL/database/mysql/img/cte.png](Desktop/TIL/database/mysql/img/cte.png)
  
### 계층형 데이터에 CTE 활용하기
```mysql
with recursive cte as (
	select bc.board_comment_id, bc.board_comment_parent_id, bc.board_comment_content,
		cast (board_comment_id as varchar(128)) as comment_path,
		0 as level
	from board_comment bc
	where board_comment_id = 1
	
	union all 
	select bc.board_comment_id, bc.board_comment_parent_id, bc.board_comment_content,
		concat(c.comment_path, ' ->', bc.board_comment_id) as comment_path,
		level +1
	from cte c
	inner join board_comment bc 
	on c.board_comment_id = bc.board_comment_parent_id 
)

select * from cte
order by comment_path;
```

