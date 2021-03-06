RDD
====
분산되어 존재하는 데이터 요소들의 모임

스파크의 모든 동작은 
 - 새로운 RDD를 만들기
 - 존재하는 RDD변경
 - 결과 계산을 위해 RDD에서 연산을 호출

중에 하나로 표현 

내부에서는 파이크가 자동으로 RDD에 있는 데이터들을 클러스터에 분배하며 클러스터 위에서 수행하는 연산들을 병렬화

RDD 기초
====
개념
----
- 분산되어 있는 변경 불가능한 객체들의 모음
- 각 RDD는 클러스터의 서로 다른 노드들에서 연산 가능하도록 여러 개의 파티션으로 나뉜다. 
RDD는 사용자가 정의한 클래스를 포함한 파이썬, 자바, 스칼라의 어떤 타입의 객체든 가질 수 있다. 

RDD 만드는 방법
---
- 외부 데이터 세트를 로드하는 방법
- 드라이버 프로그램에서 객체 컬랙션(list나 set)을 분산시키는 방법

만들어진 RDD는 두가지 타입의 연산을 지원 action과 transformation 임 

action & transformation
-----
- transformation은 존재하는 RDD에서 새로운 RDD를 만듬
    - val pythonLines = lines.filter(line => line.conttains("Python"))  <"Python"을 포함하는 모든 라인을 lines에서 뽑아서 pythonLines로 만듬>

- action은 RDD를 기초로 결과 값을 계산하여 그 값을 드라이버 프로그램에 되돌려 주거나 외부 스토리지에 저장(예: HDFS)
    - pythonLines.first()

lazy evaluation
====
RDD가 생성되는 시점은 처음 액션을 사용하는 시점으로 액션이 실행되기 전에는 선언만 해놓는다. 
- 입력하자마자 파일의 모든 라인을 로드해서 저장해 놓았더라면 상당한 스토리지 공간의 낭비가 발생
- 당장에 수많은 라인을 필터링해야 할 처지가 됨
- 대신에 스파크가 한번에 모든 트랜스포메이션끼리의 연계를 파악한다면 결과 도출에 필요한 데이터만을 연산하는 것이 가능하다. 
    - 예: first()엑션에서도 처음에 일치하는 라인까지만 읽을 뿐 이후는 읽지 않음
- 스파크의 RDD들은 기본적으로 액션이 실행될 때마다 매번 새로 연산을 한다. 만약 여러 액션에서 RDD하나를 재사용하고 싶으면 스파크에게 RDD.persist()를 사용함
    - persist는 계속 결과를 유지하도록 요청할 수 있다. (메모리, 디스크)에 데이터를 보관해 주도록 스파크에 요청 가능
    - 첫 연산이 이루어진 후 스파크는 RDD의 내용을 메모리에 저장하게 됨(클러스터의 여러 머신에 나눠서) 이후의 액션들에서 재사용한다. 메모리 대신 디스크에 RDD를 저장하는 것도 가능하다. 
    - 데이터를 계산만 하고 저장을 하지 않는 경우도 있다. (일회성 데이터) --> 굳이 스토리지 낭비할 이유가 없어서

예제 3-4 메모리 RDD 보존하기
------------
pythonLines.persist()

pythonLines.count()
 
pythonLines.first()

1. 외부 데이터에서 RDD를 만든다. 
2. filter 같은 transformation을 써서 새로운 RDD를 정의
3. 재사용을 위한 중간 단계의 RDD들을 보존하기 위해 스파크에 persist()를 요청


RDD 생성하기 
====
RDD를 만드는 두가지 방법
- 외부 데이터세트의 로드
- 직접 만든 드라이버 프로그램에서 데이터 집합을 병렬화하는 것을 제공

가장 간단한 방법은 데이터세트를 가져다가 sc의 parallelize()메소드에 넘겨 주는 것
(이 방식은 하나의 머신 메모리에 모든 데이터세트를 담고 있어서 잘 안쓰임)
- lines = sc.parallelize(["pandas", "i like pandas"])

RDD 연산
=====
- transformation, action
- map(), filter() 과 같이 새로운 RDD를 만드는 것이 transformation
- 드라이버 프로그램에 결과를 돌려주거나 스토리지에 결과를 써넣는 연산 count(), first()가 action이다. 
- tranformation은 return타입이 rdd이고 action은 이외의 것

transformation
====
transformation된 RDD의 계산은 다소 늦게 계산된다.(lazy evaluation)
- RDD는 변경 불가능한 것(불변)
- 왠만해서 하나의 요소씩 업데이트된다. (elementwise)
- 꼭 모든 transformation이 그런건 아님

union
----
val inputRDD = sc.textFile("log.txt")
val errorRDD = inputRDD.filter(line => inputRDD.contain("error"))
val warnRDD = inputRDD.filter(line => inputRDD.contain("warn"))
val badRDD = errorRDD.union(warnRDD)

RDD의 가계도(lineage graph)
-----
spark는 RDD에 대해 관계 그래프를 가지고 있다. 스파크는 이 정보를 필요 시 각 RDD를 재연산하거나 저장된 RDD가 유실될 경우 복구를 하는 등의 경우에 활용한다. 

action
====
결과 값을 돌려 주거나 외부 저장소에 값을 기록하는 연산 작업. transformation이 계산을 수행하도록 강제한다. 

예제 3-16

RDD의 데이터 일부를 가져오기 위해 take()를 사용. 그리고 프로그램의 정보를 출력하기 위하여 로컬에서 반복 작업을 수행. RDD의 전체 데이터를 가져올 수 있는 collect()도 있다. 

take vs collect
---
take
- RDD를 filter와 같은 작은 크기의 데이터 세트의 RDD로 만든 후 분산이 아닌 로컬에서 데이터를 처리하고 싶을 때 유용하다. 

collect
- 전체 데이터 세트의 크기가 단일 컴퓨터의 메모리에 올라올 수 있을 정도의 크기여야 하고 너무 크면 사용 할 수 없음


Lazy execution
===
- action을 만나기 전까진 transformation을 수행하지 않음. 
- 메타 데이터에 요청되었다는 사실만 기록. 
- RDD에 데이터를 로드하는 것도 여유롭게 수행.
    - sc.textFile()을 수행하면 실제로 필요한 시점이 되기 전까지는 로딩되지 않음

하둡 맵리듀스 vs 스파크
----
하둡 맵리듀스 
- 어떤 식으로 연산을 그룹화 할지 고민하는 것이 관건 
    - 연산 개수가 많다는 것은 곧 네트워크로 데이터를 전송하는 단계가 많다. 

스파크
- 단순한 연산ㄴ들이 많이 연결해서 사용하는 것이나 하나의 복잡한 매핑 코드를 쓰는 것이나 큰 차이가 없다. 
- 프로그램을 더 작게 만들고 효율적인 연산의 코드를 만들어 내야 한다는 부담에서 자유롭다. 

스파크에 함수 전달하기
====
transformation과 action의 일부는 스파크가 실제로 연산할 때 쓰일 함수들을 전달해야 하는 구조를 가진다. (언어마다 전달하는 구조는 다름)

파이썬 
-----
주의 할 점으로 무의식적으로 함수를 포함한 객체를 직렬화하는 것. 객체의 맴버인 함수를 전달하거나 객체의 필드 참조를 갖고 있는 함수를 전달 할 때 노드의 전체 객체가 전달되므로 필요 이상으로 거대한 정보가 전달 될 수 있다.

예제 3-19)

스칼라
----
스칼라의 경우도 객체의 메소드나 필드를 전달할 때 전체 객체에 대한 참조 또한 포함된다. 따라서 필요 변수만 추출해서 보낸다. 

예제 3-21)

스칼라에서 NotSerializableException 에러가 뜨면 직렬화 불가능한 클래스의 메소드나 필드를 참조하는 문제일 가능성이 많다. 최상위 객체의 멤버인 지역 변수나 함수 내에서 전달하는 것은 항상 안전하다.

많이 쓰이는 트랜스포메이션과 액션
===
특별한 데이터 타입을 취급하는 RDD를 위한 추가적인  연산들도  존재.

기본 RDD
---
모든 데이터에 사용할 수 있는 RDD

데이터 요소 위주 트랜스포메이션
- map: 함수를 받아 RDD의 각 데이터에 적용하고 새  결과 값을 담음
    - 입력타입과 반환타입이 같지 않아도 되어 URL을  받고 그 데이터를 받아오는 작업 등 다양하게 쓰인다. 
    - 예제 3-27
    - 여러 개의 아웃풋으로 나오는 경우(1대1 대응이 아닌 경우) flatmap을 사용한다. 
        - 예제 3-30
- filter: 함수를  받아 통과한 데이터만 담는다. 

가상 집합 연산
----
union, intersect, distinct, subtract는 RDD가 서로 같은 타입이어야 함

distinct
---
유일성(uniqueness)를  보장하기 위해선 distinct를 사용
-  단 이는 네트워크로 데이터를 이리 저리 보내야 하므로 코스트가 비싼 연산임
- 이런 셔플링(shuffling)을 피하는 방법은 4장에

union
---
양쪽의 데이터를 합하여 돌려줌
- 중복은 유지

intersection
---
공통인 요소만 전달
- 동작하면서 모든 중복을 제거(원래 있던 중복 포함)
- distinct처럼 셔플링이 수반되기 때문에 성능이 떨어짐

subtract
---
RDD에서 다른 RDD 를 받아서 같은 것이 있으면 제거한다. 
- 셔플링이 있으므로 코스트가 쌤

cartesian
---
RDD두개에 대하여 가능한 모든 쌍을 만들어 돌려준다. 
- 큰 RDD에 대해서는 비용이 매우 큰 작업임

action
====
reduce
- 인자 2개를 함수로 받아서 두개의 데이터를 합쳐 같은 타입 데이터하나를 반환하는 함수 

fold
- reduce에 전달되는 것과 동일한 형태의 함수를 인자로 받고 거기에 추가로 각 파티션의 초기 호출에 쓰이는 "제로 벨류(zero value)"를 인자로 받는다. 

aggregate
- 동일한 데이터 형식을 리턴할 필요없음
- 리턴 타입에 맞는 제로 벨류가 필요하다. 
- RDD의 값들을 누적값에 연계해 주는 함수가 필요
- 두개의 누적값을 합쳐주는 두 번째 함수가 필요

collect
- 테스트하기 좋음
- 모든 데이터 셋을 메모리에 올릴 수 있어야함

take
- 접근하는 파티션 갯수가 최소가 되도록 하기 때문에 특정 파티션의 값들만 되돌려 줄 수 있다. 
- 즉 기대하는 순서대로 값을 돌려주지 않음

이들은 병목현상을 일으킬 수 있다. 

top
---
- 데이터의 특정 순서가 정의 되어 있는 경우 사용
- RDD에서 상위 값들만 뽑아옴
- default는 내림차순

takeSample
---
표본추출 함수는 복원 혹은 비복원 추출로 표본을 추출하게 해 준다. 

countByValue
---
RDD에 있는 각 값의 개수 리턴 

takeOrdered
---
제공된 ordering 기준으로 num개 값 리턴

foreach
---
각 값에 대하여 function 적용

RDD 타입 간 변환하기
===
수치형 RDD <DoubleRDDFunctions>
- mean(), variance()

키/값 페어 RDD <PairRDDFunctions>
- join()

영속화(캐싱)
===
여러번 같은 연산을 반복하지 않기 위해선 스파크에 데이터 영속화(persist/persistence)요청을 할 수 있다. 

노드에 장애가 생기면 유실 데이터 파티션을 재연산 만약 지연 없이 노드 장애에 대응하려면 복제하는 정책을 사용

스칼라에서는 질렬화하지 않은 객체 형태로 데이터를 저장 

데이터를 디스크나 오프힙 저장 공간에 쓸때는 데이터가 늘 직렬화된다. 

스토리지 레벨 종류
- MEMORY_ONLY
- MEMORY_ONLY_SER
- MEMORY_AND_DISK
- MEMORY_AND_DISK_SER
- DISK_ONLY

persist
---
이 인스트럭션을 쓰는 순간 메모리 혹은 디스크가 생기는 듯
- 정책은 lru


unpersist
---
메모리 혹은 디스크를 삭제
