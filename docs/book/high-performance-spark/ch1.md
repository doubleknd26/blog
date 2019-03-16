---
layout: post
title: ch1. Introduction to High Performance Spark
nav_order: 1
parent: High Performance Spark
permalink: /docs/book/high-performance-spark/ch1
---

# Introduction to High Performance Spark
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

<br>

## 스파크는 무엇인가
> Apache Spark is a fast and general-purpose cluster computing system. - [apache spark](https://spark.apache.org/docs/latest/#spark-overview)

- 아파치 스파크는 범용 목적의 고성능 분산 처리 시스템이다.
- 아파치 스파크는 일반화된 병렬 처리로 데이터를 다룰 수 있는 수단을 제공하는 오픈소스 프레임워크다.
  - 즉, 동일한 고수준의 스파크 API들로 크기와 구조가 다른 여러 가지 데이터에 대해 서로 다른 데이터 처리 작업을 수행할 수 있다.

<br>

## 스파크 버전 규칙
스파크는 표준처럼 쓰이는 [메이저].[마이너].[유지 보수] 형식의 의미론적 버전 번호 규칙을 따른다.
- [apache spark github](https://github.com/apache/spark/releases)
- [apache spark release note](https://spark.apache.org/releases/)

<br>

## 왜 스칼라인가

- 스파크는 스칼라로 쓰였기 때문에 스칼라 코드를 읽을 수 없다면 스파크 소스 코드를 이해하기 어렵다.
  - 게다가 RDD 클래스들의 메서드들은 스칼라 컬렉션 API의 구조와 거의 동일하다. e.g) map(), filter() ..
  - 본질적으로 스파큰느 함수형 프레임워크이며, 불변성(immutability)이나 람다 정의 같은 개념에 매우 크게 의존하고 있기 때문에 함수형 프로그래밍에 대해 기본적인 지식이 있다면 스파크 API를 좀 더 직관적으로 이해할 수 있다.
- 상대적으로 사용하기 쉬운 API를 제공한다.
  - 자바보다 훨씬 간결하다. 이는 스파크가 많은 부분에서 인라인 함수 정의나 람다 표현식을 사용하고 있기 때문이다.
  - spark shell은 디버깅이나 개발에 있어서 갈력한 도구인데, 이는 자바를 제외한 REPL[^1]을 지원하는 언어들로만 사용할 수 있다.
- 스칼라는 파이썬보다 성능이 뛰어나다.
  - 파이썬은 JVM과의 통신 비용이 든다.



<br><br><br>

[^1]:REPL(Read-Eval-Print Loop), 프로그래밍 언어에서 라인 단위로 실행해 볼 수 있는 셸을 제공한다면 REPL을 지원한다고 볼 수 있다.





