스텁(Stub)과 목(Mock)의 차이점에 대해 상세히 설명해 드리겠습니다.

## 스텁(Stub)

스텁은 테스트 중인 코드가 의존하는 컴포넌트를 대체하는 간단한 구현체입니다.

### 스텁의 주요 특징:
- **목적**: 테스트 대상 코드가 의존하는 외부 시스템을 대체하여 미리 정의된 응답을 제공합니다.
- **동작 방식**: 호출되면 미리 프로그래밍된 응답을 반환합니다.
- **검증 기능**: 스텁 자체는 호출 여부나 호출 방법을 검증하지 않습니다.
- **복잡도**: 일반적으로 단순하며, 테스트 성공을 위한 최소한의 기능만 구현합니다.

### 스텁 사용 사례:
- 데이터베이스나 외부 API처럼 실행하기 어렵거나 느린 의존성을 대체할 때
- 특정 시나리오(예: 오류 상황)를 테스트하기 위해 특정 응답을 반환해야 할 때
- 의존 객체가 완성되지 않았을 때 임시로 대체할 때

## 목(Mock)

목은 스텁보다 더 발전된 형태로, 예상되는 호출을 검증하는 기능이 추가되어 있습니다.

### 목의 주요 특징:
- **목적**: 테스트 대상 코드와 의존 객체 간의 상호작용을 검증합니다.
- **동작 방식**: 스텁처럼 미리 정의된 응답을 제공하지만, 추가로 호출 여부, 횟수, 순서, 전달된 인자 등을 기록합니다.
- **검증 기능**: 목 객체가 예상대로 호출되었는지 검증하는 기능을 제공합니다.
- **복잡도**: 스텁보다 복잡하며, 행위(behavior) 검증에 초점을 맞춥니다.

### 목 사용 사례:
- 테스트 대상 코드가 의존 객체와 올바르게 상호작용하는지 검증할 때
- 특정 메서드가 정확한 매개변수로 호출되었는지 확인할 때
- 호출 순서나 횟수가 중요한 경우

## 스텁과 목의 핵심 차이점

1. **검증 초점**:
    - 스텁: 상태(state) 검증에 사용됩니다. 테스트 대상 코드의 최종 상태만 검증합니다.
    - 목: 행위(behavior) 검증에 사용됩니다. 테스트 대상 코드가 의존 객체와 어떻게 상호작용하는지 검증합니다.

2. **사용 목적**:
    - 스텁: "이런 입력이 들어오면 이런 출력을 내보내라"는 단순한 대체제입니다.
    - 목: "이 메서드가 특정 매개변수로 정확히 N번 호출되어야 한다"는 엄격한 기대치를 설정합니다.

3. **구현 복잡도**:
    - 스텁: 간단하고 최소한의 기능만 구현합니다.
    - 목: 호출 검증 로직이 추가되어 더 복잡합니다.

## 코드 예시

### 자바에서의 스텁 예시:
```java
// 스텁 구현
public class UserRepositoryStub implements UserRepository {
    @Override
    public User findById(String id) {
        // 항상 같은 사용자 객체를 반환
        return new User("test-id", "Test User", "test@example.com");
    }
}

// 테스트 코드
@Test
public void testGetUserName() {
    UserService userService = new UserService(new UserRepositoryStub());
    String name = userService.getUserName("any-id");
    assertEquals("Test User", name);
}
```

### 자바에서의 목 예시 (Mockito 라이브러리 사용):
```java
@Test
public void testSendWelcomeEmail() {
    // 목 객체 생성
    EmailService emailService = mock(EmailService.class);
    UserRepository userRepository = mock(UserRepository.class);
    
    // 스텁 동작 정의
    when(userRepository.findById("user-id")).thenReturn(new User("user-id", "Test User", "test@example.com"));
    
    // 테스트 대상 객체 생성
    UserService userService = new UserService(userRepository, emailService);
    
    // 메서드 실행
    userService.sendWelcomeEmail("user-id");
    
    // 상호작용 검증
    verify(emailService).sendEmail("test@example.com", "Welcome!", "Welcome to our service, Test User!");
}
```

## 사용 시 고려사항

1. **적절한 도구 선택**:
    - 단순히 외부 의존성을 대체하려면 스텁이 적합합니다.
    - 테스트 대상 코드가 다른 객체와 올바르게 상호작용하는지 검증하려면 목이 적합합니다.

2. **과도한 목 사용 주의**:
    - 목을 과도하게 사용하면 테스트가 구현 세부사항에 지나치게 결합되어 유지보수가 어려워질 수 있습니다.
    - 때로는 행위보다 최종 결과(상태)를 검증하는 것이 더 중요할 수 있습니다.

3. **효과적인 조합**:
    - 많은 경우 스텁과 목을 함께 사용하는 것이 효과적입니다.
    - 외부 의존성은 스텁으로 대체하고, 중요한 상호작용은 목으로 검증할 수 있습니다.

테스트 전략에 따라 스텁과 목을 적절히 선택하여 효과적인 테스트 코드를 작성하는 것이 중요합니다.
