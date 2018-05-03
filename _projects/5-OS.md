---
layout: project
title: (C) Play with xv6 Operating System
date: June 20, 2017
image: /portfolio/public/images/5.OS.png
---
## Overview
xv6운영체제를 건드려보는 내용입니다. <br>
기존 xv6운영체제의 프로세스 스케줄링은 프로세스ID별로 순차적으로 실행을 합니다.<br>
하지만 이를 좀 더 효율적으로 바꾸기 위해, MLFQ와 Stride스케줄링을 적용했습니다.
또한 기존의 Process구조체를 이용하여 스레드로 변형해보기도 하였습니다.

## MLFQ
>프로세스를 Fair하게, Starvation이 없도록, Game하는 프로세스(99% time quantum사용후 I/O)가 없도록 만들어진 스케줄링 정책입니다.  
> 각자 다른 우선순위를 가진 Queue를 이용하여 각각의 프로세스들은 Queue를 이동하면서 스케줄링이 됩니다.  
> 운영체제는 The highest priority를 가진 queue의 head를 뽑아서 실행합니다.  

> Mlfq 5가지 규칙
1. A의 Priority가 B보다 높을 때, A가 실행된다. (B는 실행되지 않는다)<br>[Priority]

2. A와 B의 Priority가 같을 때, A와 B는 Round Robin으로 실행된다.<br>[Fairness in same queue level]

3. Job이 System에 도착하면, 가장 높은 Priority를 가진다.

4. 만약 어떤 Job이 할당받은 시간동안 CPU를 오래 점유한다면(언제 포기했는지, 얼마나 많이 포기했는지에 관계없이) Priority는 감소한다.<br>[Demote -> Game the scheduler 방지]

5. 설정한 어떤 주기 S가 되었을 때, 모든 Job의 우선순위를 최고치로 높인다.<br>[Promote -> Starvation 방지]


## Stride Scheduling
> Stride Scheduling은 Turnaround Time, Response Time을 위주로 고려한 것이 아닌, Proportional Share 스케줄링 정책입니다. 각 프로세스가 실제 요청한 Ticket 만큼 더 많이 CPU를 사용하도록 합니다.

> 방법은 다음과 같습니다.

1. 특정 프로세스는 Ticket만큼 CPU를 요구한다. 예를 들어 10을 요구.
2. 임의의 Large Number를 두자(예를 들어 10000) 이 프로세스의 Stride는 LargeNumber/#Ticket 이다.(10000/10)
3. 이 프로세스는 한 Time Quantum마다 Stride만큼 진행한다.
4. 이 프로세스는 현재까지 누적된 Stride값인 Pass를 가진다.
5. 운영체제는 최소 Pass를 가진 프로세스를 스케줄링한다.

## MLFQ + Stride
> 이 프로젝트에서는 위에서 설명한 MLFQ와 Stride를 섞어서 구현하였습니다.<br>
 최대 Ticket(Portion)수를 100으로 두었고, Ticket 100개가 다 사용 중이라면, Scheduling을 거부하여 프로세스는 실행되지 못합니다.<br>
 <br>
 **프로세스가 스케줄링되는 방법**은 다음과 같습니다.
 1. 프로세스는 원하는 Ticket(Portion)과 Mode를 인자로 sched_syscall()를 호출합니다.
 2. Mode가 MLFQ라면 Portion과 무관하게 MLFQ에 의해 스케줄링 되고, MLFQ  고정으로 20개 Ticket을 가지며, Stride Scheduler에 의해 스케줄링 됩니다.(즉 Stride Scheduler입장에서는 MLFQ가 하나의 프로세스입니다.)
 3. Mode가 Stride라면 현재 부여된 Ticket수를 초과하여 Ticket을 요구하였는지 체크하고, 넘었다면 스케줄링을 거부합니다.
 4. 넘지 않았다면 Stride Scheduler에 의해 스케줄링 됩니다.
<br>

> **Timer와 Context Switch**
1. Timer Interrupt가 발생할 때마다, 전체 시간을 가지는 tick변수를 증가시킵니다.(MLFQ 5번 Rule Boost를 구현하기위해)
2. 또한 현재 실행되는 프로세스 구조체의 consumedTime을 증가시킵니다.
3. 정해진 STQ(Stride Time Quantum)만큼 프로세스가 시간을 소모했다면 strideSched()에 의해 Context Switch가 일어납니다.( Min pass를 가진 프로세스가 스케줄됨)
4. 그렇지 않다면 실행되는 프로세스가 MLFQ인지 확인합니다. 아니라면 return
5. MLFQ라면 MLFQ안에서 실행되는 프로세스의 Consumed Time을 체크하여, 해당 레벨에서 정해진 Time Quantum을 초과하여 계속 동작하였거나(MLFQ 2번 Rule), 정해진 Time Allotment를 초과하여 현재 레벨에 있었는지(MLFQ 4번 Rule)를 확인합니다. 두 조건중 하나라도 만족되면 mlfqSched()에 의해 Context Switch가 일어납니다. ( 제일 높은 Level Queue의 head가 스케줄됨)



> **Diagram**
 
![](/portfolio/public/images/5-OS/overview.png){: width="1080" height="680"}




## 사용 언어 / 도구
* C
* Vim
* Qemu


## 소스코드
> [github](https://github.com/PBW99/HYU_3rd-1se/tree/master/OS)