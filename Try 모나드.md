## Java의 예외 처리 한계

```java
// 기존 방식 - 예외 처리가 번거로움
try {
    String result = riskyOperation1()
        .flatMap(this::riskyOperation2)  // 컴파일 에러! try-catch 필요
        .map(String::toUpperCase);
} catch (Exception e) {
    // 어느 단계에서 에러가 났는지 알기 어려움
}
```

## 커스텀 Try 모나드 구현

```java
import java.util.function.Function;
import java.util.function.Supplier;

public abstract class Try<T> {
    
    public abstract boolean isSuccess();
    public abstract boolean isFailure();
    public abstract T get() throws Exception;
    public abstract Exception getException();
    
    // unit/return
    public static <T> Try<T> of(Supplier<T> supplier) {
        try {
            return new Success<>(supplier.get());
        } catch (Exception e) {
            return new Failure<>(e);
        }
    }
    
    public static <T> Try<T> success(T value) {
        return new Success<>(value);
    }
    
    public static <T> Try<T> failure(Exception exception) {
        return new Failure<>(exception);
    }
    
    // bind (flatMap)
    public abstract <U> Try<U> flatMap(Function<T, Try<U>> mapper);
    
    // map
    public abstract <U> Try<U> map(Function<T, U> mapper);
    
    // recover - 에러 시 대체 값 제공
    public abstract Try<T> recover(Function<Exception, T> recoveryFunction);
    
    // recoverWith - 에러 시 다른 Try 제공
    public abstract Try<T> recoverWith(Function<Exception, Try<T>> recoveryFunction);
    
    // getOrElse - 값 추출 또는 기본값 반환
    public abstract T getOrElse(T defaultValue);
    
    // Optional로 변환
    public abstract Optional<T> toOptional();
    
    // 내부 구현 클래스들
    private static class Success<T> extends Try<T> {
        private final T value;
        
        Success(T value) {
            this.value = value;
        }
        
        @Override
        public boolean isSuccess() { return true; }
        
        @Override
        public boolean isFailure() { return false; }
        
        @Override
        public T get() { return value; }
        
        @Override
        public Exception getException() {
            throw new UnsupportedOperationException("Success has no exception");
        }
        
        @Override
        public <U> Try<U> flatMap(Function<T, Try<U>> mapper) {
            try {
                return mapper.apply(value);
            } catch (Exception e) {
                return new Failure<>(e);
            }
        }
        
        @Override
        public <U> Try<U> map(Function<T, U> mapper) {
            try {
                return new Success<>(mapper.apply(value));
            } catch (Exception e) {
                return new Failure<>(e);
            }
        }
        
        @Override
        public Try<T> recover(Function<Exception, T> recoveryFunction) {
            return this;
        }
        
        @Override
        public Try<T> recoverWith(Function<Exception, Try<T>> recoveryFunction) {
            return this;
        }
        
        @Override
        public T getOrElse(T defaultValue) {
            return value;
        }
        
        @Override
        public Optional<T> toOptional() {
            return Optional.of(value);
        }
    }
    
    private static class Failure<T> extends Try<T> {
        private final Exception exception;
        
        Failure(Exception exception) {
            this.exception = exception;
        }
        
        @Override
        public boolean isSuccess() { return false; }
        
        @Override
        public boolean isFailure() { return true; }
        
        @Override
        public T get() throws Exception { throw exception; }
        
        @Override
        public Exception getException() { return exception; }
        
        @Override
        public <U> Try<U> flatMap(Function<T, Try<U>> mapper) {
            return new Failure<>(exception);
        }
        
        @Override
        public <U> Try<U> map(Function<T, U> mapper) {
            return new Failure<>(exception);
        }
        
        @Override
        public Try<T> recover(Function<Exception, T> recoveryFunction) {
            try {
                return new Success<>(recoveryFunction.apply(exception));
            } catch (Exception e) {
                return new Failure<>(e);
            }
        }
        
        @Override
        public Try<T> recoverWith(Function<Exception, Try<T>> recoveryFunction) {
            try {
                return recoveryFunction.apply(exception);
            } catch (Exception e) {
                return new Failure<>(e);
            }
        }
        
        @Override
        public T getOrElse(T defaultValue) {
            return defaultValue;
        }
        
        @Override
        public Optional<T> toOptional() {
            return Optional.empty();
        }
    }
}
```

## Try 모나드 사용 예제

```java
public class TryExample {
    
    public static void main(String[] args) {
        // 기본 사용법
        Try<Integer> result = Try.of(() -> riskyOperation("10"))
            .map(Integer::parseInt)
            .map(x -> x * 2)
            .flatMap(x -> divide(x, 2));
        
        if (result.isSuccess()) {
            System.out.println("Result: " + result.getOrElse(0));  // 10
        } else {
            System.out.println("Error: " + result.getException().getMessage());
        }
        
        // 체이닝 with recovery
        String finalResult = Try.of(() -> parseAndProcess("invalid"))
            .recover(ex -> "Parse failed: " + ex.getMessage())
            .getOrElse("Unknown error");
        
        System.out.println(finalResult);
        
        // 다중 연산 체이닝
        Try<String> pipeline = Try.of(() -> readFile("config.txt"))
            .flatMap(content -> Try.of(() -> parseJson(content)))
            .map(json -> json.get("value"))
            .flatMap(value -> Try.of(() -> processValue(value)))
            .recoverWith(ex -> {
                if (ex instanceof FileNotFoundException) {
                    return Try.success("Default config");
                } else {
                    return Try.failure(ex);
                }
            });
        
        pipeline.toOptional().ifPresent(System.out::println);
    }
    
    static String riskyOperation(String input) throws Exception {
        if (input == null) throw new IllegalArgumentException("Null input");
        return input;
    }
    
    static Try<Integer> divide(int a, int b) {
        return Try.of(() -> {
            if (b == 0) throw new ArithmeticException("Division by zero");
            return a / b;
        });
    }
    
    static String parseAndProcess(String input) throws Exception {
        int value = Integer.parseInt(input);  // NumberFormatException 가능
        return String.valueOf(value * 2);
    }
    
    static String readFile(String path) throws FileNotFoundException {
        // 파일 읽기 로직
        throw new FileNotFoundException("File not found: " + path);
    }
    
    static JsonObject parseJson(String content) throws JsonParseException {
        // JSON 파싱 로직
        throw new JsonParseException("Invalid JSON");
    }
    
    static String processValue(Object value) throws Exception {
        // 값 처리 로직
        return value.toString().toUpperCase();
    }
}
```

## Vavr 라이브러리 사용하기

직접 구현하는 대신 **Vavr** (구 Javaslang) 라이브러리를 사용할 수도 있습니다:

```xml
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.10.4</version>
</dependency>
```

```java
import io.vavr.control.Try;
import io.vavr.control.Either;

public class VavrExample {
    public static void main(String[] args) {
        // Vavr의 Try 사용
        Try<Integer> result = Try.of(() -> riskyOperation())
            .map(x -> x * 2)
            .flatMap(x -> Try.of(() -> divide(x, 2)))
            .recover(Exception.class, ex -> 0);
        
        result.forEach(System.out::println);
        
        // Either와 함께 사용
        Either<String, Integer> either = parseInteger("123")
            .map(x -> x * 2)
            .flatMap(x -> divideEither(x, 2));
        
        either.fold(
            error -> System.out.println("Error: " + error),
            value -> System.out.println("Success: " + value)
        );
    }
    
    static Either<String, Integer> parseInteger(String s) {
        try {
            return Either.right(Integer.parseInt(s));
        } catch (NumberFormatException e) {
            return Either.left("Not a valid integer: " + s);
        }
    }
    
    static Either<String, Integer> divideEither(int a, int b) {
        return b == 0 
            ? Either.left("Division by zero") 
            : Either.right(a / b);
    }
}
```

## 결론

- **Java 내장**: Optional (null 처리), Stream (collection 처리)
- **예외 처리**: Try 모나드는 직접 구현하거나 Vavr 같은 라이브러리 사용
- **상태 관리**: State 모나드도 직접 구현 필요
- **함수형 스타일**: Optional과 Stream을 활용해서 어느 정도는 가능하지만, 완전한 모나드 생태계를 원한다면 Vavr 권장

Try 모나드는 예외 처리를 함수형 스타일로 할 수 있게 해주는 강력한 도구입니다. 예외가 발생할 수 있는 연산들을 안전하게 체이닝할 수 있어서 매우 유용합니다.
