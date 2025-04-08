마틴 파울러의 Null 객체 패턴(Null Object Pattern)은 null 검사를 줄이고 코드를 더 읽기 쉽게 만드는 유용한 디자인 패턴입니다. 몇 가지 실용적인 예제를 통해 이 패턴을 설명해 드리겠습니다.

## 예제 1: 고객 신용 등급 확인

기본적인 null 검사를 사용하는 코드:

```java
public class Customer {
    private CreditRating creditRating;
    
    public CreditRating getCreditRating() {
        return creditRating;
    }
}

// 사용하는 코드
Customer customer = getCustomer();
double discount = 0;
if (customer.getCreditRating() != null) {
    discount = customer.getCreditRating().getDiscountRate();
} else {
    discount = 0;
}
```

Null 객체 패턴을 적용한 코드:

```java
public interface CreditRating {
    double getDiscountRate();
}

public class StandardCreditRating implements CreditRating {
    private final int score;
    
    public StandardCreditRating(int score) {
        this.score = score;
    }
    
    @Override
    public double getDiscountRate() {
        if (score > 800) return 0.10;
        if (score > 700) return 0.05;
        if (score > 500) return 0.02;
        return 0;
    }
}

public class NullCreditRating implements CreditRating {
    @Override
    public double getDiscountRate() {
        return 0; // 신용 등급이 없는 고객은 할인이 없음
    }
}

public class Customer {
    private CreditRating creditRating;
    
    public Customer(CreditRating creditRating) {
        this.creditRating = creditRating != null ? creditRating : new NullCreditRating();
    }
    
    public CreditRating getCreditRating() {
        return creditRating;
    }
}

// 사용하는 코드 - null 검사가 필요 없어짐
Customer customer = getCustomer();
double discount = customer.getCreditRating().getDiscountRate();
```

## 예제 2: 로그 시스템

null 검사를 사용하는 코드:

```java
public class Service {
    private Logger logger;
    
    public void setLogger(Logger logger) {
        this.logger = logger;
    }
    
    public void doOperation() {
        // 작업 전 로깅
        if (logger != null) {
            logger.log("작업 시작");
        }
        
        // 작업 수행
        // ...
        
        // 작업 후 로깅
        if (logger != null) {
            logger.log("작업 완료");
        }
    }
}
```

Null 객체 패턴을 적용한 코드:

```java
public interface Logger {
    void log(String message);
}

public class ConsoleLogger implements Logger {
    @Override
    public void log(String message) {
        System.out.println(message);
    }
}

public class NullLogger implements Logger {
    @Override
    public void log(String message) {
        // 아무 작업도 하지 않음
    }
}

public class Service {
    private Logger logger = new NullLogger(); // 기본값으로 NullLogger 사용
    
    public void setLogger(Logger logger) {
        this.logger = logger != null ? logger : new NullLogger();
    }
    
    public void doOperation() {
        // null 검사 없이 직접 호출
        logger.log("작업 시작");
        
        // 작업 수행
        // ...
        
        logger.log("작업 완료");
    }
}
```

## 예제 3: 결제 프로세서

null 검사를 사용하는 코드:

```java
public class Order {
    private PaymentProcessor paymentProcessor;
    
    public void setPaymentProcessor(PaymentProcessor processor) {
        this.paymentProcessor = processor;
    }
    
    public boolean processPayment(double amount) {
        if (paymentProcessor != null) {
            return paymentProcessor.charge(amount);
        } else {
            // 결제 처리기가 없으면 청구 못함
            return false;
        }
    }
}
```

Null 객체 패턴을 적용한 코드:

```java
public interface PaymentProcessor {
    boolean charge(double amount);
}

public class CreditCardProcessor implements PaymentProcessor {
    private String cardNumber;
    
    public CreditCardProcessor(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    @Override
    public boolean charge(double amount) {
        // 실제 신용카드 결제 처리 로직
        return true; // 성공 가정
    }
}

public class NullPaymentProcessor implements PaymentProcessor {
    @Override
    public boolean charge(double amount) {
        // 결제 불가 - 항상 실패 반환
        return false;
    }
}

public class Order {
    private PaymentProcessor paymentProcessor = new NullPaymentProcessor();
    
    public void setPaymentProcessor(PaymentProcessor processor) {
        this.paymentProcessor = processor != null ? processor : new NullPaymentProcessor();
    }
    
    public boolean processPayment(double amount) {
        return paymentProcessor.charge(amount);
    }
}
```

## 예제 4: 사용자 권한 체크

null 검사를 사용하는 코드:

```java
public class UserService {
    public User findUserById(String id) {
        // 사용자를 데이터베이스에서 찾음
        User user = database.findUser(id);
        return user; // null일 수 있음
    }
}

// 사용하는 코드
User user = userService.findUserById("123");
if (user != null && user.hasPermission("ADMIN")) {
    // 관리자 작업 수행
} else {
    // 권한 없음 또는 사용자 없음
}
```

Null 객체 패턴을 적용한 코드:

```java
public interface User {
    boolean hasPermission(String permission);
    String getName();
    String getEmail();
}

public class RealUser implements User {
    private String id;
    private String name;
    private String email;
    private Set<String> permissions;
    
    // 생성자 및 기타 메서드
    
    @Override
    public boolean hasPermission(String permission) {
        return permissions.contains(permission);
    }
    
    @Override
    public String getName() {
        return name;
    }
    
    @Override
    public String getEmail() {
        return email;
    }
}

public class GuestUser implements User {
    @Override
    public boolean hasPermission(String permission) {
        return false; // 게스트는 권한이 없음
    }
    
    @Override
    public String getName() {
        return "Guest";
    }
    
    @Override
    public String getEmail() {
        return "";
    }
}

public class UserService {
    public User findUserById(String id) {
        // 사용자를 데이터베이스에서 찾음
        User user = database.findUser(id);
        return user != null ? user : new GuestUser();
    }
}

// 사용하는 코드 - null 검사가 필요 없어짐
User user = userService.findUserById("123");
if (user.hasPermission("ADMIN")) {
    // 관리자 작업 수행
} else {
    // 권한 없음
}
```

Null 객체 패턴을 사용하면 다음과 같은 이점이 있습니다:

1. null 검사 코드가 줄어들어 가독성이 향상됩니다.
2. NullPointerException 발생 위험이 줄어듭니다.
3. 객체의 기본 동작을 명시적으로 정의할 수 있습니다.
4. 코드가 더 객체지향적이 됩니다.

단, 모든 상황에 적합한 것은 아니므로 적절한 상황에서 사용하는 것이 중요합니다.
