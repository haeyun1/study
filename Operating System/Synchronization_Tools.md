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

![image.png](attachment:47b6581b-b9cb-4edf-8f2d-acdc50904b18:image.png)
![[Pasted image 20250606192459.png]]
Critical section problem

프로세스들이 데이터를 협력적으로 공유하기 위해 자신들의 활동을 동기화하는 데 사용할 수 있는 프로토콜을 설계하는 것

- Entry section: critical section에 진입할 허가를 요청하는 코드 부분
- Critical section: shared resource를 바꿀 수 있는 코드 부분
- Exit section: critical section을 벗어났음을 알리는 코드 부분
- Remainder section: shared resource를 바꾸지 않는 코드 부분

Critical-Section Handling in OS