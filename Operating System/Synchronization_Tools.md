## Synchronization Problem

### The Producer/Consumer Problem
- Producer : data items를 생산하고, buffer (circular queue로 구현된 shared memory)에 넣음
- Consumer : buffer에서 data items를 꺼내고 소비함

### Synchronization Problem
Race condition
- 여러 process (thread)가 같은 데이터에 동시에 접근하려고 할 때 발생한다 (읽기만 하면 상관없음)
- 실행 결과가 접근하는 순서와 관련이 있음

Synchronization
- 시간에 맞춰 일련의 사건들이 동시에 작동하도록 조율하는 것
- 한 타임에 오직 하나의 process만 shared data에 접근 가능하도록 보증해야 함

Critical-Section
- 각 process는 critical section segment of code를 가지고 있음
- 한 process가 critical section에 있다면 다른 process는 critical section에 접근할 수 없음

Critical section problem
프로세스들이 데이터를 협력적으로 공유하기 위해 자신들의 활동을 동기화하는 데 사용할 수 있는 프로토콜을 설계하는 것
- Entry section: critical section에 진입할 허가를 요청하는 코드 부분
- Critical section: shared resource를 바꿀 수 있는 코드 부분
- Exit section: critical section을 벗어났음을 알리는 코드 부분
- Remainder section: shared resource를 바꾸지 않는 코드 부분

Critical-Section Handling in OS
- Non-preemptive
	커널 모드에서 실행 중인 프로세스가 선점되는 것을 허용하지 않음 -> race condition으로부터 자유로움 
- Preemptive kernels
	커널 모드에서 실행 중인 프로세스의 선점을 허용함
	반응성이 더 좋음 -> real-time process에 더 적합함

## Locks

The Basic Idea
- 어떤 critical section이든 single atomic instruction인 것처럼 실행되도록 보장해야 함
- Lock variable (e.g. mutex): lock 상태를 저장함
	- Available (unlocked, free): lock을 가지고 있는 thread가 없음
	- Acquired (locked, held): 하나의 thread가 lock을 가지고 있고 critical section에 있음
- lock()
	- Try to acquire the lock -> 다른 thread가 lock을 가지고 있지 않으면 해당 thread가 lock을 획득함
	- critical section으로 진입한 thread: owner of the lock
- unlock()
	- owner of the lock이 이 함수를 호출하면 lock은 다시 available

Pthread Locks: Mutex
- Creating and initializing the lock
```c
#include <pthread.h>
pthread_mutex_t mutex; // global declaration

// create and initialize the mutext lock
pthread_mutex_init(&mutex, NULL); // call before first lock
// or
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
```
- Acquiring and releasing the lock
```c
pthread_mutex_lock(&mutex); // acquire the mutex lock
pthread_mutex_unlock(&mutex); // release the mutex lock
```

Example: Pthread Locks
```c
#include <stdio.h>
#include <pthread.h>

#define NUM_THREAD 10

void *thread_inc(void *arg);
void *thread_des(void *arg);

long long num = 0;
pthread_mutex_t mutex;

int main(int argc, char *argv[])
{
    pthread_t thread_id[NUM_THREAD];
    int i;

    pthread_mutex_init(&mutex, NULL);

    for (i = 0; i < NUM_THREAD; i++)
    {
        if (i % 2) // even num
            pthread_create(&(thread_id[i]), NULL, thread_inc, NULL);
        else // odd num
            pthread_create(&(thread_id[i]), NULL, thread_des, NULL);
    }

    for (i = 0; i < NUM_THREAD; i++)
    {
        pthread_join(thread_id[i], NULL);
    }

    printf("result: %lld \n", num);
    pthread_mutex_destroy(&mutex);
    return 0;
}

void *thread_inc(void *arg)
{
    int i;
    for (i = 0; i < 100000; i++)
    {
        pthread_mutex_lock(&mutex);
        num += 1;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void *thread_des(void *arg)
{
    int i;
    for (i = 0; i < 100000; i++)
    {
        pthread_mutex_lock(&mutex);
        num -= 1;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}
```

-> result: 0

## Building a Lock with Hardware Support

### Requirements for Lock
- Mutual Exclusion
	lock이 작동하여 여러 thread가 critical section에 진입하는 것을 방지하는가?
- Fairness
	각 thread가 lock을 획득할 공정한 기회를 얻는가?
	- Starvation
	- Progress 
		만약에 critical section에 아무 process도 없다면 다음으로 critical section에 진입할 process를 선택하는 과정은 무한히 연기될 수 없음
	- Bounded Waiting
		어떤 process가 critical section에 들어가겠다고 요청하면 다른 process는 해당 process가 진입하기 전까지 들어갈 수 있는 상한이 존재해야 한다
- Performance
	lock을 사용하면 time overhead가 늘어남

### Controlling Interrupts
- Disable interrupts for critical sections
	가장 초기에 mutual exclusion을 제공하는 데 사용된 solution -> single-processor system을 위해 고안됨

```c
void lock()
{
	DisableInterrupts();
}

void unlock()
{
	EnableInterrupts();
}
```

Problem:
- application에 너무 많은 신뢰를 요구함 -> Greedy of malicious program이 process를 독점할 수 있음
- multiprocessor에서 작동하지 않음
- 최신 CPU에서 interrupt를 mask, unmask하는 코드가 느리게 실행될 수 있음 (overhead)

### Using a Flag
- lock 상태인지 아닌지 나타내는 flag 사용
```c
typedef struct __lock_t {
    int flag;
} lock_t;

void init(lock_t *mutex) {
    // 0 → 락 사용 가능 (available), 1 → 락이 잠김 (held)
    mutex->flag = 0;
}

void lock(lock_t *mutex) {
    while (mutex->flag == 1) // TEST the flag (플래그 테스트)
        ; // spin-wait (아무것도 안 함)
    mutex->flag = 1; // now SET it ! (이제 플래그를 설정)
}

void unlock(lock_t *mutex) {
    mutex->flag = 0;
}
```

- Problem 1: No Mutual Exclusion
	첫 번째 thread가 lock()을 호출해 flag를 1로 바꾸기 전에 두 번째 thread로 전환되면 아직 flag는 0이기 때문에 flag를 1로 바꿀 수 있음 -> 두 thread가 lock을 얻었다고 생각함
- Problem 2: 다른 thread를 기다리는 동안의 Spin-waiting

그래서 Hardware가 지원하는 atomic instruction이 필요하다
- Test-and-Set instruction (atomic exchange)

### Test-And-Set
간단한 lock creation을 지원하는 instruction

```c
int TestAndSet(int *ptr, int new) {
	int old = *ptr; // fetch old value at ptr
	*ptr = new; // store ‘new’ into ptr
	return old; // return the old value
}
```

- 이 operation의 순서는 원자적으로 수행됨

A simple spin lock using Test-And-Set
```c
typedef struct __lock_t {
	int flag;
} lock_t;

void init(lock_t *lock) {
	// 0 indicates that lock is available,
	// 1 that it is held
	lock->flag = 0;
}

void lock(lock_t *lock) {
	while (TestAndSet(&lock->flag, 1) == 1)
		; // spin-wait
}

void unlock(lock_t *lock) {
	lock->flag = 0;
}
```

- single processor에서 올바르게 작동하려면 preemptive scheduler가 요구됨

### Evaluation Spin Locks
- Correctness: yes
	spin lock은 오로지 한 개의 thread만 critical section에 접근할 수 있음

- Fairness: no
	thread spinning may spin forever

- Performance:
		12

