과도한 DRY(Don't Repeat Yourself) 원칙을 방지하는 방법에 대해 알려드리겠습니다.

## 과도한 DRY 방지 전략

1. **DRY와 WET 사이의 균형 찾기**
    - DRY(Don't Repeat Yourself)와 WET(Write Everything Twice) 사이의 적절한 균형점을 찾으세요
    - 모든 중복을 제거하려 하기보다는 의미 있는 중복과 유해한 중복을 구분하세요

2. **AHA 원칙 적용하기**
    - AHA(Avoid Hasty Abstractions) 원칙을 따르세요
    - 너무 일찍 추상화하지 말고, 패턴이 최소 3번 이상 반복될 때 추상화를 고려하세요

3. **응집도와 결합도 고려하기**
    - 높은 응집도, 낮은 결합도를 목표로 하세요
    - 단순히 중복 제거를 위해 서로 관련 없는 기능들을 하나로 묶지 마세요

4. **YAGNI 원칙 기억하기**
    - YAGNI(You Aren't Gonna Need It) 원칙을 따르세요
    - 현재 필요하지 않은 기능을 위한 과도한 추상화를 피하세요

5. **맥락에 맞는 추상화 수준 선택하기**
    - 모든 상황에 일률적인 추상화 수준을 적용하지 마세요
    - 비즈니스 도메인의 중요도와 변경 가능성에 따라 추상화 수준을 조절하세요

과도한 추상화는 오히려 코드 이해와 유지보수를 어렵게 만들 수 있습니다. 실용적인 접근 방식으로 코드의 명확성과 유연성 사이의 균형을 찾는 것이 중요합니다.

## 과도한 DRY 원칙 적용과 적절한 균형을 찾는 자바 코드 예제

## 1. 과도한 DRY의 예 (문제 코드)

```java
// 너무 추상화된 범용 유틸리티 클래스
public class SuperGenericUtils {
    
    // 너무 많은 매개변수와 조건을 가진 과도하게 일반화된 메서드
    public static <T, R, E extends Exception> R processAnything(
            T input, 
            Function<T, R> processor,
            BiFunction<T, Exception, R> errorHandler,
            Predicate<T> validator,
            Consumer<T> preProcessor,
            Consumer<R> postProcessor,
            Supplier<R> defaultValueSupplier) {
        
        R result = null;
        
        try {
            if (input == null) {
                return defaultValueSupplier.get();
            }
            
            if (validator != null && !validator.test(input)) {
                throw new IllegalArgumentException("Invalid input");
            }
            
            if (preProcessor != null) {
                preProcessor.accept(input);
            }
            
            result = processor.apply(input);
            
            if (postProcessor != null) {
                postProcessor.accept(result);
            }
            
            return result;
        } catch (Exception e) {
            return errorHandler.apply(input, e);
        }
    }
}
```

## 2. 개선된 접근법 (균형 잡힌 코드)

```java
// 사용자 서비스 - 명확한 책임 범위를 가짐
public class UserService {
    
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    // 사용자 등록 기능
    public User registerUser(String username, String email, String password) {
        validateRegistrationInput(username, email, password);
        
        User user = new User(username, email, encryptPassword(password));
        userRepository.save(user);
        
        sendWelcomeEmail(user);
        
        return user;
    }
    
    // 사용자 비밀번호 변경 기능
    public void changePassword(Long userId, String oldPassword, String newPassword) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException("User not found"));
        
        validatePassword(oldPassword, user.getPassword());
        validateNewPassword(newPassword);
        
        user.setPassword(encryptPassword(newPassword));
        userRepository.save(user);
        
        sendPasswordChangedEmail(user);
    }
    
    // 중복되는 내부 로직이지만 명확한 컨텍스트와 적절한 수준의 추상화를 가짐
    private void validateRegistrationInput(String username, String email, String password) {
        if (username == null || username.length() < 3) {
            throw new InvalidInputException("Username must be at least 3 characters");
        }
        
        if (email == null || !email.contains("@")) {
            throw new InvalidInputException("Invalid email format");
        }
        
        validateNewPassword(password);
    }
    
    private void validateNewPassword(String password) {
        if (password == null || password.length() < 8) {
            throw new InvalidInputException("Password must be at least 8 characters");
        }
    }
    
    private String encryptPassword(String password) {
        // 실제 암호화 로직
        return BCrypt.hashpw(password, BCrypt.gensalt());
    }
    
    private void validatePassword(String rawPassword, String encryptedPassword) {
        if (!BCrypt.checkpw(rawPassword, encryptedPassword)) {
            throw new AuthenticationException("Invalid password");
        }
    }
    
    private void sendWelcomeEmail(User user) {
        EmailTemplate template = new EmailTemplate("welcome-template");
        template.setVariable("username", user.getUsername());
        
        emailService.sendEmail(user.getEmail(), "Welcome to our platform", template);
    }
    
    private void sendPasswordChangedEmail(User user) {
        EmailTemplate template = new EmailTemplate("password-changed-template");
        template.setVariable("username", user.getUsername());
        
        emailService.sendEmail(user.getEmail(), "Your password has been changed", template);
    }
}
```

## 3. AHA 원칙 적용 예제 (점진적 추상화)

```java
public class PaymentProcessor {

    // 초기 구현 - 단일 결제 방식
    public void processCreditCardPayment(Order order) {
        // 신용카드 결제 로직
        validateOrder(order);
        String cardNumber = order.getPaymentDetails().getCardNumber();
        double amount = order.getTotalAmount();
        
        try {
            // 결제 처리
            chargeCard(cardNumber, amount);
            order.setStatus(OrderStatus.PAID);
            notifyCustomer(order);
        } catch (PaymentException e) {
            handlePaymentError(order, e);
        }
    }
    
    // 두 번째 결제 방식 추가 - 여전히 중복이 있지만 패턴 확인 중
    public void processPayPalPayment(Order order) {
        // 페이팔 결제 로직
        validateOrder(order);
        String paypalAccount = order.getPaymentDetails().getPaypalAccount();
        double amount = order.getTotalAmount();
        
        try {
            // 결제 처리
            chargePayPal(paypalAccount, amount);
            order.setStatus(OrderStatus.PAID);
            notifyCustomer(order);
        } catch (PaymentException e) {
            handlePaymentError(order, e);
        }
    }
    
    // 세 번째 결제 방식 추가 - 이제 패턴이 명확해져 리팩토링 시작
    public void processBankTransferPayment(Order order) {
        // 은행 송금 결제 로직
        validateOrder(order);
        String accountNumber = order.getPaymentDetails().getAccountNumber();
        double amount = order.getTotalAmount();
        
        try {
            // 결제 처리
            transferToBank(accountNumber, amount);
            order.setStatus(OrderStatus.PAID);
            notifyCustomer(order);
        } catch (PaymentException e) {
            handlePaymentError(order, e);
        }
    }
    
    // 패턴이 확인된 후 적절한 추상화 적용
    public void processPayment(Order order, PaymentMethod paymentMethod) {
        validateOrder(order);
        double amount = order.getTotalAmount();
        
        try {
            // 결제 처리 - 결제 방식에 따라 다른 처리
            switch (paymentMethod) {
                case CREDIT_CARD:
                    String cardNumber = order.getPaymentDetails().getCardNumber();
                    chargeCard(cardNumber, amount);
                    break;
                case PAYPAL:
                    String paypalAccount = order.getPaymentDetails().getPaypalAccount();
                    chargePayPal(paypalAccount, amount);
                    break;
                case BANK_TRANSFER:
                    String accountNumber = order.getPaymentDetails().getAccountNumber();
                    transferToBank(accountNumber, amount);
                    break;
                default:
                    throw new UnsupportedPaymentMethodException("Payment method not supported");
            }
            
            order.setStatus(OrderStatus.PAID);
            notifyCustomer(order);
        } catch (PaymentException e) {
            handlePaymentError(order, e);
        }
    }
    
    // 더 개선된 전략 패턴 적용 - 새로운 결제 방식 추가가 더 쉬워짐
    // (YAGNI 원칙에 따라 4번째 결제 방식이 필요할 때 도입)
    public void processPaymentWithStrategy(Order order, PaymentStrategy paymentStrategy) {
        validateOrder(order);
        
        try {
            paymentStrategy.processPayment(order);
            order.setStatus(OrderStatus.PAID);
            notifyCustomer(order);
        } catch (PaymentException e) {
            handlePaymentError(order, e);
        }
    }
    
    private void validateOrder(Order order) {
        if (order == null) {
            throw new IllegalArgumentException("Order cannot be null");
        }
        if (order.getTotalAmount() <= 0) {
            throw new IllegalArgumentException("Order amount must be greater than zero");
        }
    }
    
    private void notifyCustomer(Order order) {
        // 고객에게 결제 완료 알림
    }
    
    private void handlePaymentError(Order order, PaymentException e) {
        order.setStatus(OrderStatus.PAYMENT_FAILED);
        order.setErrorMessage(e.getMessage());
        // 추가 오류 처리 로직
    }
    
    // 그 외 결제 처리 메서드들...
    private void chargeCard(String cardNumber, double amount) { /* 구현 */ }
    private void chargePayPal(String paypalAccount, double amount) { /* 구현 */ }
    private void transferToBank(String accountNumber, double amount) { /* 구현 */ }
}

// 전략 패턴을 위한 인터페이스
interface PaymentStrategy {
    void processPayment(Order order) throws PaymentException;
}

// 구체적인 결제 전략 구현체
class CreditCardPaymentStrategy implements PaymentStrategy {
    @Override
    public void processPayment(Order order) throws PaymentException {
        String cardNumber = order.getPaymentDetails().getCardNumber();
        double amount = order.getTotalAmount();
        // 신용카드 결제 처리
    }
}

class PayPalPaymentStrategy implements PaymentStrategy {
    @Override
    public void processPayment(Order order) throws PaymentException {
        String paypalAccount = order.getPaymentDetails().getPaypalAccount();
        double amount = order.getTotalAmount();
        // 페이팔 결제 처리
    }
}

class BankTransferPaymentStrategy implements PaymentStrategy {
    @Override
    public void processPayment(Order order) throws PaymentException {
        String accountNumber = order.getPaymentDetails().getAccountNumber();
        double amount = order.getTotalAmount();
        // 은행 송금 처리
    }
}
```

## 4. 의미 있는 중복 예제 (유사하지만 다른 도메인)

```java
public class CustomerValidator {
    public void validateCustomer(Customer customer) {
        if (customer == null) {
            throw new IllegalArgumentException("Customer cannot be null");
        }
        
        if (customer.getName() == null || customer.getName().isEmpty()) {
            throw new ValidationException("Customer name is required");
        }
        
        if (customer.getEmail() == null || !customer.getEmail().contains("@")) {
            throw new ValidationException("Valid customer email is required");
        }
        
        // 고객 특정 유효성 검사
        if (customer.getCustomerType() == CustomerType.BUSINESS && customer.getTaxId() == null) {
            throw new ValidationException("Business customers must provide a tax ID");
        }
    }
}

public class SupplierValidator {
    public void validateSupplier(Supplier supplier) {
        if (supplier == null) {
            throw new IllegalArgumentException("Supplier cannot be null");
        }
        
        if (supplier.getName() == null || supplier.getName().isEmpty()) {
            throw new ValidationException("Supplier name is required");
        }
        
        if (supplier.getEmail() == null || !supplier.getEmail().contains("@")) {
            throw new ValidationException("Valid supplier email is required");
        }
        
        // 공급업체 특정 유효성 검사
        if (supplier.getProducts() == null || supplier.getProducts().isEmpty()) {
            throw new ValidationException("Supplier must provide at least one product");
        }
    }
}

// 과도한 추상화 대신 유지 - 두 도메인이 별개로 발전할 가능성이 높음
// 코드 중복이 있지만, 도메인 개념의 독립성을 유지하는 것이 더 중요함
```

이 예제들은 각각 과도한 DRY 적용, 적절한 추상화 수준, 점진적 리팩토링, 그리고 의미 있는 중복의 허용을 보여줍니다. 실제 비즈니스 상황과 코드의 변경 가능성을 고려하여 균형 잡힌 접근법을 선택하는 것이 중요합니다.
