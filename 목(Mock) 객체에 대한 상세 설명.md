# 목(Mock) 객체에 대한 상세 설명

목(Mock) 객체는 테스트 더블(Test Double)의 한 종류로, 단위 테스트에서 중요한 역할을 합니다. 목 객체는 실제 객체의 행동을 시뮬레이션하면서 동시에 해당 객체와의 상호작용을 검증할 수 있는 기능을 제공합니다.

## 목 객체의 핵심 특징

### 1. 행위 검증(Behavior Verification)
- 목의 가장 중요한 특징은 **행위 검증**에 초점을 맞춘다는 점입니다.
- 테스트 대상 코드가 목 객체와 어떻게 상호작용하는지 세밀하게 검증합니다.
- 특정 메서드가 호출되었는지, 몇 번 호출되었는지, 어떤 매개변수로 호출되었는지, 호출 순서는 어떠한지 등을 확인합니다.

### 2. 기대 설정(Expectations)
- 테스트 실행 전에 목 객체에 대한 기대치를 미리 설정합니다.
- "이 메서드는 이런 매개변수로 정확히 한 번 호출되어야 한다" 등의 기대를 정의합니다.
- 설정된 기대치와 실제 실행 결과를 비교하여 테스트 성공/실패를 결정합니다.

### 3. 응답 스텁(Stubbed Responses)
- 목 객체도 스텁처럼 미리 정의된 응답을 반환할 수 있습니다.
- 다양한 입력에 대해 다른 응답을 정의할 수 있습니다.
- 예외 발생이나 지연 응답 같은 특수한 상황도 시뮬레이션 가능합니다.

### 4. 자기 검증(Self-Verification)
- 테스트 종료 시 목 객체는 모든 기대치가 충족되었는지 스스로 검증합니다.
- 명시적인 `verify()` 호출이나 테스트 프레임워크의 자동 검증 기능을 통해 이루어집니다.

## 목 객체의 종류

### 1. 엄격한 목(Strict Mocks)
- 예상되지 않은 호출이 발생하면 즉시 테스트를 실패시킵니다.
- 정확히 정의된 상호작용만 허용하며, 예상치 못한 호출은 오류로 간주합니다.
- 엄격한 인터페이스 계약을 검증할 때 유용합니다.

### 2. 느슨한 목(Loose Mocks)
- 예상되지 않은 호출을 허용하며 테스트를 실패시키지 않습니다.
- 명시적으로 검증이 필요한 상호작용만 확인합니다.
- 일부 상호작용만 중요한 경우 유용합니다.

### 3. 부분 목(Partial Mocks)
- 실제 객체의 일부 메서드만 오버라이드하여 목으로 만듭니다.
- 나머지 메서드는 실제 구현을 그대로 사용합니다.
- 복잡한 객체의 일부 동작만 목으로 처리할 때 유용합니다.

## 목 프레임워크 기능

대부분의 목 프레임워크는 다음과 같은 기능을 제공합니다:

### 1. 목 객체 생성
```java
// Mockito 예시
UserService mockService = mock(UserService.class);

// JUnit 5 + Mockito
@Mock
private UserService mockService;
```

### 2. 스텁 동작 정의
```java
// 특정 매개변수에 대한 반환값 정의
when(mockService.findById("user1")).thenReturn(new User("user1", "John"));

// 예외 발생 시뮬레이션
when(mockService.findById("invalid")).thenThrow(new NotFoundException());

// 매개변수와 무관하게 항상 같은 값 반환
when(mockService.getCount()).thenReturn(5);

// 연속 호출에 대해 다른 응답 정의
when(mockService.nextValue())
    .thenReturn(1)
    .thenReturn(2)
    .thenReturn(3);
```

### 3. 상호작용 검증
```java
// 특정 메서드가 호출되었는지 검증
verify(mockService).save(any(User.class));

// 정확한 매개변수로 호출되었는지 검증
verify(mockService).findById("user1");

// 호출 횟수 검증
verify(mockService, times(3)).getCount();
verify(mockService, never()).delete(any());
verify(mockService, atLeastOnce()).save(any());
verify(mockService, atMost(2)).update(any());
```

### 4. 매개변수 매칭
```java
// 정확한 값 매칭
verify(mockService).findById("user1");

// 타입 기반 매칭
verify(mockService).save(any(User.class));

// 조건 기반 매칭
verify(mockService).save(argThat(user -> user.getName().startsWith("J")));
```

### 5. 호출 순서 검증
```java
// 호출 순서 검증
InOrder inOrder = inOrder(mockService);
inOrder.verify(mockService).findById("user1");
inOrder.verify(mockService).update(any(User.class));
inOrder.verify(mockService).save(any(User.class));
```

## 목 사용 시 모범 사례

### 1. 행위가 중요한 경우에만 목 사용
- 상호작용 자체가 중요한 경우에만 목을 사용합니다.
- 단순히 의존성을 대체하는 것이 목적이라면 스텁이 더 적합할 수 있습니다.

### 2. 필요한 것만 검증
- 모든 상호작용을 검증하려 하지 말고, 테스트의 목적과 관련된 상호작용만 검증합니다.
- 과도한 검증은 테스트를 구현 세부사항에 지나치게 결합시킵니다.

### 3. 명확한 테스트 구조 유지
```java
// 표준 목 테스트 패턴
@Test
public void sendEmailToUser() {
    // 준비(Arrange): 목 설정 및 스텁 정의
    when(userRepository.findById("user1")).thenReturn(new User("user1", "John", "john@example.com"));
    
    // 실행(Act): 테스트 대상 코드 실행
    notificationService.notifyUser("user1", "Hello!");
    
    // 검증(Assert): 상호작용 검증
    verify(emailService).sendEmail("john@example.com", "Notification", "Hello!");
}
```

### 4. 적절한 추상화 수준 선택
- 너무 낮은 수준의 구현 세부사항을 목으로 검증하면 리팩토링에 취약해집니다.
- 의미 있는 비즈니스 상호작용을 중심으로 목을 설계합니다.

## 목의 장단점

### 장점
1. **정밀한 상호작용 검증**: 객체 간 상호작용을 세밀하게 검증할 수 있습니다.
2. **격리된 테스트**: 외부 의존성을 완전히 제어할 수 있어 테스트 격리성이 높습니다.
3. **곤란한 상황 시뮬레이션**: 실제 환경에서 재현하기 어려운 상황(예: 네트워크 오류)을 쉽게 시뮬레이션할 수 있습니다.
4. **명세 중심 개발**: 인터페이스 명세를 중심으로 개발 가능합니다(TDD와 궁합이 좋음).

### 단점
1. **구현에 결합**: 테스트가 내부 구현 세부사항에 지나치게 결합될 수 있습니다.
2. **과도한 검증**: 모든 상호작용을 검증하려는 경향으로 테스트가 취약해질 수 있습니다.
3. **복잡성 증가**: 스텁보다 설정이 복잡하고 유지보수가 어려울 수 있습니다.
4. **오용 가능성**: 과도하게 사용하면 테스트가 실제 동작보다 테스트 코드 자체에 집중할 수 있습니다.

## 주요 목 프레임워크

1. **자바**
    - Mockito: 가장 인기 있는 자바 목 프레임워크
    - EasyMock: 초기 목 프레임워크 중 하나
    - PowerMock: 정적 메서드, 파이널 클래스 등 목킹이 어려운 상황을 지원

2. **JavaScript**
    - Jest: 리액트 애플리케이션에 많이 사용되는 테스트 프레임워크
    - Sinon.js: 독립적인 JavaScript 목 라이브러리

3. **Python**
    - unittest.mock: 파이썬 표준 라이브러리
    - pytest-mock: pytest와 함께 사용하는 목 라이브러리

4. **C#**
    - Moq: C#에서 가장 인기 있는 목 프레임워크
    - NSubstitute: 간결한 문법을 강조하는 대안 프레임워크

## 실제 개발 상황에서의 활용

1. **서비스 계층 테스트**
    - 데이터 접근 계층을 목으로 대체하여 서비스 로직만 테스트
    - 외부 서비스 호출을 목으로 시뮬레이션

2. **이벤트 기반 시스템**
    - 이벤트 발행자가 올바른 이벤트를 발행하는지 검증
    - 특정 조건에서 콜백이 호출되는지 검증

3. **복잡한 의존성 체인**
    - 깊은 의존성 체인에서 중간 계층을 목으로 대체
    - 테스트하기 어려운 레거시 코드 테스트

목 객체는 올바르게 사용하면 테스트의 품질을 크게 향상시킬 수 있지만, 과도하게 사용하면 오히려 테스트 유지보수성을 떨어뜨릴 수 있습니다. 목적에 맞게 적절히 활용하는 것이 중요합니다.
