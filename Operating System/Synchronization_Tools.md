## Synchronization Problem

### The Producer/Consumer Problem
- Producer : data items를 생산하고, buffer (circular queue로 구현된 shared memory)에 넣음
- Consumer : buffer에서 data items를 꺼내고 소비함

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

### Locks

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
```
#include <pthread.h>
pthread_mutex_t mutex; // global declaration

// create and initialize the mutext lock
pthread_mutex_init(&mutex, NULL); // call before first lock
// or
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
```
- Acquiring and releasing the lock
```
pthread_mutex_lock(&mutex); // acquire the mutex lock
pthread_mutex_unlock(&mutex); // release the mutex lock
```

Example: Pthread Locks
```
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

### Building a Lock with Hardware Support

Requirements for Lock
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

Controlling Interrupts
