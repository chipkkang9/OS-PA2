OS-PA2
태그	
Goal!
: 지금까지 배웠던 process scheduling 방법들을 구현해보는 것이 목표



Problem Specification
Basics
시뮬레이터는 ticks 변수로 시간을 유지한다. scheduling이 발생하면 1이 증가한다. 읽을 수는 있지만, 수정은 불가능
우선, scheduling 가능한 process entity를 정의한다. 시뮬레이터는 시뮬레이션할 프로세스를 설명하는 인수로 프로세스 설명 파일을 허용한다. 아래는 두 가지 프로세스에 대한 예이다.
process 1
  start 0
  lifespan 4
  prio 0
end

process 2
  start 5
  lifespan 10
  prio 10
end


위 예에 따르면, ticks = 0 에서 process 1을 생성하고, ticks = 4까지(lifespan 4)만큼 시간이 지날 때까지 process 1을 유지한다.
prio 값이 클 경우 process의 우선 순위가 높음을 의미한다. 마찬가지로, process 2는 ticks 5의 시점에서 시작되고 10의 시간(ticks) 동안 유지된다.
이 정보는 프로그램 실행 시작 시에도 다음과 같이 표시된다.
- Process 1: Forked at tick 0 and run for 4 ticks with initial priority 0
- Process 2: Forked at tick 5 and run for 10 ticks with initial priority 10


프로세스가 예약되어 있고, 1개의 tick 동안 1개의 tick 만큼 시간이 지난다.
시뮬레이터는 process.h에 정의된 struct process로 process를 구현할 수 있다.
⇒ 시스템의 process를 설명하는 field는 파일을 참조하도록 한다.

두 개의 언더바(__)로 시작하는 변수에 접근하는 것은 금지한다.
struct process *current는 현재 실행 중인 프로세스를 가리킨다. → 현재 실행중인 프로세스에 접근해야 할 때마다 변수를 사용할 수 있다.


Interacting with the framework
구현하는 시뮬레이터는 scheduling 메커니즘(ex. replacing current, counting ticks 등)을 구현하며, sched.h에 있는 struct scheduler로 정의된 scheduling 방법과 상호작용한다.
struct scheduler는 함수 포인터의 모음으로 구성되어 있다.
FIFO scheduler를 구현하는 pa2.c의 fifo_scheduler(sched.c)를 참고하자.
또 다른 instance들 또한 존재한다.


struct process *(*filename)(filename)는 scheduling의 핵심 기능이다.
시뮬레이터는 다음에 실행할 프로세스를 예약할 때마다 이 기능을 호출한다.
따라서 함수는 다음에 실행할 프로세스를 반환하거나, NULL을 반환하여 실행할 프로세스가 없음을 프레임워크에 나타내야 한다.
pa2.c의 fifo_schedule() 참조하기


시뮬레이터에는 struct list_head readyqueue라고 하는 readyqueue가 존재한다. 
readyqueue에서는 실행 준비된 프로세스 목록을 저장해야 한다.
실행 중인 프로세스는 준비 대기열에 존재하면 안 된다.


프로세스가 시뮬레이터에 의해 생성되면 forked() 콜백 함수가 호출된다.
마찬가지로 프로세스가 완료되면 exiting() 콜백 함수가 호출된다.


Simulating resources
시스템에는 프로세스에만 할당할 수 있는 16개의 시스템 리소스가 있다.
struct resource는 resource.h에 있는 시스템 리소스를 추상화한다.
프로세스는 시뮬레이터에게 리소스를 획득하고 사용 이후 리소스를 해제하도록 요청할 수 있다.
이런 리소스의 사용은 acquire 키워드를 사용하여 프로세스 설명 파일에 지정된다.
ex. acquire 1 4 2는 프로세스가 4 tick에 대해 경과된 리소스 #1을 필요로 하고 획득한 후에는 2 tick에 대해 리소스를 사용한다는 것을 의미한다.
testcase/resources 살펴보기


시뮬레이터는 리소스 획득 요청을 받으면 acquire() 함수를 호출한다.
마찬가지로 시뮬레이터는 프로세스가 리소스 사용을 마치면 release() 함수를 호출한다.
FIFO scheduler에서 사용하는 기본 FCFS acquire/release 함수는 pa2.c에서 찾을 수 있다.


비우선순위 기반(Non-priority-based) scheduling 정책은 선착순 방식으로 리소스 획득 요청을 처리해야 한다.
반면에 우선 순위 기반 예약 방식(first-come-first-served way)은 released 리소스를 가장 높은 우선 순위를 가진 프로세스로 전송해야 한다.
이를 위해 자신만의 acquire/release 함수를 정의하고 scheduler 구현과 연결하여 올바른 scheduling 결정을 내릴 수 있다.
Priority가 동일한 두 프로세스가 동일한 리소스를 요청하는 경우 이전 프로세스가 리소스를 수신한다.


Scheduling Policies
시뮬레이터는 
Shortest-Job First(SJF) scheduler
Shortest Time-to-Complete First(STCF) scheduler
Round-Robin scheduler
Base Priority-based schduler
Priority-based scheduler with aging(PA)
Priority-based scheduler with priority ceiling protocol(PCP)
Priority-based scheduler with priority inheritance protocol (PIP)
로 구분된다.


시작 옵션과 함께 실행할 scheduler를 선택할 수 있으며, 시뮬레이터는 해당 scheduler를 자동으로 사용하도록 설정된다. ⇒ 옵션 없이 프로그램을 실행하여 옵션을 확인할 수 있다.


FCFS와 SJF는 non-preemptive이다.
시뮬레티어가 scheduler에게 tick마다 실행할 다음 프로세스를 선택할 수도 있지만, scheduler는 완료되지 않는 한 현재 실행 중인 프로세스를 변경해서는 안 된다.


STCF scheduler는 priority가 높은 프로세스가 도착하면 현재 실행 중인 프로세스를 선점(preempt)할 수 있지만, 그렇지 않으면 현재 프로세스를 유지해야 한다.


Round-Robin schduler의 경우,
time quantum은 tick과 일치한다. 프레임워크가 scheduler()을 호출하면, time quantum이 만료되었음을 말한다.
RR scheduler를 구현하는 동안은 priority를 무시할 수 있다.


Priority-based scheduler의 경우,
Round-Robin과 같은 방식으로 동일한 우선 순위를 가진 프로세스를 처리해야 한다.
둘 이상의 프로세스가 동일한 우선 순위를 가진 경우 각 tick으로 전환해야 한다.


PA(Priority-based scheduler with aging)의 경우,
scheduling을 하는 순간마다, 현재 프로세스의 priority가 원래의 priority로 재설정되고 준비 대기열의 모든 프로세스가 priority 1만큼 증가한다.
Priority는 process.h에 정의된 MAX_PRIO까지 증가시킬 수 있다.
scheduler는 이 시점에서 조정된 우선 순위가 가장 높은 프로세스를 선택해야 한다.
동일한 priority를 가진 프로세스는 원래 priority scheduler와 마찬가지로 Round-Robin 방식으로 처리되어야 한다.


PCP(Priority-based scheduler with priority ceiling protocol)의 경우,
PCP에서 프로세스의 priority를 증가시키려면 MAX_PRIO를 사용한다.


PIP(Priority-based scheduler with priority inheritance protocol)의 경우,
리소스를 release할 때 프로세스의 priority가 올바르게 설정되어 있는 지 확인해야 한다.
PIP를 구현해야 하는 복잡한 경우들이 존재한다.
서로 다른 priority 값을 가진 둘 이상의 프로세스가 release 리소스를 기다릴 수 있다. 한 프로세스가 하나의 리소스 유형을 보유하고 다른 프로세스가 동일한 리소스 유형을 획득한다고 가정한다. 그런 다음 우선 순위가 더 높은(또는 더 낮은) 다른 프로세스는 리소스 유형을 다시 획득하는 것이다.
서로 다른 priority 값을 가진 많은 프로세스가 보유한 서로 다른 리소스를 기다리고 있다. 이러한 사례가 모두 적절하게 처리될 경우에만 PIP 구현에 대한 만점을 받을 수 있다.
Hint : 리소스 acquition status를 확인하여 release 프로세스의 현재 우선 순위를 계산한다.





Tips and Restriction
프로세스 및 리소스 상황을 보려면 dump_status() 함수를 사용한다.
list_head queue를 구현하기 위해서는 PA0와 PA1을 생각해보도록 하자.
list_for_*_safe 변형을 사용하고 있는지 확인하고, list_del_init을 사용하여 목록을 항목에서 제거하도록 한다.
인터넷 검색을 통해 차이점을 찾도록 한다.
