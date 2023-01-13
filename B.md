### 각요구사항을 100ms 이하로 반환 0.1보다 작게 100ms = 0.1sec

- [X] coding as hobby 0.187 sec

```sql
select round(count(case when hobby='yes' then 1 end) / count(hobby)*100,1),
round(count(case when hobby='no' then 1 end) / count(hobby)*100,1)
from programmer;
```
![image](https://user-images.githubusercontent.com/79621675/212216768-654673de-f1bb-405c-818b-2da7ddfeeb56.png)<br>
다음과같이 ALL로 풀테이블 스캔을 하고있습니다. 여기에 hobby에 인덱스를추가해보겠습니다
![image](https://user-images.githubusercontent.com/79621675/212216920-fe9d712b-ef86-4413-8c9b-22fbcc4c0048.png)<br>
![image](https://user-images.githubusercontent.com/79621675/212216998-afeea238-bf36-41b7-932a-a26aa82882f7.png)<br>

인덱스 적용후 0.04초대가 나옵니다



### 각 프로그래머별로 해당하는 병원 이름을 반환하세요. (covid.id, hospital.name)
![image](https://user-images.githubusercontent.com/79621675/212226266-8816a3f5-665f-45f2-b2e3-036cc5b3b723.png)

![image](https://user-images.githubusercontent.com/79621675/212226972-1aa14c91-83c6-487c-8cce-60a34605985a.png)
![image](https://user-images.githubusercontent.com/79621675/212227031-af5fcb58-21ac-483d-8852-27ad88e73c1a.png)


### 3번문제
전
![image](https://user-images.githubusercontent.com/79621675/212227977-b11bd4b6-2745-41fd-9d12-31f03aea61bb.png)
![image](https://user-images.githubusercontent.com/79621675/212227948-709c489d-5ea8-4a77-8dd8-66ada96edb30.png)
후는 할게없음
```sql
-- 프로그래밍이 취미인 학생 혹은 주니어(0-2년)들이 다닌 병원 이름을 반환하고 user.id 기준으로 정렬하세요. 
-- (covid.id, hospital.name, user.Hobby, user.DevType, user.YearsCoding)

select covid.id, hospital.name, programmer.hobby, programmer.dev_type, programmer.years_coding
from programmer
inner join covid on covid.programmer_id = programmer.id
inner join hospital on covid.hospital_id = hospital.id
where (programmer.hobby= 'Yes' and programmer.student != 'no') or programmer.years_coding = '0-2 years';
```


### 4번문제

