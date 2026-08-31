## Redis 자료형 특징
1. 데이터 타입에 따라 명령어가 다르다.
2. 대소문자 구별을 하지 않기 때문에 명령어를 대문자/소문자 구분없이 작성 가능하다.
 

### Strings
문자열, 숫자, serialized object(JSON) 등 여러 자료형을 저장하는 Strings Type이다. 레디스는 int, Integer 등 별도의 숫자 관련 자료형 없이 모두 strings로 저장한다. strings로 저장되어도 사칙 연산이 가능하다. 

#### Strings 명령어
1. SET, GET
```
$ set lecture inflearn-learn
>> OK
$ get lecture
>> "inflearn-learn"
```
2. MSET, MGET (여러 값을 set, get)
```
$ mset price 100 item banana
>> OK 
$ mget price item
>>
1) "100"
2) "banana"
```

3. 숫자 형태 Strings 저장할 때 (INCR, INCRBY)
- INCR : 숫자형 string 값을 +1
- INCRBY: 숫자형 string값을 지정한 만큼 +n
```
$ SET price 10
>> OK
$ INCR price 
>> price: 11
$ INCRBY price 9
>> price: 20
```

4.JSON 직접 저장
```
$ SET inflearn-redis '{"price": 100, "item": "abc"}'

# key: inflearn-redis
# value: JSON objects
```


5. redis에서는 일반적으로 key를 만들 때 의미별로 콜론을 통해 구별한다.
`$ SET inflearn-redis:ko:price 20` 

 

### Lists
Strings를 Linked List로 저장하는 데이터 타입으로, Double Linked List로 구현되어 있어, 그 특성에 맞게 양 끝에서 데이터를 조작할 수 있다. 즉, 양끝에서의 push/pop에 최적화되어있다. 상수 복잡도((O(1))을 갖는다.

 

#### Lists 명령어
1. LPUSH와 RPOP 조합으로 queue 조작하기
```
$ LPUSH queue job1 job2 job3
$ LPOP queue jop3 or RPOP queue job1 
```

2. LRANGE를 통해 다수의 아이템 조회하기
```
$ LPUSH stack job1 job2 job3
$ LRANGE queue -2 -1 
>>
1) "job2"
2) "job1"
$ LRANGE queue 0 -1 
>>
3) "job3"
4) "job2"
5) "job1"

$ LTRIM queue 0 1
>>
1) "job3"
2) "job2"
```
인덱스는 왼쪽부터 0으로 시작하고, 오른쪽(거꾸로)부터 -1로 시작한다.
 ![](https://velog.velcdn.com/images/ohjiwoo123/post/b3f66a02-9863-4694-a638-11a0670765db/image.png)

#### Sets
정렬 없이, 중복되지 않은 string을 저장하는 집합이다.


#### Sets 명령어
1. 집합 생성 및 원소 추가 명령어 (중복은 하나만 추가됨)
```
$ sadd person:1:favorite apple banana grape grape
```
2. 집합 조회
```
$ smembers person:1:favorite # apple banana grape
```
3. 집합 Cardinality(특정 집합의 유니크한 개수) 조회
```
$ scard person:1:favorite
>> 3
```

4. 특정 아이템 포함 여부 조회 
- 1(true) 또는 0(false) 반환
```
$ sismember person:1:favorite banana
$ sismember person:1:favorite melon
```
5. Set Operation
```
# 집합 추가 
$ sadd person2:favorite apple melon
>> (integer) 2
# person1, person2 교집합
$ sinter person1:favorite person:2:favorite
>> 1) "apple"
# 차집합 person1 - pesrson2
$ sdiff person1:favorite person:2:favorite
>> 
1) "banana"
2) "grape"
# 차집합 person2 - person1
$ sdiff person2:favorite person:1:favorite
>>
`) "melon"
# 합집합 person1 + person2
$ sunion person1:favorite person:2:favorite
>>
1) "melon"
2) "banana"
3) "grape"
4) "apple"

```

![](https://velog.velcdn.com/images/ohjiwoo123/post/c14f152a-3bf5-4b28-b6d9-c13846b3bdb3/image.png)

6. 집합에서 특정 멤버 제거 및 집학 삭제
```
SADD myset a b c d   # myset에 a, b, c, d 추가
SREM myset b c       # myset에서 b, c 제거
SMEMBERS myset       # 남은 값 확인 → a, d
DEL key              # 집합 전체 삭제
```

