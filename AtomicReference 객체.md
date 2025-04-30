# AtomicReference 객체

`AtomicReference`는 Java의 `java.util.concurrent.atomic` 패키지에 포함된 클래스로, 객체 참조에 대한 원자적(atomic) 연산을 제공합니다. 주로 멀티스레드 환경에서 락(lock)을 사용하지 않고도 안전하게 객체 참조를 업데이트할 때 사용합니다.

## 주요 특징

1. **원자성 보장**: 여러 스레드가 동시에 접근해도 데이터 일관성이 보장됩니다.
2. **락 프리(Lock-free)**: 전통적인 동기화 메커니즘인 `synchronized`보다 성능이 우수할 수 있습니다.
3. **참조 타입 지원**: 모든 객체 타입에 대한 원자적 연산을 제공합니다.

## 주요 메서드

- `get()`: 현재 저장된 참조 값을 반환합니다.
- `set(V newValue)`: 새로운 값으로 원자적으로 설정합니다.
- `getAndSet(V newValue)`: 현재 값을 반환하고 새 값을 설정합니다.
- `compareAndSet(V expect, V update)`: 현재 값이 예상 값과 같으면 새 값으로 업데이트하고 true를 반환합니다.
- `updateAndGet(UnaryOperator<V> updateFunction)`: 함수를 적용한 새 값으로 원자적으로 업데이트합니다.
- `getAndUpdate(UnaryOperator<V> updateFunction)`: 현재 값을 반환하고 함수를 적용한 새 값으로 업데이트합니다.

## 사용 예제

```java
import java.util.concurrent.atomic.AtomicReference;

public class AtomicReferenceExample {
    public static void main(String[] args) {
        // AtomicReference 생성
        AtomicReference<String> atomicString = new AtomicReference<>("초기값");
        
        // 현재 값 가져오기
        String currentValue = atomicString.get();
        System.out.println("현재 값: " + currentValue);
        
        // 값 설정하기
        atomicString.set("새로운 값");
        System.out.println("변경 후 값: " + atomicString.get());
        
        // CAS(Compare-And-Swap) 패턴 사용
        boolean wasUpdated = atomicString.compareAndSet("새로운 값", "업데이트된 값");
        System.out.println("업데이트 성공?: " + wasUpdated);
        System.out.println("현재 값: " + atomicString.get());
        
        // 잘못된 예상값으로 업데이트 시도
        wasUpdated = atomicString.compareAndSet("다른 값", "이 값은 설정되지 않음");
        System.out.println("업데이트 성공?: " + wasUpdated);
        System.out.println("현재 값: " + atomicString.get()); // 변경되지 않음
        
        // 함수형 업데이트
        String oldValue = atomicString.getAndUpdate(value -> value + " + 추가된 텍스트");
        System.out.println("이전 값: " + oldValue);
        System.out.println("현재 값: " + atomicString.get());
    }
}
```

## 복잡한 객체 업데이트 예제

```java
import java.util.concurrent.atomic.AtomicReference;

class User {
    private final String name;
    private final int age;
    
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    public User withAge(int newAge) {
        return new User(this.name, newAge);
    }
    
    @Override
    public String toString() {
        return "User{name='" + name + "', age=" + age + '}';
    }
}

public class AtomicReferenceUserExample {
    public static void main(String[] args) {
        AtomicReference<User> userRef = new AtomicReference<>(new User("Alice", 25));
        
        // 여러 스레드에서 안전하게 나이 증가시키기
        Thread t1 = new Thread(() -> {
            User oldUser, newUser;
            do {
                oldUser = userRef.get();
                newUser = oldUser.withAge(oldUser.getAge() + 1);
            } while (!userRef.compareAndSet(oldUser, newUser));
            
            System.out.println("Thread 1 updated: " + userRef.get());
        });
        
        Thread t2 = new Thread(() -> {
            User oldUser, newUser;
            do {
                oldUser = userRef.get();
                newUser = oldUser.withAge(oldUser.getAge() + 5);
            } while (!userRef.compareAndSet(oldUser, newUser));
            
            System.out.println("Thread 2 updated: " + userRef.get());
        });
        
        t1.start();
        t2.start();
        
        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        System.out.println("Final user: " + userRef.get());
    }
}
```

`AtomicReference`는 특히 CAS(Compare-And-Swap) 연산을 통해 락 없이도 여러 스레드 간 경쟁 상태(race condition)를 방지하면서 효율적인 동시성 제어를 가능하게 합니다.
