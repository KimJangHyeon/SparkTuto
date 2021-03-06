세가지 장점
1. 쓰기 쉽다. 
2. 대화형 작업, 복잡한 알고리즘을 사용하게 해줌 + 빠름
3. 각기 다른 엔진을 쓰던 여러 타입의 연산(SQL znjfl, 텍스트 처리, 머신 러닝)을 한군데서 돌릴 수 있는 범용 엔진

아파치 스파크란 무엇인가?
------------------------
빠른 속도로 작업을 수행할 수 있도록 설계한 클러스터용 연산 플랫폼

맵리듀스 모델을 (스트리밍, 대화형, 명령어 쿼리)가 가능하도록 확장함 

메모리가 아닌 디스크에서 돌더라도 맵리듀스 보다는 빠름

배치 어플리케이션, 반복 알고리즘, 대화형 쿼리, 스트리밍 같은 다양한 알고리즘을 동시에 커버 가능하도록 설계 
이러한 워크로드(workload)들을 단일 시스템에서 지원하게 됨에 따라 스파크는 실제의 데이터 분석 파이프라인(pipeline)에 
서로 다른 형태의 작업을 쉽고 저비용으로 연계를 할 수 있음

유지보수 --> 관리 비용 감소

통합된 구성
------
밀접하게 연동된 여러 개의 컴포넌트로 구성

스파크는 다수의 작업 머신이나 클러스터 위에서 돌아가는 많은 연산 작업 프로그램을 스케줄링하고 분배하고 감시하는 역할을 한다. 

스파크의 핵심 엔진은 빠르고 범용적이라 SQL이나 머신 러닝같은 다양한 워크로드에 특화된 여러 고수준 컴포넌트를 실행할 수 있게 해준다. 

이는 사용자가 컴포넌트들을 연동해 쓸 수 있게 상호 호환되도록 설계되어있다. 

이러한 밀접하게 컴포넌트를 연동하는 방식의 장점 
- 하위레이어의 성능향상에 의해ㅐ 고수준 컴포넌트들이 직접적인 이익을 볼 수 있다. 
 - 운영비용 최소화(여러 개 돌리는 대신 하나만 돌리면 됨)
- 서로 다른 데이터 처리 모델을 깔끔하게 합쳐서 하나의 app로 만들 수 있다
- 오직 하나의 시스템만 관리하면 됨

스파크 코어 
----
구성: 작업 스케줄링, 메모리 관리, 장애 복구, 저장 장치와의 온동 등등 

RDD(탄력적인 분산 데이터세트)

스파크 SQL
----
JSON, 파이케이, 하이브 테이블 emddmf wldnjs
SQL -> data warehousing도구가 뛰어남


스트리밍
----
도착하는 순서대로 데이터를 처리 할 수 있다. 

상태 업데이트 메시지가 저장되는 queue, log file 등이 이에 해당

RDD API와 저의 질치하는 형태의 데이터 스트림 조작 API를 지원함 --> 빠르게 익숙해짐

스트림방식이든 메모리 디스크에 있는 데이터를 받는 방식이든 혼란이 없어짐

스파크 코어와 동일한 수준의 장애 관리, 처리량, 확정성을 지원

MLlib
---
ML의 핵심 기능 지원

경사 강하 최적화 알고리즘 지원

그래프 X
---
SNS의 친구 관계 그래프 

간선이나 점에 임의의 속성을 추가한 지향성 그래프를 만들 수 있다. 

클러스터 매니저
---
유연성을 극대화 --> 하둡의 얀, 아파치 메소스 단독 스케쥴러 위에서 동작 

