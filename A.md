v2 sql 0.562
```sql
select 다섯명.사원번호, 사원.이름, 다섯명.연봉, 직급.직급명, 사원출입기록.지역, 사원출입기록.입출입구분, 사원출입기록.입출입시간
from ( 
select 부서관리자.사원번호, 급여.연봉 from 부서관리자
inner join 부서 on 부서관리자.부서번호 = 부서.부서번호
inner join 급여 on 부서관리자.사원번호 = 급여.사원번호
where 부서.비고 = 'active'
order by 급여.연봉 desc limit 5
) 다섯명
inner join 사원 on 다섯명.사원번호 = 사원.사원번호
inner join 직급 on 다섯명.사원번호 = 직급.사원번호
inner join 사원출입기록 on 다섯명.사원번호 = 사원출입기록.사원번호
where 사원출입기록.입출입구분 = 'O'
order by 다섯명.연봉 desc;
```

v3 0.391
``` sql
select 다섯명.사원번호, 사원.이름, 다섯명.연봉, 직급.직급명, 사원출입기록.지역, 사원출입기록.입출입구분, 사원출입기록.입출입시간
from ( 
select 부서관리자.사원번호, 급여.연봉 from 부서관리자
inner join 부서 on 부서관리자.부서번호 = 부서.부서번호
inner join 급여 on 부서관리자.사원번호 = 급여.사원번호
where 부서.비고 = 'active' AND 부서관리자.종료일자 > NOW() AND 급여.종료일자 > NOW()
order by 급여.연봉 desc limit 5
) 다섯명
inner join 사원 on 다섯명.사원번호 = 사원.사원번호
inner join 직급 on 다섯명.사원번호 = 직급.사원번호
inner join 사원출입기록 on 다섯명.사원번호 = 사원출입기록.사원번호
where 사원출입기록.입출입구분 = 'O' AND 직급.종료일자 > NOW()
order by 다섯명.연봉 desc;
``` 


v4 0.34
``` sql
select 다섯명.사원번호, 사원.이름, 다섯명.연봉, 다섯명.직급명, 사원출입기록.지역, 사원출입기록.입출입구분, 사원출입기록.입출입시간
from ( 
select 부서관리자.사원번호, 급여.연봉, 직급.직급명 from 부서관리자
inner join 부서 on 부서관리자.부서번호 = 부서.부서번호
inner join 급여 on 부서관리자.사원번호 = 급여.사원번호
inner join 직급 on 부서관리자.사원번호 = 직급.사원번호
where 부서.비고 = 'active' AND 부서관리자.종료일자 > NOW() AND 급여.종료일자 > NOW() AND 직급.종료일자 > NOW()
order by 급여.연봉 desc limit 5
) 다섯명
inner join 사원 on 다섯명.사원번호 = 사원.사원번호
inner join 사원출입기록 on 다섯명.사원번호 = 사원출입기록.사원번호
where 사원출입기록.입출입구분 = 'O'
order by 다섯명.연봉 desc;
```

핵심은 5명으로 거르고 진행하는것이다 인덱스처럼 조건문으로 최대한 걸러서 진행하면 옵티마이저가 더 빠르게 찾아줄것이다.
거르고 진행하지않는다면 where는 join다음에 작동되니 부서관리자의 수만큼 row를 생성한 다음 where조건절이 진행되므로 시간이 더걸린다.
그래서 처음에는 부서관리자.사원번호로 되어있어서 다른테이블들이 PK로 사원번호를 갖는걸보고 1:1매핑이고 그래서 1:1매핑친구들은 row개수가 어차피같기때문에
한꺼번에 모아서 진행을 해도 지장이없었다. 출입기록은 복합키이며 중복되는 사원번호가 많기때문에 join을 한다면 row가 더생길거기때문에 나누어서 진행하는게 맞았다.
ERD를 통해 관계를 파악하고 진행하는게 핵심이다
```
select count(사원번호) from 사원출입기록; -- 660,000
select count(distinct(사원번호)) from 사원출입기록; -- 150,000
```

우테코
``` sql
SELECT 관리자.사원번호, 관리자.이름, 관리자.연봉, 관리자.직급명, 사원출입기록.입출입시간, 사원출입기록.지역, 사원출입기록.입출입구분 
FROM (
    SELECT 부서관리자.사원번호, 사원.이름, 급여.연봉, 직급.직급명 FROM 부서관리자
    JOIN 부서 ON 부서관리자.부서번호 = 부서.부서번호
    JOIN 사원 ON 부서관리자.사원번호 = 사원.사원번호
    JOIN 급여 ON 부서관리자.사원번호 = 급여.사원번호
    JOIN 직급 ON 부서관리자.사원번호 = 직급.사원번호
    WHERE 부서.비고 = 'active' AND 부서관리자.종료일자 > NOW() AND 급여.종료일자 > NOW() AND 직급.종료일자 > NOW()
    ORDER BY 급여.연봉 DESC 
	LIMIT 5
) 관리자 
JOIN 사원출입기록 ON 관리자.사원번호 = 사원출입기록.사원번호
WHERE 사원출입기록.입출입구분 = "O"
ORDER BY 관리자.연봉 DESC, 관리자.사원번호, 사원출입기록.지역
```

### 인덱스 설정을 추가하여 50 ms 이하로 반환한다.
![image](https://user-images.githubusercontent.com/79621675/212217757-e222a445-3357-4c76-85b5-c90969bb03da.png)
```
사원출입기록에 풀테이블스캔과 row의 개수가 많은걸보고 사원번호에 인덱스를 걸었다
```
### 적용후
![image](https://user-images.githubusercontent.com/79621675/212219749-e17fa021-f130-453f-a788-4d112f102cbb.png)


where 조건을 뒤로밀었다 변화없었다...
![image](https://user-images.githubusercontent.com/79621675/212220170-443d10aa-a3ee-4438-980e-8291a42d658c.png)

