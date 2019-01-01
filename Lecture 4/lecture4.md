<h3>키/값 페어에  작업하기</h3>

intro
===
- ETL(시스템 간에 대량 데이터를 추출해 다른 시스템에 적재하는 작업)작업에도 이 방식이 쓰임
- 파티셔닝 활용법

페어 RDD 생성
===
map을 사용하여 만든다. [?, ?, ?] 형태로 만들 수 잇으니까
- eg) val pairs = lines.map(x => (x.split(" ") (0), x))
- ._1() / ._2()로 요소에 접근 가능

페어 RDD 트렌스포메이션
===
한 RDD에 대해서
---
- reduceByKey
- groupByKey
- combineByKey(
createCombiner,
mergeValue,
mergeCombiners,
partitioner)
- mapValues(func)
- flatMapValues(func)
- keys()
- values()
- sortByKey()

두 RDD에  대해서 
---
- subtractByKey
- join
- rightOuterJoin
- leftOuterJoin
- cogroup

두번째 요소에 대한 단순 필터 적용
- eg)pairs.filter{case (key, value) => value.length < 20}

mapValues(func) vs map()
----
mapValues 
- ret: key, (val1, val2, val3)

map
- ret: key, val1, val2, val3

인듯


reduceByKey 와 reduce 유사

foldByKey 와 fold 유사

단어 세기
---
eg) 

- val input =  sc.textFile()
- val words = input.flatMap(x => x.split(" "))
- val count = words.map(x => (x, 1)).reduceByKey((x, y) => x + y)

키에 의해 데이터를 병합하는 여러가지 방법이 있다. 대부분 combineByKey를 사용하여 구현하지만 더욱 간단한 인터페이스도 제공한다. 어떤 방법을 선택하든 스파크에서 제공하는 특화된 집합 연산 함수들을 사용하는 것보다 훨씬 빠르다. 

병렬화 수준 최적화
===
모든 RDD는 고정된 개수의 파티션을 갖고 있으며 이것이 RDD에서 연산이 처리될  때 동시 작업의 수준을 결정하게 된다. 

대개의 경우,  클러스터의 사이즈에 맞는 적절한 파티션 개수를 찾는 방식으로 동작하지만, 특별한 경우에는 더 나은 퍼포먼스를 내기 위해 병렬화의 수준을 직접정해 주어야 하는 경우도  있을 수 있다. 

eg)
- val data = Seq[("a", 3), ("b", 4), ("c", 1)]
- sc.parallelize(data).reduceByKey((x, y) =>  x + y)
- sc.parallelize(data).reduceByKey((x, y) => x + y, 10) //병렬화 수준 정의

repartition
---
그룹화 작업 코드 범위의 바깥에서 파티셔닝을 바꾸고 싶을때 쓸 수 있지만 데이터 교환이 일어나 코스트가 큰 작업임

coalesce
---
repartition의 최적화 버전으로  파티션을 줄이는  한에서만 셔플링이  발생하지 않는다. 
- 제대로 동작할 수 있는지 알고 싶다면 rdd.partition.size로 파티션 개수를 파악해 더 적은 파티션을로 합칠 수 있는지 확인 가능

데이터 그룹화
===
groupBy_()
---
쌍을 이루지는 않았으나 현재 키와 관계되지 않은 다른 조건을 써서 데이터를 그룹화 할때 쓰임. (K, [V])

cogroup
---
키에 대해서 value를 그룹화 할 수 있다. (K, ([V], [W]))


조인
===
다른 키를 가진 RDD와 함께 사용함
- 모든 범위의 조인 다 지원


데이터 정렬
===
- 다운스트림 데이터를  생성하는 경우에 유용하다. 
- 키에 대해 순서가 정의되어 잇는 경우 정렬 가능
- collect, save 함수 호출은 결과가 정렬되어 나오게 됨

eg) p74 4-20


페어 RDD에서 쓸 수 있는 액션
===
- countByKey()
    - key의 갯수와 페어로 
- collectAsMap()
    - 쉬운 검색을 위해  결과를 맵 형태로 모은다.
- lookup(key)
    - 들어온 key에 대해서 value들을 리턴



데이터 파티셔닝(고급)
===
파티셔닝을 어떻게 제어할지
네트워크 부하 줄이는 선에서 

파티셔닝(해시 파티셔닝, 레인지 파티셔닝)
---
조인 같은 키 중심의 연산에서 데이터 세트가 여러번 재활용될 때만 의미가 있지 한번만 스케닝 될때는 의미가 없다. 

키의 모음들이 임의의 노드로 가는 것이 보장되도록 해준다. 

eg) 예제 4-22
- join이 데이터세트에서 키가 어떻게 파티션 되었는지 모르기때문에 이 연산은 양쪽 데이터 세트를 모두 해싱하고 동일 해시 키의 데이터끼리 네트워크로 보내 동일 머신에 모이도록 한후 해당 머신에서 동일한 키의 데이터끼리 조인을 수행한다. 

해결책(partitionBy)
----
- 이를 사용하여 미리 파티션을 나눈 데이터를 전달하면 이를 해결할 수 있다. 미리 나뉘어져 있는 파티션만 비교하면 됨 
- 트렌스포메이션임 
- sequenceFile의 결과가 아닌 partitionBy의 결과를 영속화하고 저장해야 한다. 
- 파티션 개수(얼마나 많은 병렬 작업들이 RDD에서 이후에 이루어질 연산 작업을 수행하는지 나타냄)는 최소한 클러스터의 전체 CPU코어 개수 이상이 되도록 한다.
    
RDD의 파티셔너 정하기
===
RDD가 어떤 파티션이 될지 partitioner 속성을 써서 결정 가능

isDefined
---
값이 있는지 체크

get
---
값이 있따면 값을 받아올 수 잇음



파티셔닝이 도움이 되는 연산들
===
- cogroup, groupWith, join, leftOuterJoin, right.., groupByKey, reduceByKey, combineByKey, lookup

reduceByKey
--- 
- 이미 파티셔닝 되어있는 경우 각 키에 관련된 모든 값들을 단일 머신에서 처리하며, 각 작업 노드에서 병합된 최종 결과 값은 마스터 노드로 다시 보내진다. 

cogroup, join
---
- 최소 하나이상의 RDD가  셔플될 필요가 없게 해 준다.
- 만약 두 RDD가 동일한 파티셔너를 갖고 동일한 머신들에 캐시되어 있거나 둘중 하나가 연산이 아직 완료되지 않았다면 네트워크를 통한 셔플은 발생하지 않는다. 


파티셔닝에 영향을 주는 연산들
===
데이터를 파티션하는 연산에 의해 만들어진 RDD에는 자동적으로 partitioner를 세팅함. 

- 즉 파티션된 값을 이용해 join을 한 결과 join도 partitioner를 갖고 있다 
    - 파티셔너가 지정되는 연산들
        - cogroup, groupWith, join, leftOuterJoin, right.., groupByKey, combineByKey, partitionBy, sort,  
        
    - 부모가 파티셔너를 갖는 경우
        - mapValues, flatMapValues, filter

- 하지만 map같은 경우 데이터가 변할 가능성이 높아 partitioner를 갖지 않는다. 


예제: 페이지 랭크
===
얼마나 많은 문서를 링크하고 있는지에 기초하여 각 문서에  랭크를 매기는 것

많은 조인을 수행하는 반복 알고리즘으로 RDD파티셔닝에  대한 좋은 사례 
- 두종류의 데이터세트를 관리함
    1. (pageID, link list)
        - 페이지 아이디와  그이웃 페이지를 가지고 있음
    2. (paageID, rank)
        - 페이지와 그 랭크
    
    이다. 

알고리즘
1. 각페이지를 랭크 1.0으로 초기화
2. 매번 반복 주기마다 페이지 p는 공헌치 랭크(p)/이웃숫자(p) 를 이웃들에게  보낸다. 
3.  각 페이지의  랭크를 0.15 + 0.85*(받은 공헌치)로 갱신한다. 

대략  10번의 iter



사용자 지정 파티셔너
===
파티셔닝을 잘해야 통신 비용을 줄일  수 잇다. 

url에 대해서 해싱을 하면 비슷한 것이 다른 노드로 가버릴 수도 있으므로 
사용자 지정 파티셔닝을 하여 서로 완전히 다른 노드에 해싱되어  버릴 경우를 배제할 수 있도록 도메인 네임에 대해서만 파티셔닝을 한다.

 eg)p85 4-26

질문
=====
- 파티셔닝은 그러면 하나의 데이터를 범위에 따라 나눠서 스케닝은 안하는지?(collect같은 경우)
- p76,77 예제 4-22, 예제 4-23은 네트워크 코스트는 같지 않나?
    - 어차피 조인할때 파티셔닝해서 보내긴한다매