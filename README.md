# -Large_volume_data_processing_index
대용량데이터 처리의 꽃, INDEX를 알아보고, 성능 비교를 해보자

## 생각
![image](https://github.com/HyungjuLee95/-Large_volume_data_processing_index/assets/111270174/e29f7b24-b50c-46fa-ae04-0e413025d437)

CPU가 데이터를 가져오거나 저장하기 위해서는 I/O(입출력) 버스를 통하게 되며, 아래 장치들로 데이터를 보내게된다.<br>
여기서 저장하기 위한 용도로 메모리와 디스크를 사용한다. 생각을 해본다면 결국에는 데이터는 디스크에 저장이 되어야한다.<br>
<strong>디스크는 메모리에 비해 속도가 느리기 때문에, 결국! 데이터베이스 성능의 핵심은 디스크 I/O를 최소화 하는 것이다.</strong><br>

### 어떻게?
메모리에 올라온 데이터로 최대한 요청을 처리하는 것 => 메모리 캐시 히트율을 높이는 것<br>
읽기 쓰기 중, 쓰기도 메모리에 먼저 쓰는 방법 -> 전원이 꺼져 메모리에 저장한 데이터가 유실된다면? 이것을 고려하여 WAL(Write Ahead Log)를 사용.<br>

![image](https://github.com/HyungjuLee95/-Large_volume_data_processing_index/assets/111270174/effadfd0-59cc-41e8-bb46-babed49c9899)<br>
랜덤I/O 정말 무작위로 데이터를 가져오는 것.<br>
순차 I/O 연속된 블락의 데이터를 읽어오는 것.<br>

대부분의 트랜잭션은 무작위하게 Write가 발생한다.<br>
예를 들어 A테이블의 10번 ID를 업데이트를 하거나, 100 번 ID를 가진 걸 업데이트를 할 수 있는데, 이를 보면 대부분 무작위하게 Write할 수 있음.<br>
앞에서 언급한 wal은 하나의 파일에 끝부분부터 순차적으로 write만 쌓아두는 것.<br><br>

즉, 이렇게 하게된다면 랜던 I/O보다는 순자 I/O가 진행이 된다.<br><br>

그래서! 데이터베이스는 업데이트는 디스크에 가서 저장하는 것은 비효율적이기 때문에 메모리에 쌓아두고 한번에 넣으면 되잖아! 라는 생각을 할 수 있다.<br>
그럼 전원이 꺼지게된다면? 데이터가 유실이 된다면 파일에 순차적으로 쌓아두었던 로그를 순차적으로 실행시키면 다시 복구할 수 있다.<br><br>

### Index?
-> 정렬된 자료 구조, 이를 통해 탐색 볌위를 최소<br>
아래 테이를 봐보자<br>
![image](https://github.com/HyungjuLee95/Large_volume_data_processing_index/assets/111270174/e46faae0-a410-47d0-8009-3047d0462991)<br>

앞 번호는 pk라고 생각할 수 있고, 만약 여기서 나이가 가장 낮은 데이터를 찾고싶다고하자.<br>
칼럼을 순서대로 돌면서 아마 나이들을 확인하고 어딘가에 저장할 것이다.<br>
여기서 나이를 index로 설정을 한다면, 아래처럼 인덱스별로 정렬이 되어있으면, 첫번째 데이터를 찾고 해당 인덱스를 찾아간다.<br>
![image](https://github.com/HyungjuLee95/Large_volume_data_processing_index/assets/111270174/ceb06db6-f4a5-4f54-83db-9b99afaa9e4e)<br><br>

주의할 것은! 인덱스도 테이블이다!<br>
실제로 데이터 베이스는 쿼리가 들어오면 index를 한번 확인하고, index를 조회한 후, 원본 데이터를 찾아가게된다.<br>
<strong>즉, 인덱스의 핵심은 정렬을 통해 탐색(검색) 범위를 최소화하는 것.</strong><br><br>

### 그렇다면 검색이 빠른 자료 구조는 무엇이 있을까?<br>
Hash Map, List, Binary Search Tree<br>
#### 하나씩 확인해보자
1. HashMap<br>
> a) 단건 검색 속도O(1) -> key와 value가 있기에 충돌이 있음을 감안하더라도 상수시간 안에 가능하다<br>
> b) 범위 탐색은 O(N) -> 예를 들어서 key가 나이로 되어있다면, HashMap은 전체를 다 보아야할 것이다.<br>
> c) 전방 일치 탐색 불가 -> 예를 들어서 'like 'AB%' 라고 쿼리가 나갔을 때, Hash Map은 하나하나 key를 다 꺼내서 확인을 해보아야한다.<br><br>
2. List<br>
> a)정렬되지 않은 리스트의 탐색은 O(N)<br>
> b) 정렬된 리스트의 탐색은 O(logN)<br>
> c) 정렬되지 않은 리스트의 정렬 시간 복잡도 O(N) ~ )(N*logN)<br>
> d) 삽입/ 삭제 비용이 매우 높음 -> 왜? 중간 것을 삭제한다면 앞이든 뒤든 다 꺼낸다음 삭제하고 다시 넣어야하기 때문이다.<br><br>
3. Binary Search Tree<br>
> a) 트리 높이에 따라 시간 복잡도가 결정됨 -> Binary Search을 이용한다는 기준 하에 트리 높이에 따라서 결덩이 된다.<br>
> b) 트리 높이를 최소화하는 것이 중요<br>
> c) 한쪽으로 노드가 치우치지 않도록 균형을 잡아주는 트리 사용 -> Red Black Tress, B+Tree<br>
> -> 대부분 B+Tree를 사용함<br><br>
4.B+Tree<br>
> a) 삽입 / 삭제시 항상 균형을 이룸<br>
> b) 하나의 노드가 여러 개의 자식 노드를 가질 수 있음<br>
> c) 리프노드에만 데이터 존재<br>
> ->연속적인 데이터 접근시 유리<br>(b Tree와의 차이는 각 node가 데이터가 된다. 다만 B+Tree는 리프 노드에만 데이터가 존재하고, 위에 노드들은 데이터를 찾아가기 위한 Key가 된다. 그래서 연속적인 데이터를 접근할 때 리프 노드만 흝으면 된다. )<br><br>
> ※Pk를 가지고 있는 B+tree를 생각해 보았을 때, 데이터를 삭제하거나 삽입할 때, B+Tree의 모습이 바뀌는데, 이러한 것은 데이터에서도 동일하게 일어난다. 그 말은 즉슨, 조회의 속도는 높일 수 있으나, 쓰기나 갱신의 성능을 낮출 수 있으므로 벨런스를 잡는 것이 중요하다. 
---

### 클러스터 인덱스
1. 클러스터 인덱스는 데이터 위치를 결정하는 키 값이다.
2. MySql의 Pk는 클러스터 인덱스다.
3. MySql에서 PK를 제외한 모든 인덱스는 Pk를 가지고있다.

하나씩 생각을 해보자<br>
![image](https://github.com/HyungjuLee95/Large_volume_data_processing_index/assets/111270174/f6844701-2d59-405a-b1aa-3dfd3dea6d88) <br>
이 상황에서 클러스터 키가 4번인 녀석이 insert된다고 생각해보자.
클러스터 키는 정렬되어 있고, 정렬된 순서에 따라 데이터 주소가 결정된다는 특징을 기억해보자.<br>
그렇다면 4번은 3번과 5번 사이에 있을 것으로, 5번이 아랫쪽 데이터 주소로 밀리게 되며, 그 사이에 4번이 insert되게 된다.<br><br>

정리하면! 클러스터 키는 정렬된 자료 구조이고, 클러스터의 위치에 따라서 데이터 위치가 결정이 된다.<br>
만약 위와 같은 경우, 한건이지만 백만건이 들어온다고 가정하면 그만큼 더욱 변경되는게 많을것이다.<br>
그래서 클러스터 키 삽입/수정 시에 성능이슈가 발생할 수 있다.<br>

----
### 고민을 해보자
Pk로 Auto Increment와 UUID 둘 중 어떤 것을 사용할지. 두개의 장단점을 알아보자.<br><br><br>

---
Mysql에서 Pk를 제외한 모든 인덱스는 Pk를 가지고있다. 즉, Pk의 사이즈가 인덱스의 사이즈를 결정하게된다.<br>
Mysql이 Pk를 가지고 있는 이유는 Pk가 클러스터 인덱스이기 때문에 Pk가 insert/update가 된다면 데이터 주소가 바뀌게 된다.<br>
그런데 index들이 pk가 아닌 데이터 주소를 직접적으로 가지고있다면 이 data 순서가 바뀔 때, index도 같이 갱신이 되어야하기 때문에 더욱 부화가 될 수 밖에 없다.<br>
하여, 실제로 index는 pk를 들고있고, pk의 위치가 변경될 때 index는 아무 상관없이 pk를 들고있으니 영향이 덜한다.'
그렇기 때문에 세컨더리 인덱스(보조 인덱스)만으로는 데이터를 찾아갈 수 없다.<br>
-> 그래서 항상 세컨더리 인덱스에서 pk를 찾고, 이 pk에서 데이터를 찾아가야한다.<br><br>

위 내용만 본다면 단점 밖에 없는 것 같으나, 장점도 있다.<br>
1. Pk를 활용한 검색이 빠름, 특히 범위 검색이 빠르다.<br>
-> Pk가 정렬되어 있고, 이에 맞게 데이터가 모여있기 때문에 빠르며, 공간적 캐쉬 이점을 노릴 수 있다.<br><br>
2. 세컨더리 인덱스 들이 Pk를 가지고 있어, 커버링에 유리<br>
->커버링이 뭐지? 간단하게 인덱스 테이블 자체만으로도 데이터 응답을 모두 내려줄 수 있어서 테이블까지 가지 않아도 된다.<br><br>

클러스터 인덱스와 세컨더리 인덱스, 커버링 인덱스의 예시를 생각해보자.

---

예시 1: 주문 테이블

가정: 주문 테이블에는 주문 ID, 고객 ID, 주문 날짜, 주문 상태 등의 열이 있습니다. 주문 상태를 기반으로 세컨더리 인덱스를 생성한 상황입니다.

세컨더리 인덱스 예시:

세컨더리 인덱스: 주문 상태
주문 상태가 '배송 중'인 주문을 찾고자 할 때, 세컨더리 인덱스를 사용하여 해당 주문 ID를 찾을 수 있습니다.
그러나 데이터베이스에서 해당 주문의 고객 이름을 가져오려면 주문 테이블에 직접 액세스해야 합니다.
커버링 인덱스 예시:

세컨더리 인덱스: 주문 상태
커버링 인덱스: 주문 상태 + 고객 이름
이 경우, 세컨더리 인덱스에 주문 상태와 고객 이름을 함께 저장하면 주문 상태가 '배송 중'인 주문의 주문 ID를 찾을 때 고객 이름까지 함께 얻을 수 있습니다. 따라서 주문 테이블에 직접 액세스하지 않아도 됩니다.

---

## 그렇다면 최적화를 하지 않은 상태에서의 객체 생성 및 쿼리 생성 시간의 차이를 Test해보자.
#### Bulk Data를 10,000,000건 넣을 때를 기준으로 생각을 해보자. 속도 측정은 Spring의 StopWatch 메서드를 사용하여 진행.
#### EasyRandom을 이용하여 진행(https://github.com/j-easy/easy-random/wiki)

# Test
```
@SpringBootTest
public class PostBulkInsertTest {
    @Autowired
    private PostRepsitory postRepsitory;

    @Test
    public void bulkInsert(){
        var easyRandom = PostFixtureFactory.get(3L,
                LocalDate.of(2023, 9,3),
                LocalDate.of(2023, 9,6)

        );
        var stopWatch = new StopWatch();
        stopWatch.start();

       var posts =  IntStream.range(0, 1000000)
               .parallel()
                .mapToObj(i->easyRandom.nextObject(Post.class) )
                .toList();
        stopWatch.stop();
        System.out.println("객체 생성 시간 : " + stopWatch.getTotalTimeSeconds());

        var queryStopwatch = new StopWatch();
        queryStopwatch.start();

        postRepsitory.bulkInsert(posts);
        queryStopwatch.stop();

        System.out.println("쿼리 생성 시간 : " + queryStopwatch.getTotalTimeSeconds());


    }
}

```






#해볼것 : 회원 등록 인증 ( Spring securities, auth)
# 1-5 챕터 내용 정리 필요 





