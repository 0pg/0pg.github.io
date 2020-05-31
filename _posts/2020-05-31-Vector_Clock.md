---
title: "Vector Clock"
categories:
  - distributed
tags:
  - distributed system
  - vector clock
last_modified_at: 2020-05-31T00:00:00+09:00
toc: true
toc_sticky: true
---
* 분산 시스템에서 event 들의 부분 순서를 정의하고 인과관계를 위반을 탐지하기 위한 알고리즘
* Vertor clock 은 N 개의 프로세스가 있을 때 N 개의 logical clock 의 배열로 표현된다
	* clock 의 초기값은 0부터 시작
	* 내부 이벤트가 있을 때마다 자신의 clock 값을 1 증가시킨다
	* 메세지를 전송할 때마다 자신의 clock 값을 1 증가시키고, 자신이 가지고 있는 vector clock 복사본을 보낸다
	* 메세지를 수신할 때마다 자신의 clock 값을 1 증가시키고 수신한 vector clock 과 자신의 vector clock 모든 요소들에 대해 각각 더 큰 값으로 업데이트 한다
* event `a, b` 와 Vector Clock `VC(a), VC(b)` 에 대해 
	* `VC(a)[k] <= VC(b)[k] for k = 1 to N` 이면서 `VC(a)[k] < VC(b)[k] 인 k가 적어도 하나 존재한다` 를 만족하는 경우 `a < b` 임을 보장한다

> Lamport timestamp 와의 차이
> 
> Lamport timestamp 는 송신 프로세스의 logical clock 상태만을 전송한다.
> event `a, b` 와 clock `C(a), C(b)` 에 대해 `a < b -> C(a) < C(b)` 를 보장하지만, 그 역(`C(a) < C(b) -> a < b`)은 그렇지 않다 (모든 event 의 causality 를 파악할 수 없다)
