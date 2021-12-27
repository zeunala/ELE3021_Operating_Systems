# 운영체제 실습과제
운영체제 수업 과제로 작성했던 코드로, 교육용 운영체제 xv6를 이용하여 운영체제 이론 시간에 배웠던 내용들을 실제로 적용해보는 것을 목표로 한다.

- Project 1: 시스템 콜, 인터럽트 추가
- Project 2: FCFS, Multilevel Queue, MLFQ 스케줄러 구현
- Project 3: inode의 double indirect 구현

## Project 1

### 목표
* 부모프로세스의 id를 가져오는 getppid() 시스템 콜을 추가하고 이를 확인하기 위한 유저프로그램(project01_1.c)을 작성한다.
* 128번 인터럽트를 추가하고 이를 유저프로그램(project01_2.c)으로 확인한다.
  * 128번 인터럽트에서는 단순히 "use interrupt 128 called!"를 print하고 프로세스를 종료하는 역할을 하도록 한다.

## Project 2

### 목표
* 기존 xv6에서는 매우 간단한 Round-Robin 방식의 스케줄러를 사용하고 있는데, 이를 이론 시간에 배운 FCFS, Multilevel Queue, MLFQ 스케줄러를 이용하도록 구현한다.
* FCFS에서는 먼저 fork된 프로세스가 먼저 스케줄링 되도록 구현하며, 기본적으로 스케줄링된 프로세스는 종료되기 전까지는 switch-out 되지 않도록 한다.
  * 단, 스케줄링 된 이후 200 ticks가 지날 때까지 종료되거나 SLEEPING 상태로 전환되지 않으면 강제로 종료시킨다.
  * 실행 중인 프로세스가 SLEEPING으로 전환시 다음으로 생성된 프로세스가 스케줄링되며, SLEEPING상태이면서 먼저 생성된 프로세스가 깨어나면 다시 그 프로세스가 스케줄링 되도록 한다.
* Multilevel Queue에서는 Round Robin 큐와 FCFS 큐 2개의 큐로 구성한다.
  * pid가 짝수이면 Round Robin, pid가 홀수이면 FCFS로 스케줄링 되도록 한다.
  * Round Robin 큐가 FCFS 큐보다 우선순위가 높도록 한다.
* MLFQ에서는 2-level feedback queue로 L0(우선순위 높음)와 L1(우선순위 낮음) 두 개의 큐로 구성한다.
  * L0는 4 ticks, L1은 8 ticks의 time quantum을 가지도록 한다.
  * 처음 실행된 프로세스는 L0에 들어가며 quantum을 모두 사용한 경우 L1 큐로 내려가도록 한다. L1 큐의 프로세스가 quantum을 모두 사용하면 해당 프로세스의 priority를 1 감소시킨다. (최소 0)
  * L0큐는 기본 RR정책을 따르며, L1큐는 priority scheduling을 하도록 한다.
    * 처음 프로세스의 priority는 0이며, setpriority() 시스템 콜을 추가하여 우선순위를 0~10으로 설정할 수 있게 한다. (클수록 높은 우선순위)
    * priority는 L1큐에서만 유효하며, 값이 같을 경우 FCFS로 우선순위를 결정한다.
  * Starvation을 막기 위해 200 ticks마다 모든 프로세스들을 L0큐로 올리고 priority도 0으로 리셋한다. (Priority boosting)
  * monopolize 시스템 콜을 추가하여 MLFQ 스케줄링을 무시하고 프로세서를 독점할 수 있도록 한다. 인자로 암호를 받게 하여 일치하지 않을 경우 프로세스를 강제로 종료시킨다.
  

## Project 3

### 목표
* 기존 xv6의 inode에서 한 번의 indirect를 지원했던 것에서 확장하여, 더 큰 파일을 다룰 수 있도록 double indirect를 구현한다.
