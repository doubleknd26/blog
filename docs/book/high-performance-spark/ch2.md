---
layout: post
title: ch2. How Spark Works
nav_order: 2
parent: High Performance Spark
permalink: /docs/book/high-performance-spark/ch2
---

# How Spark Works
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 스파크는 빅데이터 생태계에서 어떻게 자리 잡고 있는가?

![spark ecosystem](https://cazton.com/Content/images/consulting/apache-spark-consulting/apache-spark-ecosystem.jpg)
[image1] spark echosystem[^1]


스파크는 하나의 단일 머신 위에서 한 JVM만으로도 실행할 수 있다.(standalone mode) 하지만, 분산 저장 시스템와 클러스터 매니저가 함께 쓰이는 경우가 대부분이다.
- 저장 시스템은 스파크에서 처리한 데이터를 보관하는 역할을 한다. e.g) HDFS, Cassandra, S3.
- 클러스터 매니저는 클러스터 전반에서 스파크 애플리케이션의 분산 처리가 원활하게 이루어질 수 있도록 도와준다. e.g) Standalone Cluster Manager, Apache Mesos, Hadoop Yarn.

<br>

### Spark Components
- Spark Core
  - 스파크 생태계에서 핵심적인 데이터 처리 프레임워크. 스칼라, 자바, 파이썬, R로 API가 각각 제공된다.
- Spark SQL
  - Spark Core와 함께 쓰이면서 스칼라, 자바, 파이썬, R API를 포함해 기본적인 SQL 질의를 지원한다.
  - Spark Core와는 다른 질의 최적화 엔진을 갖고 있다.
  - Dataset이라는 이름의 semi-structured 데이터 타입을 지원한다.
  - 스파크 코어만으로 할 수 있는 일도 많은 부분 스파크 SQL과 함께 적용해서 성능을 극대화시킬 수 있다.

그 밖에도 퍼스트 파티 컴포넌트로 Spark MLlib, Spark Streaming, Graph X 등이 있다.

<br>
## 스파크의 병렬 연산 모델: RDD
> Spark revolves around the concept of a resilient distributed dataset (RDD), which is a fault-tolerant collection of elements that can be operated on in parallel. - [apache spark](https://spark.apache.org/docs/latest/rdd-programming-guide.html#resilient-distributed-datasets-rdds)

- 스파크는 대용량 데이터셋을 RDD로 표현하는데 이들은 executor 혹은 slave node에 저장된다.
- RDD를 구성하는 객체를 partition이라고 한다.
- 스파크 클러스터 매니저는 스파크 애플리케이션에서 설정한 파라미터에 따라 분산 시스템에 executor들을 실행하고 분산해주는 역할을 한다.
- 스파크 실행 엔진은 연산을 위해 executor들에게 데이터를 분산해 주고 실행을 요청한다.

<br>

### 지연 평가 (Lazy Evaluation)
- RDD는 지연 평가 방식(lazy evaluation)으로 동작하기 때문에 액션이 호출되기 전까지는 계산되지 않는다.
- 액션은 스케줄러를 시작하게 하며 스케줄러는 RDD 트랜스포메이션 간의 종속성에 바탕을 둔 DAG를 생성한다. 

**장점**
- 성능상의 장점으로는 드라이버와 통신할 필요가 없는 연산들을 모아서 한번에 처리할 수 있다.
  - e.g) map()과 filter() 함수를 executor에서 한번에 처리할 수 있다.
- 구현상의 장점으로는 개발자는 직업 연관성 있는 연산들을 함께 엮어놓기만 하면 이들을 합쳐 주는 복잡한 작업은 스파크가 알아서 처리해준다.

**지연 평가와 장애 내구성**
- 지연 평가를 하기 때문에 (즉, 액션이 발생되는 시점에 모든 연산이 스케줄링되고 DAG가 생성되기 때문에) 각 파티션은 어떠한 이유에서 연산에 실패해도, 자신이 가지고 있는 종속성 정보 데이터 (DAG 등)를 사용하여 재계산할 수 있다.

<br>

### 메모리 영속화와 메모리 관리
![spark memory](https://blog.cloudera.com/wp-content/uploads/2015/03/spark-tuning2-f1.png)  
[image2] spark memory[^2]

- 맵리듀스와 비교해 스파크의 성능상 이점은 반복 연산이 들어있는 사례에서 상당한 우위를 보인다. 이 성능 향상의 많은 부분은 스파크가 메모리 영속화를 활용하는 덕택이다.  
- 스파크는 메모리 관리에 대해 세 가지 옵션을 제공한다.
  - 직렬화되지 않은 객체
  - 직렬화된 객체
  - 디스크
    - executor 메모리에 담기 너무 큰 RDD라면 디스크에 데이터를 쓸 수 있다. 당연히 속도 면에서는 불리하다.
    - 오래 걸리는 트랜스포메이션들이 반복되는 경우, 가장 장애에 안전해야하는 경우, 막대한 양의 연산을 해야하는 경우에 사용할 수 있다.
- 새로운 파티션을 연산하거나 캐시하는 공간이 필요한 경우, 사용한지 가장 오래된 파티션을 삭제하는 것이(LRU) 기본 구현이다. 하지만 사용하자 메모리 우선순위를 제어할 수 있다.

<br>

### 넓은 종속성 vs 좁은 종속성
> coursera, [Wide vs Narrow Dependencies](https://www.coursera.org/lecture/scala-spark-big-data/wide-vs-narrow-dependencies-shGAX)

![wide vs narrow dependencies](https://image.slidesharecdn.com/sparkoverview-151003203756-lva1-app6892/95/ibm-spark-technology-center-realtime-advanced-analytics-and-machine-learning-with-spark-and-cassandra-12-638.jpg?cb=1443904732)  
[image3] wide vs narrow dependencies[^3]

좁은 종속성
- 자식 RDD의 각 파티션이 부모 RDD의 파티션들에 대해 단순한 종속성을 가지는 것이다.
- 부모 RDD의 파티션은 최대 한 개의 자식 RDD 파티션에서 사용된다.
- 자식 파티션은 부모 파티션의 집합을 파티션의 값에 상관없이 결정할 수 있다.

넓은 종속성
- 임의의 데이터만으로 실행할 수 없고, 다른 파티션의 정보를 필요로 한다.
- 부모 RDD의 파티션은 둘 이상의 자식 RDD 파티션에서 사용될 수 있다. 즉, 셔플이 필요하다.
- 자식 파티션은 부모 파티션의 집합을 파티션의 값에 따라 결정해야 한다.

<br>

## 스파크 잡 스케줄링

## 스파크 잡의 해부





<br><br><br>

[^1]: spark echosystem, https://cazton.com/consulting/big-data-development/apache-spark
[^2]: spark memory, https://blog.cloudera.com/blog/2015/03/how-to-tune-your-apache-spark-jobs-part-2/
[^3]: wide vs narrow dependencies, https://image.slidesharecdn.com/sparkoverview-151003203756-lva1-app6892/95/ibm-spark-technology-center-realtime-advanced-analytics-and-machine-learning-with-spark-and-cassandra-12-638.jpg?cb=1443904732