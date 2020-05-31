---
title: "Conflict Free Replicated Data Types"
categories:
  - distributed system
tags:
  - distributed system
  - CRDTs
last_modified_at: 2020-05-31T00:00:00+09:00
toc: true
toc_sticky: true
---
* 분산 시스템에서 각 노드사이에 **데이터를 공유**하는 방법
* 데이터 복제는 **동시적**이며 **독립적**으로 수행될 수 있다

## 배경
* 데이터 업데이트를 위한 노드들 간 특별한 조정 없이 여러 복제본에 동일한 데이터 동시 업데이트를 하기 위해서는 해결해야 할 여러 문제점이 있었다
	* 복제본 사이 데이터 일관성이 깨질 수 있다
	* 이 경우 데이터 무결성과 일관성을 복구하려면 최악의 경우 전체를 업데이트 하거나 일부를 버려야 한다
* *optimistic replication*(낙관적 복제)라는 개념이 등장
	* 일관성이 깨지더라도 우선 동시 업데이트 수행을 허용한다
	* 업데이트가 완료된 후 결과들을 통합해가며 일관성 문제를 해결한다
	* 위의 과정을 통해 복제본들은 일관성을 갖게된다
* 분산 시스템에서 자주 사용되는 방식이다

### 충돌 해결
* 불일치를 허용하기 때문에 서로 다른 복제본이 함께 존재할 수 있다
* 불일치 해결을 위한 조건
	* 데이터의 version 정보를 교환해야 한다
	* 복제본의 가장 **적절한** 최종 상태를 결정해야 한다
* **적절한** 상태는 경우에 따라 다르겠지만 결국 위의 두 가지 조건을 만족하면 최종적으로 복제본들은 데이터 일관성을 가질 수 있게 된다

## CRDTs 종류
* *strong eventual consistency* 를 제공하는 두 가지 접근 방법이 있다
	* Operation based CRDTs
	* State based CRDTs
* 각자 서로를 구현할 수 때문에 이론적으로는 동일하다고 볼 수 있다

### Operation-based CRDTs
* update 연산만 전파한다
* communication middleware 을 필요로 한다
* 교환법칙이 성립하는 복제 데이터 타입
    * 복제 간 전파되는 연산의 순서는 상관없다
* 멱등성을 보장할 필요는 없다
    * 복제 간 전파되는 연산은 중복이 있어선 안된다

### State-based CRDTs
* 설계과 구현이 상대적으로 더 간단하다
    * gossip protocol 기반의 통신 프로토콜만 요구됨
* 최종적으로는 CRDT 내 전체 상태를 전파해야하다는 비용이 존재한다
* local 의 전체 상태를 다른 복제에 전달한다
    * 교환법칙, 결합법칙, 멱등성을 만족하는 함수에 의해 통합(merge)된 상태
* *merge* 함수는 임의의 두 복제에 *join* 연산을 제공한다
    * 상태 집합은 *semi-lattice* 를 만족한다


> **strong eventual consistency**
>
> 데이터 복제에 대한 여러 연산 후 최종적으론 모든 복제본들이 일관성을 가지게 될 것임을 의미
