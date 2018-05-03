---
layout: project
title: (Java) Parallel BST & Lockfree LinkedList
date: December 10, 2017
image: /portfolio/public/images/4.lockfreeLL.png
permalink: "project-4.html"
---
## Overview
> 학교 과제로 주어진 Parallel Binary Search Tree와 LockFree Linked List구현입니다. Java로 구현했으며 Linux 프로그램 성능 분석 도구인 Perf를 이용하여 성능을 분석했습니다.

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
7-2. 이 과정을 C가 T를 가리킬 때 까지 반복한다. 이 방식으로 스레드는 2개의 lock을 잡고 순회하며, C와 P_C가 갱신 되기 전에 P_C를 unlock하고, 갱신한다. 1-2(1번과 2번사이), 4-5를 제외하고 스레드는 2개의 lock을 잡고 있다. C와 P_C 두 개의 lock을 잡고, C하나의 lock만을 잡지 않는다. C가 T일 경우 T에 삽입하는 동안, C가 Delete될 수 있기 때문이다.


### 결과
> Insert Only
![](/portfolio/public/images/4-ParBSTLFLL/ParBST-4-core-IO.png){: width="480" height="320"}
![](/portfolio/public/images/4-ParBSTLFLL/ParBST-8-core-IO.png){: width="480" height="320"}
("$perf stat –d"를 이용하여 테스트)
4-Core Sum : 14825(ms) 11.944 (GHz) 2830.646(M/sec) 23.44(% of cache hits)
8-Core Sum : 15317(ms) 14.953(GHz) 2181.157(M/sec) 22.93(% of cache hits)

* Execution time
스레드 2개일 경우의 성능이 가장 좋았습니다. 스레드가 3개 이상일 경우 root노드에 lock을 잡기 위해 기다리는 스레드가 많아, 느린 것으로 추정됩니다.
* Execution time의 Sum
 보다시피 코어가 8개로 늘어났음에도 불구하고, 큰 성능 증가가 보이진 않았습니다. 이 Fine-Grained lock BST는 성능이 않음을 알 수 있습니다.
* Cache-miss와 cycles, execution time
 BST의 Insert는 Leaf에만 Write를 하므로, 같은 노드를 바라보는 스레드가 있을 경우 한 스레드가 Write를 하여 다른 스레드에게 캐시invalidate영향이 미치는 경우는 별로 없다. 그렇다면 cache-miss가 증가하는 것은 다른 원인으로 볼 수 있습니다. 그 원인은 lock입니다. 한 스레드는 자신이 바라보는 node의 lock이 풀릴 때까지 기다려야 하는데, 한 락을 잡거나 풀기 위해서 모두 그 락에 write를 하기 때문에 invalidate가 생긴 것입니다.
따라서 락에 대기하는 시간이 길어지므로, 스레드는 wait을 하게 되고 cycles수도 그만큼 줄어든 것입니다.
* 스레드가 3,4개 일 때,  8개 일 때
 이것은 thread-switch타임과, CPU 휴식 비용을 생각해 볼 필요가 있습니다. lock의 구현마다 다르겠지만 스레드가 3,4개 일 경우 lock에서 기다리고 있는 스레드는 Sleep을 하게 됩니다. 이 때에는 실질적으로 하는 일이 없으므로 아무런 성능도 내지 못합니다.
하지만 스레드가 8개 일 경우, CPU의 휴식 비율을 줄이고 더 많이 사용함으로써 성능이 좋아졌다고 볼 수 있습니다. 하지만 thread-switch에도 비용이 들기 마련인데, 그럼에도 불구하고 성능이 좋아졌다는 것은 thread-switch cost < CPU burning or Resting cost 이기 때문일 것입니다.


> Insert 100만개 후 , Insert/Search
![](/portfolio/public/images/4-ParBSTLFLL/ParBST-4-core-IS.png){: width="480" height="320"}
![](/portfolio/public/images/4-ParBSTLFLL/ParBST-8-core-IS.png){: width="480" height="320"}

> [Read/Write lock이용] Insert 100만개 후 , Insert/Search
![](/portfolio/public/images/4-ParBSTLFLL/ParBST-4-core-RW-IS.png){: width="480" height="320"}
![](/portfolio/public/images/4-ParBSTLFLL/ParBST-8-core-RW-IS.png){: width="480" height="320"}


### LockFree Linked List



## 사용 언어 / 도구
* Java
* Eclipse
* Shell Script
* (linux) perf


## 테스트 영상
{% include youtube_embed.html id="GOOLeYomCxM" %}  

## 소스코드
> [github](https://github.com/PBW99/HYU_3rd-2se/tree/master/SoftwareEngineering)