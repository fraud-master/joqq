Thread에서 catch 라인에 인터럽트를 넣는 주된 이유는 인터럽트 상태를 보존하기 위함입니다. 이러한 패턴이 흔히 사용되는 상황을 설명해 드리겠습니다.

## 인터럽트 상태 보존 패턴

일반적으로 다음과 같은 코드 패턴을 볼 수 있습니다:

```java
try {
    // 인터럽트에 응답하는 블로킹 메서드 호출
    Thread.sleep(1000);
    // 또는 lock.lockInterruptibly();
} catch (InterruptedException e) {
    // 인터럽트 상태 복원
    Thread.currentThread().interrupt();
    // 예외 처리 로직
}
```

### 이 패턴이 필요한 이유

1. **인터럽트 상태 초기화 문제**:
    - `InterruptedException`이 발생하면 Java는 해당 스레드의 인터럽트 상태를 자동으로 초기화(clear)합니다.
    - 즉, `Thread.interrupted()`를 호출하면 false를 반환하게 됩니다.

2. **인터럽트 전파**:
    - 메서드가 인터럽트를 처리하고 종료한 후에도, 호출자가 인터럽트 상태를 확인할 수 있어야 합니다.
    - 상위 메서드나 스레드가 적절히 종료 작업을 수행하는 데 필요합니다.

## 구체적인 예시

```java
public void performTask() {
    try {
        // 시간이 오래 걸리는 작업
        doLongRunningTask();
    } catch (InterruptedException e) {
        // 인터럽트 상태 복원
        Thread.currentThread().interrupt();
        // 작업 정리
        cleanup();
        return; // 일찍 반환
    }
    
    // 나머지 작업
    completeTask();
}

private void doLongRunningTask() throws InterruptedException {
    // 블로킹 작업들...
}
```

이 예시에서 `doLongRunningTask()`가 인터럽트되면:
1. `InterruptedException`이 발생합니다.
2. catch 블록에서 `Thread.currentThread().interrupt()`를 호출하여 인터럽트 상태를 복원합니다.
3. 이렇게 하면 `performTask()`를 호출한 상위 코드에서도 인터럽트 상태를 감지할 수 있습니다.

## 주요 이점

1. **인터럽트 연속성 유지**: 인터럽트 신호의 전파 경로를 유지합니다.
2. **취소 메커니즘 보존**: 작업 취소 메커니즘이 상위 수준까지 전파됩니다.
3. **리소스 정리 보장**: 인터럽트 시 리소스를 적절히 정리할 수 있습니다.
4. **응답성 향상**: 시스템이 인터럽트에 적절히 반응할 수 있게 합니다.

이 패턴은 특히 라이브러리 코드나 병렬 실행 프레임워크를 개발할 때 매우 중요하며, 인터럽트 신호를 적절히 전파하여 스레드가 올바르게 종료되도록 돕습니다.
