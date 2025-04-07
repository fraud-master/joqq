# Reentrant Lock (재진입 락) 설명

Reentrant Lock은 Java의 동시성 제어 메커니즘 중 하나로, `java.util.concurrent.locks` 패키지에 있는 `ReentrantLock` 클래스를 통해 제공됩니다. 이는 Java 5부터 도입되었으며 기존 `synchronized` 키워드보다 더 유연한 락킹 기능을 제공합니다.

## 1. 기본 개념

재진입 락(Reentrant Lock)은 이름에서 알 수 있듯이 "재진입"이 가능한 락입니다. 여기서 재진입이란 이미 락을 획득한 스레드가 다시 같은 락을 획득할 수 있다는 의미입니다. 이것은 중첩된 메소드 호출이나 재귀적 코드에서 매우 유용합니다.

## 2. ReentrantLock의 주요 특징

### 2.1. 명시적 락킹 제어
```java
Lock lock = new ReentrantLock();
lock.lock();  // 락 획득
try {
    // 임계 영역 코드
} finally {
    lock.unlock();  // 락 해제 (항상 finally 블록에서 해제)
}
```

### 2.2. 락 획득 시도 (타임아웃 지원)
```java
boolean acquired = lock.tryLock(1, TimeUnit.SECONDS);
if (acquired) {
    try {
        // 1초 내에 락을 획득했을 경우 실행되는 코드
    } finally {
        lock.unlock();
    }
} else {
    // 1초 내에 락을 획득하지 못했을 경우 실행되는 코드
}
```

### 2.3. 인터럽트 가능한 락 획득
```java
try {
    lock.lockInterruptibly();
    try {
        // 임계 영역 코드
    } finally {
        lock.unlock();
    }
} catch (InterruptedException e) {
    // 락 획득 대기 중에 인터럽트 발생 시 처리 코드
}
```

### 2.4. 공정성 지원
```java
// 공정 락 생성 (FIFO 순서로 대기 스레드에 락 제공)
Lock fairLock = new ReentrantLock(true);
```

### 2.5. 락 상태 확인 메소드
```java
ReentrantLock lock = new ReentrantLock();

// 현재 락이 누군가에 의해 보유되고 있는지 확인
boolean isLocked = lock.isLocked();

// 현재 스레드가 이 락을 보유하고 있는지 확인
boolean heldByCurrentThread = lock.isHeldByCurrentThread();

// 현재 대기 중인 스레드 수 확인
int waitingThreads = lock.getQueueLength();

// 현재 락의 홀드 카운트 확인 (재진입 횟수)
int holdCount = lock.getHoldCount();
```

## 3. ReentrantLock vs synchronized

### 3.1. ReentrantLock의 장점
- **타임아웃 지원**: 락 획득을 특정 시간 동안만 시도할 수 있음
- **인터럽트 지원**: 락 획득 대기 중 인터럽트 처리 가능
- **공정성 지원**: 먼저 대기한 스레드가 우선 락을 획득하도록 보장 가능
- **락 폴링**: 락 획득 시도 후 실패 시 다른 작업 수행 가능
- **락 상태 확인**: 현재 락의 상태 확인 가능
- **조건 변수 지원**: 여러 개의 Condition 객체로 대기/통지 메커니즘 세분화 가능

### 3.2. synchronized의 장점
- **문법적 간결함**: 블록 또는 메소드 단위로 간단히 사용 가능
- **자동 락 해제**: try-finally로 명시적 unlock 불필요
- **최적화**: JVM이 지속적으로 최적화하여 성능 개선 가능

## 4. Condition 사용 (조건 변수)

ReentrantLock은 Condition 객체를 통해 더 세밀한 스레드 통신을 지원합니다.

```java
ReentrantLock lock = new ReentrantLock();
Condition notFull = lock.newCondition();
Condition notEmpty = lock.newCondition();

// 생산자 코드
lock.lock();
try {
    while (isFull()) {
        notFull.await();  // 버퍼가 가득 찬 경우 대기
    }
    // 데이터 추가 작업
    notEmpty.signal();  // 버퍼에 데이터가 있음을 소비자에게 알림
} finally {
    lock.unlock();
}

// 소비자 코드
lock.lock();
try {
    while (isEmpty()) {
        notEmpty.await();  // 버퍼가 비어있는 경우 대기
    }
    // 데이터 소비 작업
    notFull.signal();  // 버퍼에 공간이 생겼음을 생산자에게 알림
} finally {
    lock.unlock();
}
```

## 5. 주의사항

1. **항상 finally 블록에서 unlock 호출**: try-finally 패턴으로 예외 발생 시에도 락이 해제되도록 보장

2. **데드락 방지**: 여러 락을 사용할 때는 항상 동일한 순서로 획득하도록 주의

3. **락 누락 방지**: unlock() 호출을 빼먹지 않도록 주의

4. **성능 고려**: 짧은 임계 영역에는 synchronized가 더 효율적일 수 있음

5. **재진입 횟수 제한**: 이론적으로는 무제한이지만, 실제로는 Integer.MAX_VALUE로 제한됨

## 6. 사용 예제: 완전한 구현

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class BoundedBuffer<E> {
    private final E[] items;
    private int putPosition, takePosition, count;
    
    private final ReentrantLock lock;
    private final Condition notFull;
    private final Condition notEmpty;
    
    @SuppressWarnings("unchecked")
    public BoundedBuffer(int capacity) {
        items = (E[]) new Object[capacity];
        lock = new ReentrantLock();
        notFull = lock.newCondition();
        notEmpty = lock.newCondition();
    }
    
    public void put(E item) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length)
                notFull.await();
            
            items[putPosition] = item;
            if (++putPosition == items.length)
                putPosition = 0;
            ++count;
            
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }
    
    public E take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0)
                notEmpty.await();
            
            E result = items[takePosition];
            items[takePosition] = null;
            if (++takePosition == items.length)
                takePosition = 0;
            --count;
            
            notFull.signal();
            return result;
        } finally {
            lock.unlock();
        }
    }
}
```

## 7. ReentrantLock 실제 사용 시나리오

- **복잡한 동시성 요구사항**: 타임아웃, 인터럽트, 조건 변수 등이 필요한 경우
- **락 획득 시도 필요**: 락을 얻지 못하면 다른 작업을 수행해야 하는 경우
- **락 공정성 필요**: 스레드 기아 현상을 방지해야 하는 경우
- **세밀한 스레드 통신**: 여러 조건 변수를 사용해야 하는 경우
- **세밀한 락 제어**: 락 상태 정보를 확인하거나 다양한 락 정책이 필요한 경우

ReentrantLock은 강력하고 유연한 도구이지만, 그만큼 올바르게 사용하려면 세심한 주의가 필요합니다. 간단한 동기화에는 synchronized를 사용하고, 더 복잡한 요구사항이 있을 때 ReentrantLock을 사용하는 것이 좋은 전략입니다.
