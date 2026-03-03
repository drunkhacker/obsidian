셀프 조인으로 여러개의 레코드들 중에 가장 최근 값 가져오기.
https://stackoverflow.com/a/2111420/22186157

left outer join으로 p1.date < p2.date 으로 조건을 걸면..

나보다 큰 애들 다 나올거고, 
거기서 p2.id is null로 거르면, 
가장 최근 놈만 나오지. 왜냐면 가장 최근 row는 나보다 늦은 row가 없기 때문에 p2 파트가 null로 나올것.




