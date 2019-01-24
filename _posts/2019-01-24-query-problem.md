---
layout: post
title:  서로 다른 테이블의 로우 숫자와 컬럼의 값이 다른지 비교하는 쿼리
---

한 테이블의 로우 개수를 구할때 그 테이블의 로우개수를 직접 구하지 않고 다른 테이블에 로우 개수를 적어 놓는 경우가 있습니다.

간단히 예를들어 기획상 로우 개수가 제한이 없는 경우 한번에 모든 데이터를 가져와서 개수를 구한다면 어플리케이션에서 문제가 발생 할 수 있기 때문이죠

이런 상황에서 테이블 로우 개수와 다른 테이블에 적어놓은 개수가 다른지 확인 하는 쿼리 입니다.


```
Sample table FRIEND 

+---------+-----------------+
| USER_ID | FRIEND_USER_ID  |
+---------+-----------------+
|    1    |       3         |
|    1    |       4         |
|    1    |       5         |
|    1    |       6         |
|    2    |       3         |
|    2    |       4         |
|    2    |       5         |
|    2    |       6         |
+---------+-----------------+

Sample table FRIEND_COUNT

+---------+-----------------+
| USER_ID | FRIEND_COUNT    |
+------------+--------------+
|    1    |       3         |
|    2    |       4         |
+---------+-----------------+
```

위의 경우에 친구 숫자가 제한이 없는 기획이라고 한다면 친구 숫자를 적는 테이블을 따로 적어 두는게 좋습니다. 

하지만 어떤 이유로 FRIEND 테이블의 로우 개수와 FRIEND_COUNT 테이블의 FRIEND_COUNT 값이 싱크가 맞지 않을 수 있습니다.  

위 테이블에 USER_ID 1번의 경우 FRIEND 테이블에 FRIEND_USER_ID 가 3, 4, 5, 6 4개의 로우를 가지고 있는데 FRIEND_COUNT 테이블엔 3으로 적혀있습니다.  

이때 저런 데이터들을 찾는 쿼리는 다음과 같습니다.

```
SELECT friend.USER_ID, friend.REAL_FAN_COUNT, friend_count.FRIEND_COUNT
FROM (
	SELECT USER_ID, COUNT(1) AS REAL_FAN_COUNT FROM FRIEND GROUP BY USER_ID
) AS friend
JOIN FRIEND_COUNT friend_count
ON friend.USER_ID = friend_count.USER_ID AND friend.REAL_FAN_COUNT != friend_count.FRIEND_COUNT
```


#### 참고 사이트

https://stackoverflow.com/questions/9390679/left-join-after-group-by