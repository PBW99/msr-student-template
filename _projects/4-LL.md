---
layout: project
title: (Java) Parallel BST & Lockfree LinkedList
date: December 10, 2017
image: /portfolio/public/images/4.lockfreeLL.png
permalink: "project-4.html"
---
## Overview
> 학교 과제로 주어진 Parallel Binary Search Tree와 LockFree Linked List구현입니다. Java로 구현했으며 Linux 프로그램 성능 분석 도구인 Perf를 이용하였습니다.

### Parallel Binary Search Tree
> 하나의 Binary Search Tree에 다수에 스레드가 접근할 수 있도록 만들어서 Parallel Binary Search Tree입니다.
> 다수의 스레드가 접근할 수 있도록 Fine-grained lock으로 구현했습니다. 그 중에 Hand-over-Hand lock을 이용하였습니다.
방법은 다음과 같습니다.
> > (C(Cursor) : Target을 찾기 위해 BST내부를 순회하는 포인터, 
P_C : Cursor의 부모,)

1.	제일 먼저 C를 root로 잡고 lock을 잡는다. 
2.	찾아야하는 데이터의 값에 맞는 방향으로 C를 이동하고 lock을 잡는다. 이에 맞추어 P_C도 세팅한다. 
(이것이 기본적인 BST hand-over-hand lock의 시작이다. )
3.	두 개의 lock을 잡은 스레드는 다음으로 C가 가야할 노드를 찾는다. 찾으면 다음과 같은 행동을 취한다.
4.	P_C를 unlock
5.	C를 해당 노드로 옮기고 그 노드의 lock을 잡는다. 
6.	C가 T가 될 때까지 (3~5)반복
7.	C가 T가 될 경우, 적합한 루틴 수행 후 P_C와 C를 unlock(Insert 인 경우 이미 존재하므로 삽입하지 않고, unlock만 수행
7-1. C가 null(리프의 밑)이 될 경우, false를 반환, 반환 하기 전에 P_C를 unlock
	(Insert 인 경우 그 자리에 삽입),
	(C가 null일 경우 5를 수행하지 않는다. 따라서 P_C만 unlock)


### 결과 


### LockFree Linked List
![](/portfolio/public/images/4-ParBSTLFLL/ParBST-4-core-IO.png){: width="480" height="320"}
![](/portfolio/public/images/4-ParBSTLFLL/ParBST-8-core-IO.png){: width="480" height="320"}


## 사용 언어 / 도구
* Java
* Eclipse
* Shell Script
* (linux) perf


## 테스트 영상
{% include youtube_embed.html id="GOOLeYomCxM" %}  

## 소스코드
> [github](https://github.com/PBW99/HYU_3rd-2se/tree/master/SoftwareEngineering)