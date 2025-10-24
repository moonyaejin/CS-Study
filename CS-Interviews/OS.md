## 📋 질문 목록
- [Q1. 뮤텍스(Mutex)와 세마포어(Semaphore)의 차이점](#1-뮤텍스mutex와-세마포어semaphore의-차이점은-무엇인가요)

## 1. 뮤텍스(Mutex)와 세마포어(Semaphore)의 차이점은 무엇인가요?
> 출처: 개발자: 데일리 CS 역량 강화 챌린지 (2025.10.24)

**답변**

뮤텍스와 세마포어는 둘 다 **멀티스레드 환경에서 동시성 제어를 위한 동기화 기법**이지만, 동작 방식과 사용 목적에서 결정적인 차이가 있습니다.

**뮤텍스 (Mutex)**

뮤텍스는 Mutual Exclusion(상호 배제)의 약자로, **공유 자원에 대한 배타적 접근**을 보장하는 잠금 메커니즘입니다.
- **단일 접근**: 오직 한 개의 스레드만 임계 영역에 진입 가능합니다. 화장실 열쇠를 떠올리면 쉽습니다 - 한 사람이 열쇠를 가지고 들어가면 다른 사람은 기다려야 합니다.
- **소유권 보장**: 잠금을 획득한 스레드만이 해제할 수 있습니다. Thread A가 lock()을 호출했다면 반드시 Thread A가 unlock()을 호출해야 합니다.
- **재진입 가능**: 많은 구현에서 같은 스레드가 여러 번 lock을 획득할 수 있는 재귀적 뮤텍스(Recursive Mutex)를 지원합니다.

**세마포어 (Semaphore)**

세마포어는 **정해진 개수의 자원을 관리**하는 신호 메커니즘으로, 카운터를 사용해 동시 접근을 제어합니다.

- **다중 접근 가능**: 카운터 값(N)만큼의 스레드가 동시에 접근 가능합니다. 주차장의 빈 자리 개수를 생각하면 됩니다 - 10대 주차 가능하면 10대까지 들어올 수 있습니다.
- **소유권 없음**: wait()을 호출한 스레드와 signal()을 호출하는 스레드가 달라도 됩니다. 이는 스레드 간 시그널링에 유용합니다.
- **두 가지 종류**:
  - Binary Semaphore: 0과 1만 가지는 세마포어 (뮤텍스와 유사하지만 소유권이 없음)
  - Counting Semaphore: 0부터 N까지 값을 가지는 세마포어

**핵심 차이점 정리**

| 구분 | 뮤텍스 | 세마포어 |
|------|--------|----------|
| **동시 접근** | 1개 스레드만 | N개 스레드 가능 |
| **소유권** | 있음 (lock한 스레드가 unlock) | 없음 (누구나 signal 가능) |
| **카운터** | 없음 (잠김/풀림 상태만) | 있음 (0~N 값 가짐) |
| **주 용도** | 임계 영역 보호 | 자원 카운팅, 스레드 동기화 |
| **속도** | 상대적으로 빠름 | 상대적으로 느림 |

## **예상 꼬리질문**

**Q1. Binary Semaphore와 Mutex는 기능상 거의 비슷해 보이는데, 왜 따로 존재하나요?**

A. 핵심은 **소유권(Ownership)**과 **용도**의 차이입니다.

뮤텍스는 **"내가 잠근 것은 내가 푼다"** 원칙이 있어서 우선순위 상속 같은 고급 기능을 구현할 수 있습니다. 예를 들어 낮은 우선순위 스레드가 뮤텍스를 잡고 있을 때, 높은 우선순위 스레드가 대기하면 일시적으로 우선순위를 올려주는 기능이 가능합니다. 이는 우선순위 역전 문제를 해결합니다.

Binary Semaphore는 소유권이 없어서 **스레드 간 시그널링**에 적합합니다. 생산자-소비자 패턴에서 생산자가 signal()로 "데이터 준비됐어"라고 알리고, 소비자가 wait()으로 기다리는 구조를 만들 수 있습니다. 뮤텍스로는 이런 패턴을 구현하기 어렵습니다.
**세마포어 사용 (적합)**
```java
class ProducerConsumer {
    private Semaphore items = new Semaphore(0);  // 처음엔 아이템 없음
    private Queue<String> queue = new LinkedList<>();
    
    // 생산자
    public void produce() {
        queue.offer("새 아이템");
        items.release();  // "아이템 준비됐어" 신호
    }
    
    // 소비자
    public void consume() throws InterruptedException {
        items.acquire();  // 아이템 있을 때까지 대기
        String item = queue.poll();
        System.out.println("소비: " + item);
    }
}
```

**뮤텍스 사용 (부적합)**
```java
class ProducerConsumerWrong {
    private Object mutex = new Object();
    private Queue<String> queue = new LinkedList<>();
    
    // 생산자
    public void produce() {
        synchronized(mutex) {
            queue.offer("새 아이템");
        }
        // 소비자에게 어떻게 알려주지? 계속 확인해야 함
    }
    
    // 소비자
    public void consume() {
        synchronized(mutex) {
            // 아이템이 있는지 계속 확인해야 함 (비효율적)
            while(queue.isEmpty()) {
                // Busy waiting... 
            }
            queue.poll();
        }
    }
}
```

즉, 뮤텍스는 "보호"가 목적이고, 세마포어는 "정해진 개수만큼 허용하며, 신호와 카운팅"이 목적입니다.
