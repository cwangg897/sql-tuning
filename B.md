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
