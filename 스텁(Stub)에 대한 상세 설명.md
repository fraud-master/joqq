# 스텁(Stub)에 대한 상세 설명

스텁(Stub)은 테스트 더블(Test Double)의 한 유형으로, 실제 구현체를 대체하여 테스트 중인 코드에 미리 정의된 응답을 제공하는 간단한 가짜 객체입니다. 스텁은 테스트 대상 시스템(SUT, System Under Test)이 의존하는 컴포넌트를 대체하여 테스트를 단순화하고 격리시키는 데 사용됩니다.

## 스텁의 핵심 특징

### 1. 상태 기반 검증(State-Based Verification)
- 스텁은 주로 **상태 검증**에 초점을 맞춥니다.
- 테스트 대상 코드의 최종 상태나 결과를 검증하는 데 사용됩니다.
- 스텁 자체의 상호작용이나 호출 방식을 검증하지 않습니다.

### 2. 미리 정의된 응답(Canned Answers)
- 특정 메서드 호출에 대해 미리 정의된 응답을 반환합니다.
- 응답은 고정된 값, 계산된 값, 예외 등이 될 수 있습니다.
- 입력 매개변수에 따라 다른 응답을 반환하도록 프로그래밍할 수 있습니다.

### 3. 단순함(Simplicity)
- 스텁은 일반적으로 목(Mock)보다 단순한 구현을 가집니다.
- 테스트에 필요한 최소한의 기능만 구현합니다.
- 복잡한 검증 로직이나 상태 추적 기능이 없습니다.

### 4. 일관된 응답(Consistent Responses)
- 동일한 입력에 대해 일관된 응답을 제공합니다.
- 테스트 실행 간에 예측 가능한 환경을 만듭니다.

## 스텁 구현 방법

### 1. 인터페이스 구현을 통한 방법
```java
// 원본 인터페이스
public interface UserRepository {
    User findById(String id);
    List<User> findAll();
    void save(User user);
}

// 스텁 구현
public class UserRepositoryStub implements UserRepository {
    @Override
    public User findById(String id) {
        // 항상 같은 사용자 객체를 반환
        return new User("test-id", "Test User", "test@example.com");
    }
    
    @Override
    public List<User> findAll() {
        // 미리 정의된 사용자 목록 반환
        return Arrays.asList(
            new User("user1", "User One", "one@example.com"),
            new User("user2", "User Two", "two@example.com")
        );
    }
    
    @Override
    public void save(User user) {
        // 아무것도 하지 않음 (필요 없는 기능)
    }
}
```

### 2. 익명 클래스를 사용한 방법
```java
UserRepository userRepositoryStub = new UserRepository() {
    @Override
    public User findById(String id) {
        return new User("test-id", "Test User", "test@example.com");
    }
    
    @Override
    public List<User> findAll() {
        return Collections.emptyList();
    }
    
    @Override
    public void save(User user) {
        // do nothing
    }
};
```

### 3. 람다식을 사용한 방법 (함수형 인터페이스의 경우)
```java
// 단일 메서드 인터페이스
public interface UserFinder {
    User findById(String id);
}

// 람다를 사용한 스텁
UserFinder finderStub = id -> new User("test-id", "Test User", "test@example.com");
```

### 4. 스텁 프레임워크 사용
```java
// Mockito를 스텁 생성에 사용 (행위 검증 없이)
UserRepository userRepositoryStub = mock(UserRepository.class);
when(userRepositoryStub.findById("user1")).thenReturn(new User("user1", "John Doe", "john@example.com"));
when(userRepositoryStub.findById("invalid")).thenReturn(null);

// JUnit 5 + Mockito
@Test
void testUserService(@Mock UserRepository userRepositoryStub) {
    when(userRepositoryStub.findById(anyString())).thenReturn(new User("test", "Test User", "test@example.com"));
    // 테스트 코드
}
```

## 스텁의 다양한 응용

### 1. 조건부 응답
```java
// 다양한 입력에 대해 다른 응답 제공
public class ConditionalStub implements DataService {
    @Override
    public Data fetchData(String key) {
        if ("valid".equals(key)) {
            return new Data("Valid data");
        } else if ("empty".equals(key)) {
            return new Data("");
        } else {
            return null;
        }
    }
}
```

### 2. 예외 던지기
```java
public class ExceptionThrowingStub implements NetworkService {
    @Override
    public Response send(Request request) {
        if (request.getUrl().contains("error")) {
            throw new NetworkException("Network unavailable");
        }
        return new Response(200, "OK");
    }
}
```

### 3. 상태 기반 응답
```java
public class StatefulStub implements Database {
    private boolean isConnected = false;
    
    @Override
    public void connect() {
        isConnected = true;
    }
    
    @Override
    public Result query(String sql) {
        if (!isConnected) {
            throw new DatabaseException("Not connected");
        }
        return new Result("Query result");
    }
    
    @Override
    public void disconnect() {
        isConnected = false;
    }
}
```

### 4. 지연 응답
```java
public class DelayedStub implements FileSystem {
    @Override
    public byte[] readFile(String path) {
        try {
            // 네트워크 지연 시뮬레이션
            Thread.sleep(500);
            return "file content".getBytes();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return new byte[0];
        }
    }
}
```

## 스텁 사용 시나리오

### 1. 데이터베이스 대체
```java
@Test
public void testUserService() {
    // 데이터베이스 액세스를 스텁으로 대체
    UserRepository stubRepository = new UserRepositoryStub();
    
    // 테스트 대상 객체에 스텁 주입
    UserService userService = new UserService(stubRepository);
    
    // 테스트 실행
    User user = userService.getUserById("any-id");
    
    // 검증
    assertNotNull(user);
    assertEquals("Test User", user.getName());
}
```

### 2. 외부 API 호출 대체
```java
@Test
public void testWeatherNotification() {
    // 외부 날씨 API를 호출하는 대신 스텁 사용
    WeatherApiClient stubClient = new WeatherApiClientStub();
    
    // 테스트 대상 객체에 스텁 주입
    WeatherNotifier notifier = new WeatherNotifier(stubClient);
    
    // 테스트 실행
    boolean notified = notifier.notifyIfRaining("Seoul");
    
    // 검증
    assertTrue(notified);
}
```

### 3. 오류 상황 테스트
```java
@Test
public void testPaymentFailure() {
    // 항상 실패하는 결제 게이트웨이 스텁
    PaymentGateway failingGateway = new FailingPaymentGatewayStub();
    
    // 테스트 대상 객체에 스텁 주입
    PaymentProcessor processor = new PaymentProcessor(failingGateway);
    
    // 테스트 실행 및 예외 검증
    assertThrows(PaymentException.class, () -> {
        processor.processPayment(100.0, "4111-1111-1111-1111");
    });
}
```

## 스텁의 장단점

### 장점
1. **단순성**: 구현이 단순하고 이해하기 쉽습니다.
2. **제어 가능성**: 테스트 환경을 완전히 제어할 수 있습니다.
3. **예측 가능성**: 테스트 실행이 일관되고 예측 가능합니다.
4. **격리성**: 테스트 대상 코드를 외부 의존성으로부터 격리시킵니다.
5. **속도**: 실제 의존성(예: 데이터베이스, 네트워크)보다 빠르게 실행됩니다.

### 단점
1. **유지보수**: 수작업으로 구현한 스텁은 실제 의존성이 변경될 때 함께 업데이트해야 합니다.
2. **검증 부족**: 상호작용 검증 기능이 없어 테스트 대상 코드가 의존성을 올바르게 사용하는지 확인하기 어렵습니다.
3. **과도한 단순화**: 때로는 실제 환경의 복잡성을 충분히 반영하지 못할 수 있습니다.

## 스텁과 다른 테스트 더블 비교

### 스텁 vs 목(Mock)
- **스텁**: 미리 정의된 응답을 제공하지만, 호출 여부나 방법을 검증하지 않습니다.
- **목**: 기대한 호출이 발생했는지 검증하는 기능이 추가됩니다. 행위 검증에 중점을 둡니다.

### 스텁 vs 페이크(Fake)
- **스텁**: 단순한 응답만 제공합니다.
- **페이크**: 실제 구현체의 간소화된 버전으로, 일부 실제 로직을 포함합니다(예: 인메모리 데이터베이스).

### 스텁 vs 스파이(Spy)
- **스텁**: 응답 제공이 주요 목적이며 호출 정보를 기록하지 않습니다.
- **스파이**: 스텁의 기능에 추가로 호출 정보를 기록하여 나중에 검증할 수 있습니다.

### 스텁 vs 더미(Dummy)
- **스텁**: 특정 메서드 호출에 대해 미리 정의된 응답을 제공합니다.
- **더미**: 아무 기능도 구현하지 않으며, 단지 인자 목록을 채우는 용도로만 사용됩니다.

## 스텁 사용 모범 사례

### 1. 필요한 메서드만 구현
```java
// 인터페이스의 모든 메서드 구현이 필요하지 않을 때
public class MinimalStub implements ComplexInterface {
    // 테스트에 필요한 메서드만 구현
    @Override
    public Data getImportantData() {
        return new Data("Test data");
    }
    
    // 나머지 메서드는 빈 구현 또는 UnsupportedOperationException 발생
    @Override
    public void unusedMethod1() {
        throw new UnsupportedOperationException("Not implemented in stub");
    }
    
    // ... 기타 메서드
}
```

### 2. 스텁 팩토리 사용
```java
// 다양한 스텁 설정을 위한 팩토리
public class UserRepositoryStubFactory {
    public static UserRepository createEmptyRepository() {
        return new UserRepositoryStub(Collections.emptyList());
    }
    
    public static UserRepository createWithUsers(User... users) {
        return new UserRepositoryStub(Arrays.asList(users));
    }
    
    public static UserRepository createThrowingRepository() {
        return new ThrowingUserRepositoryStub();
    }
}
```

### 3. 테스트 특화 스텁
```java
// 특정 테스트 시나리오를 위한 스텁
@Test
public void testUserNotFound() {
    // 항상 null을 반환하는 스텁
    UserRepository alwaysEmptyStub = id -> null;
    
    UserService service = new UserService(alwaysEmptyStub);
    assertThrows(UserNotFoundException.class, () -> service.getUserById("any-id"));
}
```

### 4. 프레임워크 활용
```java
// 프레임워크를 활용한 스텁 생성 (Mockito 사용)
@Test
public void testWithFramework() {
    // 스텁 생성 (행위 검증 없이 스텁으로만 사용)
    UserRepository stubRepo = mock(UserRepository.class);
    
    // 스텁 동작 정의
    when(stubRepo.findById("existing")).thenReturn(new User("existing", "Existing User"));
    when(stubRepo.findById("error")).thenThrow(new DatabaseException());
    
    // 이 테스트에서는 verify() 호출을 하지 않음 (순수 스텁 용도)
}
```

스텁은 단위 테스트에서 외부 의존성을 격리시키고 예측 가능한 테스트 환경을 제공하는 데 매우 효과적인 도구입니다. 복잡한 행위 검증이 필요하지 않은 경우 목보다 간단하고 직관적인 스텁을 사용하는 것이 테스트 가독성과 유지보수성을 높이는 데 도움이 됩니다.
