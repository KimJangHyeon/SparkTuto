로깅레벨 낮추기
----
spark/conf/log4j.properties.template에서 log4j.rootCategory=WARN으로 

rdd(Resilient Distributed Dataset) 
---
나중에 배움

스파크의 핵심 개념 소개
----
스파크 APP는 클러스터에서 다양한 병렬 연산을 수행하는 드라이버 프로그램으로 구성됨

연산 클러스터에 대한 연결을 나타내는 SparkContext객체를 통해 드라어버 프로그램들은 스파크에 접속한다. 

sc(SparkContext)를 만들었으면 RDD를 만들 수 있다. 

드라이버 프로그램은 다수의 노드를 관리함(노드: 클러스터의 머신 하나를 표현하는 의미)


