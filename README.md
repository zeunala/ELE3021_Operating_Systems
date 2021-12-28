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
  

## Project 3

### 목표
* (Project 3) 기존 xv6의 inode에서 한 번의 indirect를 지원했던 것에서 확장하여, 더 큰 파일을 다룰 수 있도록 double indirect를 구현한다.
