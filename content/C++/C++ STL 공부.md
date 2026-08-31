
*접근 : 위치를 알고 있다.
*탐색 : 위치를 모르는 상태에서 찾는 것.  

|자료구조|접근|탐색|삽입|삭제|특징|
|---|:---|:---:|---:|---:|---:|
|vector|O(1)|O(n)|끝 O(1)* / 중간 O(N)|끝 O(1) / 중간 O(N)|동적 배열, 가장 많이 사용
|list|O(n)|O(n)|위치알면 O(1)|위치알면(O(1)|이중 연결리스트|
|array|O(1)|O(n)|불가능|불가능|고정 크기 배열|
|deque|O(1)|O(n)|앞/뒤 O(1)|앞/뒤(O(1)|양쪽에서 삽입/삭제|
|set|-|O(log n)|O(log n)|O(log n)|정렬됨, 중복 엘레먼트 허용 x|
|multi-set|-|O(log n)|O(log n)|O(log n)|중복 엘레먼트 허용 o|
|unordered-set|-|O(1)|O(1)|O(1)|정렬x, 중복 엘레먼트 허용 x|
|map|-|O(log n)|O(log n)|O (log n)|key,value key 값 정렬|
|multi-map|-|O(log n)|O(log n)|O (log n)|key 중복 허용|
|unordered-map|-|O(1)|O(1)|O (1)|hash 기반 key, value|
|stack|-|-|O(1)|O(1)|LIFO (DFS, 재귀)|
|queue|-|-|O(1)|O(1)|FIFO (BFS)|
|priority-queue|-|-|O(log n)|O(log n)|우선순위가 높은 원소부터|

### vector

- 벡터 크기는 24바이트
8바이트 = Pointer
8바이트 = Size
8바이트 = Capacity

- Size와 Capacity가 같을 때, Capacity 확보가 필요할 때 쓰는 명령어
`reserve` 

만약 Capacity 확보가 필요한 데, 뒤에 메모리가 다른 메모리로 꽉 차 있다면?
--> move or copy가 일어나는데 copy가 일어 나는 경우 O(n)의 시간복잡도가 추가 됨.


### vector vs list 
- 99%의 경우 vector를 사용한다고 보면 됨.

제일 중요한 점은 find에서
vector는 메모리에 연속적으로 할당되어 있어서, 접근하는 순간 메모리에 Cache 안에 들어가서, 램에 접근할 필요가 없어지게 된다. 하지만 list의 경우에는 메모리 주소가 연속적이지 않으므로, Cache Miss 가능성이 있다. 

### set, multiset, unordered_set
- Set은 기본적으로 Red-Black-Tree (= Balanced Binary Search Tree BST)를 사용하는 자료 구조이다. 

- 값이 정렬되어 있는 것이 특징이다.


- 중복을 엘레먼트를 허용하지 않는다.
tree re-build (트리구조 재구성)
re-coloring (노드의 색깔만 바꾸는 것)

- multi-set의 경우 중복 엘레먼트를 허용한다.
- unordered_set = 해쉬를 사용한다, 버킷을 사용하고 각 버킷 안의 엘레먼트들은 링크드리스트로 연결되어 있다. re-hashing이 일어나는 경우 O(n)의 복사가 일어나게 된다. max-load-factor로 버킷당 최대 엘레먼트 수를 관리한다. (해당 엘레먼트 수를 넘으면 버킷수를 늘린다 = 즉 리해싱이 일어난다)

### map, unordered_map
- Map의 경우, Set + k,v(key-value)형태를 따른다.
그러므로 똑같이 Balanced BST가 사용되며, 키의 중복을 허용하지 않는다.

```
Map의 경우 
std::map<int,int> numPairs;
numPairs.emplace(1,101)
numPairs.emplace(2,102)
numPairs.emplace(3,103)
numPairs.emplace(4,104)
numPairs.emplace(5,105)

## 이렇게 되면 value 값을 overwrite 하게 된다.
numPairs[1] = 200; 

## 이렇게 되면, 6 key 값에 대한 value 0이 자동으로 채워짐
## square bracket에 대한 접근은 더 신중하게 사용해야 함.
std::cout << numPairs[6] << std::endl;
```
![](https://velog.velcdn.com/images/ohjiwoo123/post/e49f949d-b002-488f-b126-ac9f94bd77ce/image.png)  


### stack
```
template<
    class T,
    class Container = std::deque<T>
> class stack;
```

- LIFO (Last IN, First OUT) 
- 자주 사용하는 메소드 : emplace, pop, top

### queue, deque


### priority queue
```
template<
    class T,
    class Container = std::vector<T>,
    class Compare = std::less<typename Container::value_type>
> class priority_queue;
```

Insertion
Pop       
--> O(log n)

min or max --> O(1)

TREE 구조를 생각해야한다.
1, 3, 5, 7, 9 

|   9
| 7   5
|3 1
- 언제나 부모노드가 자식노드보다 크다. 
- POP 시에는 맨 아래 값이 트리의 부모가 된다.
- 다시 한번 부모와 자식이 비교되서 swap이 일어난다.

parent = p = (idx - 1) / 2
자식 L = 2*idx + 1 
자식 R = 2*idx + 2



