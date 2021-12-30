# 운영체제 실습과제
운영체제 수업 과제로 작성했던 코드로, 교육용 운영체제 xv6를 이용하여 운영체제 이론 시간에 배웠던 내용들을 실제로 적용해보는 것을 목표로 한다.

- Project 1: 시스템 콜, 인터럽트 추가
- Project 2: FCFS, Multilevel Queue, MLFQ 스케줄러 구현
- Project 3: inode의 double indirect 구현

## Project 1

### 목표
* (Project 1-1) 부모프로세스의 id를 가져오는 getppid() 시스템 콜을 추가하고 이를 확인하기 위한 유저프로그램(project01_1.c)을 작성한다.
* (Project 1-2) 128번 인터럽트를 추가하고 이를 유저프로그램(project01_2.c)으로 확인한다.
  * 128번 인터럽트에서는 단순히 "use interrupt 128 called!"를 print하고 프로세스를 종료하는 역할을 하도록 한다.

### Project 1-1 디자인
- xv6에서 부모프로세스의 id를 리턴하는 getppid() 시스템 콜을 구현해야 한다.
- Cscope로 getpid에 관한 정보를 참고한 결과, sysproc.c에서 자기자신의 프로세스의 id를 리턴하는 getpid() 시스템 콜을 확인할 수 있었다.
![1](https://user-images.githubusercontent.com/79515820/147581711-3ad11dc1-8df2-4e30-8391-d510ddfd6c31.png)
![2](https://user-images.githubusercontent.com/79515820/147581934-4d0aac89-4b1b-4e0e-823f-64bb357e62f6.png)
- myproc()->pid의 관한 내용은 proc.h에서 다음과 같이 확인할 수 있었다.
![3](https://user-images.githubusercontent.com/79515820/147582137-f8ede042-4611-474e-b620-84d6ad2938ab.png)
- myproc()은 현재 자기자신의 프로세스를 의미하므로, 위의 주석에 의하면 myproc()->parent가 부모 프로세스를 가리키게 된다. 따라서 myproc()->parent->pid를 리턴하게 하면 부모 프로세스의 id를 리턴하는 함수를 만들 수 있게 되며, 이를 바탕으로 다음과 같은 방법을 통해 최종적으로 구현할 수 있다.

### Project 1-1 구현
1. 먼저 sys_getpid() 함수가 있었던 sysproc.c로 들어가 그 밑에 sys_getppid()함수를 추가해준다(line 45~49). 이 함수는 myproc()->parent->pid, 즉 부모 프로세스의 id를 리턴하도록 구현한다. 별도의 파일을 만들지 않고 기존 파일에 함수를 추가하는 방식으로 작성하였으므로 시스템 콜을 추가 한 것에 대해서는 Makefile을 재작성할 필요는 없다.
![4](https://user-images.githubusercontent.com/79515820/147582139-d9a8a99d-2263-4d92-b84d-2a0d75eacba2.png)
2. 다른 함수들에게서도 새로 만든 시스템 콜을 호출할 수 있도록 defs.h에서 다음과 같이 함수의 원형을 적어준다. (line 190)
![5](https://user-images.githubusercontent.com/79515820/147582140-7fd871eb-c071-42f0-879c-5c458e22f54f.png)
3. 시스템콜 호출시 새로 만든 sys_getppid() 함수로 연결해 줄 수 있도록 syscall.h와 syscall.c 파일을 다음과 같이 변경한다.
![6](https://user-images.githubusercontent.com/79515820/147582141-c0a34d41-6eaa-4b47-9ba9-a39f5936a019.png)
- syscall.h의 line23에 새로 만든 시스템 콜에 번호 22를 부여하여 추가하였다.
![7](https://user-images.githubusercontent.com/79515820/147582143-21e9cc93-b9e6-4390-8b9f-07f4836da0a7.png)
![8](https://user-images.githubusercontent.com/79515820/147582144-9136d9ef-fe42-4b2c-acf2-04be9d75562c.png)

- SYS_getppid 번호가 처음에 구현해준 sys_getppid와 연결되도록 syscall.c의 line 106과 line129에 다음과 같이 추가해 준다. 여기까지의 과정을 통해 시스템 콜의 기본적인 구현은 완료되었다.

4. 다음으로는 새로 만든 시스템콜을 user program에서도 사용할 수 있게 해주는 과정을 거쳐야 한다. 시스템콜이 유저 프로그램에서 사용할 수 있도록 다음과 같이 user.h와 usys.S에 새로 만든 시스템 콜에 관련된 내용을 추가해준다. user.h에는 line42에, usys.S에는 line32에 새로 만든 시스템 콜에 대한 내용을 추가하였다.
![9](https://user-images.githubusercontent.com/79515820/147582147-d58cfaf4-80dc-47a5-8737-77d16fb0cfe1.png)
![10](https://user-images.githubusercontent.com/79515820/147582149-4f4d33d1-362a-4fbf-84d2-25802a9d1f52.png)
5. 이제 시스템 콜을 만들고 user program에서도 사용할 수 있게 하였으니 새로 만든 시스템 콜을 이용하는 유저 프로그램을 만들면 된다. 문제에서 제공한 실행예시와 같이 출력하기 위해 다음과 같이 project01_1.c를 구현하였다. line8에서는 getpid()호출을 통해 현재 프로세스의 id를, line9에서는 새로 만든 시스템 콜 getppid()호출을 통해 부모 프로세스의 id를 출력하게 된다.
![11](https://user-images.githubusercontent.com/79515820/147582152-26f95e58-3ba5-46e0-baf3-14135aa5eb4d.png)
6. 유저 프로그램이 xv6에서 실행될 수 있도록 Makefile에 project01_1과 관련된 내용을 추가한다. line184, line254에 project01_1이 같이 make되도록 추가하였다. 이후 make clean->make->make fs.img를 통해 다시 make를 하면 최종적으로 반영된다.
![12](https://user-images.githubusercontent.com/79515820/147582153-7a6f350e-2503-4cd4-998f-a80f0ff518af.png)
![13](https://user-images.githubusercontent.com/79515820/147582155-c6c01ae2-f6c7-45f8-ae54-9a1d534d8268.png)

### Project 1-1 실행결과
- make가 완료된 후 xv6를 실행하고 project01_1을 실행하면 아래와 같이 잘 동작하는 것을 확인할 수 있다. 첫번째 줄에서는 pid, 현재 프로세스의 id가 출력되고 두번째 줄에서는 ppid, 부모 프로세스의 id가 출력되는 것을 확인할 수 있다.
![14](https://user-images.githubusercontent.com/79515820/147582131-40376eea-a436-4f5d-b4e6-53842ae76f39.png)

### Project 1-2 디자인
- 유저 프로그램에서 \_\_asm\_\_(“int 128”); 을 통해 128번 인터럽트를 실행했을 때, 128번 인터럽트에서는 “user interrupt 128 called” 문장을 출력하고 프로세스를 종료해야 한다.
- 고려해야 할 상황은 두 가지로, 먼저 trap.c에서 trapno가 128일 때 해당 문장을 출력하고 프로세스를 종료하도록 인터럽트를 만들어주어야 한다. 두 번째는 새로 정의한 인터럽트 128이 시스템콜처럼 유저 프로그램에서도 실행이 가능하게 해주어야 한다. 이 때문에 tvinit함수에도 인터럽트 128에 대한 내용을 추가해 주어야한다.


### Project 1-2 구현
1. trap.c의 trap() 함수의 switch(tf->trapno){…}문 안에서, 다음의 내용(line 80~82)을 추가한다. trap함수의 swtich문에서는 trapframe의 trapno에 따라 처리를 하므로, 이 부분에서 128번 인터럽트에 대한 처리를 추가하는 것이다. line81에서는 지정한 문구를 출력하고, line82에서는 프로세스를 종료하는 역할을 한다.
![15](https://user-images.githubusercontent.com/79515820/147583203-9dc014ce-8804-4853-8359-f8a021db53fe.png)
2. 앞에서 정의한 128번 인터럽트는 시스템콜처럼 유저 프로그램에서도 실행이 가능하게 해야 한다. 따라서 trap.c의 tvinit()함수에 line25와 같이 추가를 해주어야 한다. 이렇게 하면 interrupt descriptor를 초기화할 때 DPL을 유저로 설정해서, 유저모드에서 128번 인터럽트의 호출/실행이 가능하게 된다.
![16](https://user-images.githubusercontent.com/79515820/147583204-a90aef7c-ecd0-475d-b109-4709da842e84.png)
3. 이후 project01_1에서 했던 것처럼, project01_2.c를 정의하고 해당 유저 프로그램과 함께 make되도록 Makefile에 관련 내용을 추가해주면 된다(line 185, 255). project01_2.c의 내용은 과제에서 제시한 것과 동일하다. 이후 make clean -> make -> make fs.img를 진행해서 완료한다.
![17](https://user-images.githubusercontent.com/79515820/147583206-89bb3292-f3f9-4609-bbd5-05f1abde7823.png)
![18](https://user-images.githubusercontent.com/79515820/147583208-bfd314d7-67b0-4945-8a09-8339f81e09ae.png)
![19](https://user-images.githubusercontent.com/79515820/147583209-cc1e04b0-e0b1-4b99-a9ca-41902688213f.png)

### Project 1-2 실행결과
- make 후 xv6를 실행한 뒤에 project01_2을 실행하면 아래와 같이 잘 동작하는 것을 확인할 수 있다. project01_2.c에서 \_\_asm\_\_(“int $128”);을 통해 인터럽트 128번을 호출하면 앞에서 인터럽트 128번을 정의한대로 “user interrupt 128 called!”를 출력하고 프로세스를 종료하는 것을 확인할 수 있다.
![20](https://user-images.githubusercontent.com/79515820/147583200-04dea67a-d4ad-4097-a412-10322284c92e.png)

## Project 2

### 목표
* 기존 xv6에서는 매우 간단한 Round-Robin 방식의 스케줄러를 사용하고 있는데, 이를 이론 시간에 배운 FCFS, Multilevel Queue, MLFQ 스케줄러를 이용하도록 구현한다.
* (Project 2-1) FCFS에서는 먼저 fork된 프로세스가 먼저 스케줄링 되도록 구현하며, 기본적으로 스케줄링된 프로세스는 종료되기 전까지는 switch-out 되지 않도록 한다.
  * 단, 스케줄링 된 이후 200 ticks가 지날 때까지 종료되거나 SLEEPING 상태로 전환되지 않으면 강제로 종료시킨다.
  * 실행 중인 프로세스가 SLEEPING으로 전환시 다음으로 생성된 프로세스가 스케줄링되며, SLEEPING상태이면서 먼저 생성된 프로세스가 깨어나면 다시 그 프로세스가 스케줄링 되도록 한다.
* (Project 2-2) Multilevel Queue에서는 Round Robin 큐와 FCFS 큐 2개의 큐로 구성한다.
  * pid가 짝수이면 Round Robin, pid가 홀수이면 FCFS로 스케줄링 되도록 한다.
  * Round Robin 큐가 FCFS 큐보다 우선순위가 높도록 한다.
* (Project 2-3) MLFQ에서는 2-level feedback queue로 L0(우선순위 높음)와 L1(우선순위 낮음) 두 개의 큐로 구성한다.
  * L0는 4 ticks, L1은 8 ticks의 time quantum을 가지도록 한다.
  * 처음 실행된 프로세스는 L0에 들어가며 quantum을 모두 사용한 경우 L1 큐로 내려가도록 한다. L1 큐의 프로세스가 quantum을 모두 사용하면 해당 프로세스의 priority를 1 감소시킨다. (최소 0)
  * L0큐는 기본 RR정책을 따르며, L1큐는 priority scheduling을 하도록 한다.
    * 처음 프로세스의 priority는 0이며, setpriority() 시스템 콜을 추가하여 우선순위를 0~10으로 설정할 수 있게 한다. (클수록 높은 우선순위)
    * priority는 L1큐에서만 유효하며, 값이 같을 경우 FCFS로 우선순위를 결정한다.
  * Starvation을 막기 위해 200 ticks마다 모든 프로세스들을 L0큐로 올리고 priority도 0으로 리셋한다. (Priority boosting)
  * monopolize 시스템 콜을 추가하여 MLFQ 스케줄링을 무시하고 프로세서를 독점할 수 있도록 한다. 인자로 암호를 받게 하여 일치하지 않을 경우 프로세스를 강제로 종료시킨다.
  
### Project 2 사전작업

아래에서 스케줄러를 구현한 것 이외에도 컴파일 과정에서 다음과 같은 조치가 선행되어야 한다.

1. 다음과 같이 make의 인자로 어떠한 스케줄링을 사용할지 결정하게 한다.
![21](https://user-images.githubusercontent.com/79515820/147749142-b9f7a9bc-29a2-4dfc-88e0-7cd95b27ab87.png)
2. 테스트 프로그램을 정상적으로 실행할 수 있도록 Makefile에 추가해준다.
![22](https://user-images.githubusercontent.com/79515820/147749040-d577abc0-fe57-4f11-879f-aa8ecdcd14ed.png)


### Project 2-1 디자인
- 이번에 구현할 FCFS 스케줄러에서는 1. 먼저 생성된 프로세스가 먼저 스케줄링이 되도록 해야하며, 2. 종료되거나/200ticks가 되도록 종료 및 sleeping으로 전환되지 않아 강제 종료 시키는 경우를 제외하고는 switch-out되어서는 안된다.
- xv6는 스케줄러에서 타이머 인터럽트가 10ms마다 발생할 때마다 현재 프로세스를 RUNNABLE로 전환시키고 다음 프로세스를 실행시키는 Round-Robin 정책을 사용하고 있다. FCFS 스케줄러를 구현할 때는 타이머 인터럽트가 발생하더라도 runnable로 바꾸고 다음 프로세스를 실행시키는 일이 없도록 해야 한다. 다만 200ms가 되도록 스케줄링이 일어나지 않을 경우 현재 프로세스를 강제로 종료하도록 별도의 변수를 추가할 필요가 있다.
- 스케줄러에서 다음 실행할 프로세스를 고를 때는 상태가 RUNNABLE 한 것 중에서 가장 먼저 생성된 프로세스를 우선적으로 골라야 한다. 처음에는 ptable기준으로 제일 앞에 있는 프로세스부터 실행하면 된다고 생각했으나, 프로세스가 작업을 마쳤을 때 그 인덱스로 새로운 프로세스가 들어갈 수 있으므로 ptable 인덱스 순으로 선택하면 안 된다. 대신 pid가 작은 프로세스가 먼저 생성되었음을 의미하므로 pid를 이용해 가장 먼저 생성된 프로세스를 고를 수 있다.

### Project 2-1 구현
1. 우선 타이머 인터럽트가 발생하더라도 다음 프로세스를 실행하지 않게 하되 스케줄러가 실행된지 200tick이 넘으면 강제종료 시켜야 한다. 스케줄러가 마지막으로 실행된 지 얼마나 시간이 발생했는지를 체크하기 위한 별도의 변수 int ticks_FCFS를 다음과 같이 trap.c, proc.c에 정의한다. 스케줄러가 실행할 때마다 이 값은 0으로 초기화되어야 한다.
![23](https://user-images.githubusercontent.com/79515820/147749041-efd9001c-9895-4658-8285-e6f8b6555c7b.png)
![24](https://user-images.githubusercontent.com/79515820/147749042-2a3082c3-5299-43fb-89c6-9a14e595f931.png)
![25](https://user-images.githubusercontent.com/79515820/147749043-d1ba9071-29ef-4069-a815-a072443e283f.png)

2. 기존 xv6에서는 타이머 인터럽트가 발생할 때마다 yield로 다른 프로세스에게 cpu를 양보하게 된다(trap.c에 이 부분이 정의되어 있다). 이 부분을 FCFS스케줄러로 xv6를 실행할 때는 yield를 하는 대신 ticks_FCFS값을 1 증가시키기만 한다. 대신 schedular함수가 실행될 때마다 초기화되는 ticks_FCFS가 200을 넘었다는 것은 프로세스가 스케줄링 된 이후 200ticks가 지날 때까지 종료되거나 SLEEPING 상태로 전환되지 않았다는 것이므로 현재 프로세스를 kill시키고 다른 프로세스에게 cpu를 넘긴다.
![26](https://user-images.githubusercontent.com/79515820/147749044-b5376816-51e2-4c50-a0fd-62f544b7e40d.png)
3. 다음으로 proc.c에 있는 scheduler함수를 변경해주면 된다. 먼저 FCFS 스케줄러로 실행할 때는 가장 작은 pid 값의 프로세스를 저장하도록 struct proc *minPid 변수를 선언한다. 처음에는 NULL값을 가진다.
![27](https://user-images.githubusercontent.com/79515820/147749045-a263932c-a50d-4859-a01c-1a5f06753f20.png)
4. 다음으로 다른 프로세스를 선택하는 과정을 바꿔줘야 한다. 기존 xv6에서는 ptable을 돌면서 상태가 RUNNABLE인 프로세스를 발견하는 순간 그 프로세스를 대상으로 cpu를 주게 된다. 이 과정을 FCFS 스케줄러로 실행하는 경우에는 ptable을 모두 확인하고, pid가 가장 작은 프로세스를 minPid에 넣어서 이후 해당 프로세스에게 cpu를 넘겨준다. minPid를 정하고 난 뒤 cpu를 주는 부분은 기존 xv6 코드에서 p를 minPid로만 바꿔주면 된다.
![28](https://user-images.githubusercontent.com/79515820/147749046-98db417b-1701-4f78-8d84-ded9fa2d30d4.png)
![29](https://user-images.githubusercontent.com/79515820/147749049-e15f9e80-7526-4466-ad7e-15b9dc38000e.png)
![30](https://user-images.githubusercontent.com/79515820/147749050-0d042be7-d68c-4b52-87cc-b0220c839277.png)
5. 추가적으로 yield를 유저프로그램에서도 사용할 수 있도록 시스템콜을 만들어줘야 테스트파일을 정상적으로 실행할 수 있다. 
 (1) proc.c에서 yield의 wrapper function을 정의하였다. yield는 리턴하지 않는 함수이지만 다른 시스템콜과 반환형을 맞추기 위해 0을 리턴하도록 하였다. 추가로 proc.h에서도 해당 함수를 선언하였다.
 (2) syscall.h, syscall.c에서 yield 시스템콜을 번호를 부여하며 추가하였다.
 (3) 마지막으로 구현한 시스템콜을 유저 프로그램에서 사용할 수 있도록 user.h와 usys.S에 관련된 내용을 추가하였다.
 
### Project 2-1 실행결과
- 파일을 모두 위와 같이 변경한 후 make SCHED_POLICY=FCFS_SCHED -> make fs.img -> ./bootxv6.sh 순으로 컴파일을 진행하고(실습강의에서 제공한 bootxv6.sh파일 사용) xv6에서 p2_fcfs_test 파일을 실행한 결과 아래와 같이 결과가 정상적으로 나오는 것을 확인할 수 있다. 일반적인 경우, yield를 하는 경우 모두 pid순으로 잘 출력되는 것을 확인할 수 있다. 다만 sleep을 하는 경우에는 다소 순서가 뒤죽박죽인데, 이는 실행중인 프로세스가 sleeping 상태로 전환된 경우 다음으로 pid가 작은 프로세스가 스케줄링이 되기 때문이다. sleep을 하다 깨어나면 그 프로세스가 다시 최소 pid 프로세스가 되므로 도중에 더 앞의 번호의 pid가 적히는 것이다. 마지막으로 무한루프를 돌 경우 도중 200ticks를 채우기 때문에 process kill이 일어나 프로그램이 종료되는 것을 확인할 수 있다.
![31](https://user-images.githubusercontent.com/79515820/147749052-46f56fba-8c39-4134-aeb5-e8304069fde1.png)
![32](https://user-images.githubusercontent.com/79515820/147749053-34a67638-f18c-4c74-be3f-98804eef029b.png)
![33](https://user-images.githubusercontent.com/79515820/147749054-e2031706-ca79-4da9-b12a-77434b69d726.png)
![34](https://user-images.githubusercontent.com/79515820/147749056-f334fcbe-63c4-4d70-9371-ca1d0ca7c16d.png)

### Project 2-2 디자인
- 이번에 구현해야 하는 Multilevel Queue에서는 Round robin 방식과 FCFS 방식을 각각 이용하는 2개의 큐로 이루어지고, pid가 짝수인 프로세스는 RR으로, pid가 홀수인 프로세스는 FCFS로 스케줄링해야 한다. 또한 RR큐가 FCFS큐보다 우선순위가 높도록 해야 한다.
- 따라서 스케줄러에서 RR큐와 FCFS큐 두 개로 나누고, RR큐를 먼저 봐서 runnable한 프로세스를 실행한 후, 만약 RR큐를 모두 보아도 runnable 프로세스가 없을 경우에 그제서야 FCFS큐를 보도록 하면 된다. RR큐에서는 원래 xv6가 하던 방식 그대로 가져오면 round robin으로 스케줄링 될 것이며, FCFS큐에서는 project 02-1에서 했던대로 pid가 가장 작은 것을 선택해서 스케줄링을 하면 된다.

### Project 2-2 구현
1. proc.h에 있는 proc 구조체에 int queueLevel 변수를 추가한다. 이 값은 해당 프로세스가 어느 큐에 들어가 있는지를 나타내며, queueLevel=0이면 RR큐에, queueLevel=1이면 FCFS큐에 들어가 있음을 의미한다.
![35](https://user-images.githubusercontent.com/79515820/147749057-1559692f-9b61-4b41-8b63-d846f4861b6a.png)
2. proc.c에 있는 allocproc함수에서 pid에 따라 서로 다른 큐에 집어넣도록 다음 코드를 추가한다. pid가 짝수이면 RR큐에, pid가 홀수이면 FCFS큐에 들어가게 된다.
![36](https://user-images.githubusercontent.com/79515820/147749058-4b9dcc63-5f3b-414b-8a5f-df4dd2afa776.png)
3. RR큐에서는 타이머 인터럽트가 올 때마다 다른 프로세스를 실행해야 하나, FCFS큐에서는 pid가 작은 프로세스가 끝까지 실행해야 한다. 따라서 다음과 같이 타이머 인터럽트가 올 때 RR큐에서만 yield를 하도록 한다. (project 02-2에서는 FCFS 200tick 강제종료 조건이 없었기에 02-1에서 사용한 기능을 넣지 않았다)
![37](https://user-images.githubusercontent.com/79515820/147749060-746a0d23-9c26-4ae4-83a0-0b58b9e985f3.png)
4. scheduler 함수에서 targetP 포인터변수를 추가한다. minPid 때와 유사하게 이 변수는 RR큐에 runnable한게 있으면 RR큐의 프로세스를 가리키고, 없을 경우 FCFS큐에서 가장 pid가 작은 프로세스를 가리키게 된다.
![38](https://user-images.githubusercontent.com/79515820/147749061-a2f169ce-5612-430e-9e0a-2e6b0a900b5c.png)
5. 이후 다음 프로세스를 고르는 부분을 다음과 같이 추가한다. RR큐를 먼저 체크해서 runnable한게 있으면 그것을 우선으로 targetP로 설정하고, runnable한 게 없으면 FCFS큐를 모두 봐서 pid가 가장 작은 프로세스를 targetP로 설정하게 된다. 이후 해당 선택한 targetP 프로세스에게 CPU를 주게 된다.
![39](https://user-images.githubusercontent.com/79515820/147749065-612a1683-6d50-4153-870d-fc19b173b7b2.png)
![40](https://user-images.githubusercontent.com/79515820/147749066-a9aad660-fce3-4a32-911e-fd28c4a3db4b.png)

### Project 2-2 실행결과
- 파일을 모두 위와 같이 변경한 후 make SCHED_POLICY=MULTILEVEL_SCHED -> make fs.img -> ./bootxv6.sh 순으로 컴파일을 진행하고(실습강의에서 제공한 bootxv6.sh파일 사용) xv6에서 p2_fcfs_test 파일을 실행한 결과 아래와 같이 결과가 정상적으로 나오는 것을 확인할 수 있다. 우선 pid가 짝수인걸 먼저 실행하고, pid가 짝수인 runnable 프로세스가 없으면 pid가 홀수인 프로세스가 pid가 작은 것부터 실행된다. RR큐에서는 10ms마다 cpu를 잡는 프로세스가 바뀌므로 pid가 짝수인 것 끼리는 순서가 다음과 같이 섞일 수 있게 된다.
![41](https://user-images.githubusercontent.com/79515820/147749068-325348b9-746c-45b1-8228-97e2b1b65d27.png)
![42](https://user-images.githubusercontent.com/79515820/147749069-3ece2f51-a277-4286-9df6-02bcae6142a6.png)
![43](https://user-images.githubusercontent.com/79515820/147749070-d8291923-a594-45c0-b677-c680ed5f1c81.png)
![44](https://user-images.githubusercontent.com/79515820/147749071-591613c6-8fae-4e0d-b3b7-1c2afffb010d.png)
![45](https://user-images.githubusercontent.com/79515820/147749128-fa033b46-eba8-4d44-bdbc-99895aac46c4.png)

### Project 2-3 디자인
- 이번에 구현하는 MLFQ에서는 기본 RR 정책을 따르되 조건에 따라 L0큐와 L1큐 사이를 이동할 수 있어야 한다. Multilevel 큐를 구현했을 때와 마찬가지로 queueLevel을 두도록 하되, monopolize를 통해 CPU를 독점하는 경우 최우선적으로 권한을 가지는 별도의 큐를 만들어 그 것으로 이동하는 방법을 취한다(queueLevel을 100/101으로 설정한 큐로 이동시키고, 스케줄러에서는 이 큐를 가장 먼저 체크하여 있을 경우 이를 최우선으로 처리한다). 또한 타이머 인터럽트가 발생할 때마다 해당 프로세스에게 주어진 time quantum이 감소하여 모두 사용한 경우 L1큐로 내려가거나 우선순위가 내려가게 해야 하며, 200 ticks가 지날 때마다 L0큐/priority0으로 초기화를 해줘야 한다.

### Project 2-3 구현
1. Multilevel queue 때와 유사하게 proc.h에 정의된 proc구조체에 int queueLevel2와 int priority2 변수를 추가한다. 이들은 각각 위치한 큐의 종류와 우선순위를 나타낸다. (MULTILEVEL일 때와 구분하기 위해 뒤에 숫자 2를 붙여주었다) 또한 proc.c의 allocproc함수에서 프로세스가 만들어질 때 이들은 모두 0으로 설정된다. (L0큐, 우선순위 0)
![46](https://user-images.githubusercontent.com/79515820/147749075-76e1257c-5df9-4d0c-add3-553be690a8a7.png)
![47](https://user-images.githubusercontent.com/79515820/147749076-4c060177-6a9f-4484-b595-dbd10c194f92.png)
2. proc.c의 scheduler함수에서 cpu를 줄 프로세스를 지정하는 targetP2 포인터변수를 추가한다. 이후 다음으로 cpu를 줄 프로세스를 선정할 때 1. monopolize 상태의 프로세스 -> 2. L0큐에 있는 runnable 프로세스 -> 3. L1큐에 있는 runnable 프로세스 순으로 찾게 된다. (L1큐안에서 프로세스를 선정할 때는 우선순위가 큰 경우 -> pid가 작은 경우 순으로 프로세스 우선순위를 갖는다)
![48](https://user-images.githubusercontent.com/79515820/147749077-8cee881c-e5e2-450e-85f9-1475b58c67c7.png)
![49](https://user-images.githubusercontent.com/79515820/147749078-64eb68fc-9739-45a4-b14b-bdc7881a4c1a.png)
![50](https://user-images.githubusercontent.com/79515820/147749079-5055ba04-222e-4fac-8edb-9f1c1bf29d7d.png)
3. 또한 프로세스가 실행될 때 특정 tick이 경과할 때마다 큐이동/priority boost의 조치를 취해줘야 하므로 이 부분도 구현해야 한다. 먼저 trap.c에서 ticks_L0, ticks_L1, ticks_boost 변수를 추가한다. ticks_L0은 프로세스가 L0큐에서 돌아갈 때 1씩 감소하며 0이 되면 L1으로 떨어지며 yield를 호출하고 다시 4로 초기화 된다. ticks_L1은 프로세스가 L1큐에서 돌아갈 때 1씩 감소하며 0이 되면 우선순위가 1 감소하며 yield를 호출하고 다시 8로 초기화된다. ticks_boost는 어느 큐에 있든 상관없이 1씩 감소하며 0이 되면 모든 프로세스를 L0큐로 이동시키고 priority도 0으로 초기화한다 (다만 이 때 monopolize 상태의 프로세스는 건드리지 않으며 원래 큐를 L1에서 L0으로 바꿔주기만 한다) ticks가 1씩 감소하는 것은 trap.c에서 타이머 인터럽트가 왔을 때 처리해주며 ticks가 0이하일 때 처리를 하는 것은 proc.c의 스케줄러에서 해준다. 이로써 기본적인 MLFQ의 구현은 완료되었다.
![51](https://user-images.githubusercontent.com/79515820/147749080-7c36144a-4292-4455-8497-69a3b88e25a0.png)
![52](https://user-images.githubusercontent.com/79515820/147749082-bbcd9369-c294-49c9-8f84-4fea35c4d4fd.png)
![53](https://user-images.githubusercontent.com/79515820/147749083-1a46b8ae-ce26-4bf6-a1be-6176e96555da.png)
![54](https://user-images.githubusercontent.com/79515820/147749085-eeaf564a-4500-4c7d-a80e-fbeee69a237b.png)
![55](https://user-images.githubusercontent.com/79515820/147749086-4cdb25d0-9469-44e9-bded-8582472f9b1a.png)
4. 마지막으로 지정한 system call을 구현하고 유저프로그램에서도 사용할 수 있게 해야 한다. getlev함수는 현재 어디 큐에 있는지 큐의 레벨을 반환한다. %100을 한 것은 기본적으로 monopolize 상태인 프로세스를 queueLevel에 100을 더해서 표시하기 때문이며, 이 경우 원래 큐레벨을 표시하기 위해 100으로 나눈 나머지를 반환한 것이다.
![56](https://user-images.githubusercontent.com/79515820/147749087-a6c30e12-1ba1-457f-b93d-f0648f80ceef.png)
setpriority함수는 pid와 priority값을 받아 지정한 프로세스의 priority값을 변경해준다. 만약 priority값이 유효하지 않으면 -2를 리턴하고, ptable에서 해당 pid를 찾을 수 없으면 -1을 리턴한다. 정상적으로 변경했을 경우 0을 리턴한다. 이 때 ptable을 확인하기 전 lock을 획득해서 race condition을 막는다.
![57](https://user-images.githubusercontent.com/79515820/147749089-aa59d22f-c6f6-43de-951a-786608dd380d.png)
monopolize함수는 암호(2017029661)를 입력받아 암호가 맞다면 스케줄러의 최우선순위를 가지게 된다. queueLevel2의 값을 100을 더하는 것으로 최우선 순위의 큐에 넣어주는 효과를 가진다. 만약 이미 queueLevel2의 값이 100 이상이면 이미 monopolize상태인 것이므로 L0큐/priority 0으로 초기화해준다. 만약 입력받은 암호가 틀렸다면 현재 프로세스를 kill하고 yield로 다른 프로세스를 부르도록 한다.
![58](https://user-images.githubusercontent.com/79515820/147749090-97248f8a-2800-4c0d-961f-88a2dccc5a74.png)
각각의 함수는 MLFQ_SCHED가 아닌 경우 단순히 0을 리턴하거나 아무것도 안 하는 함수가 된다.

5. 또한 정의한 함수를 systemcall로 만들고 유저프로그램에서 이용할 수 있도록 코드를 추가해야 한다. 다음과 같이 wrapper function을 추가하고 defs.h, syscall.h, syscall.c, user.h, usys.S에 다음의 코드를 추가해 주어 system call로 만들고 유저프로그램에서도 이용할 수 있게 한다. (실습시간에 배웠던 방법과 동일한 방법으로 진행하였으며, 다만 인자가 int형인 경우 argint를 이용하여 인자를 받아 함수를 호출하도록 하였다)
![59](https://user-images.githubusercontent.com/79515820/147749091-4fd7e461-e220-472c-aff5-df43ee25ca3d.png)
![60](https://user-images.githubusercontent.com/79515820/147749093-6802a547-e0f1-488c-b083-66155eb45d0c.png)
![61](https://user-images.githubusercontent.com/79515820/147749094-180d8986-f7c9-4c4c-be6a-c8088154c3fb.png)
![62](https://user-images.githubusercontent.com/79515820/147749095-66cab713-2068-4d6a-a325-693179517ba3.png)
![63](https://user-images.githubusercontent.com/79515820/147749096-10d48284-9a29-478f-a632-bf3f835ad7ad.png)
![64](https://user-images.githubusercontent.com/79515820/147749097-27b4908d-edbf-4564-a989-56c7a4512f9b.png)
![65](https://user-images.githubusercontent.com/79515820/147749098-f069a754-1b34-4182-8ce0-1d043e11d029.png)
![66](https://user-images.githubusercontent.com/79515820/147749099-1e081d5e-7952-42e7-8212-9b1fc41f2d11.png)
![67](https://user-images.githubusercontent.com/79515820/147749102-6252309a-4554-4107-88b9-5d21f0c65e1e.png)

### Project 2-3 실행결과
- 앞에서와 같은 방법으로 xv6를 실행해서 p2_mlfq_test 파일을 실행한 결과 다음과 같이 결과가나온 것을 확인할 수 있다. 각 프로세스가 test1에서는 20000번, test2에서는 50000번, test3에서는 10000번, test4에서는 25000번 반복문을 돌며 L0큐에 있으면 cnt[0], L1큐에 있으면 cnt[1]값을 올리며 이 둘의 값을 합치면 정상적으로 제대로 count가 된 것을 확인할 수 있다. test4의 경우 마지막 프로세스가 monopolize 되기 때문에 L0큐에서 25000번의 연산을 모두 하였고, 끝난 것도 가장 먼저 끝난 것을 확인할 수 있다.

## Project 3

### 목표
* (Project 3) 기존 xv6의 inode에서 한 번의 indirect를 지원했던 것에서 확장하여, 더 큰 파일을 다룰 수 있도록 double indirect를 구현한다.
