Java 8에서는 컬렉션을 보다 함수형 프로그래밍 방식으로 다룰 수 있는 여러 Map 관련 기능이 추가되었습니다. 주요 기능들을 소개해 드리겠습니다.

## Map 인터페이스에 추가된 새로운 메소드

### 1. `compute` 메소드
```java
V compute(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction)
```
- 지정된 키에 대한 값을 계산하고 저장합니다.
- 키가 없으면 null로 계산하고, 계산 결과가 null이면 맵에서 제거합니다.

### 2. `computeIfAbsent` 메소드
```java
V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction)
```
- 지정된 키가 없거나 값이 null일 경우에만 계산하여 저장합니다.
- 키가 이미 있으면 기존 값을 반환합니다.

### 3. `computeIfPresent` 메소드
```java
V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction)
```
- 지정된 키가 있고 값이 null이 아닌 경우에만 계산하여 저장합니다.
- 계산 결과가 null이면 매핑을 제거합니다.

### 4. `forEach` 메소드
```java
void forEach(BiConsumer<? super K, ? super V> action)
```
- 맵의 각 항목에 대해 지정된 작업을 수행합니다.

### 5. `getOrDefault` 메소드
```java
V getOrDefault(Object key, V defaultValue)
```
- 지정된 키의 값을 반환하거나, 키가 없으면 기본값을 반환합니다.

### 6. `merge` 메소드
```java
V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction)
```
- 지정된 키가 없거나 값이 null이면 지정된 값으로 설정합니다.
- 키가 있으면 기존 값과 지정된 값에 함수를 적용한 결과를 저장합니다.
- 계산 결과가 null이면 매핑을 제거합니다.

### 7. `putIfAbsent` 메소드
```java
V putIfAbsent(K key, V value)
```
- 지정된 키가 없거나 값이 null일 경우에만 지정된 값을 저장합니다.

### 8. `remove(Object key, Object value)` 메소드
```java
boolean remove(Object key, Object value)
```
- 지정된 키와 값이 모두 일치하는 경우에만 항목을 제거합니다.

### 9. `replace` 메소드들
```java
V replace(K key, V value);
boolean replace(K key, V oldValue, V newValue)
```
- 키가 있을 때만 값을 대체합니다.
- 두 번째 형태는 현재 값이 oldValue와 일치할 때만 대체합니다.

### 10. `replaceAll` 메소드
```java
void replaceAll(BiFunction<? super K, ? super V, ? extends V> function)
```
- 맵의 모든 항목의 값을 함수 적용 결과로 대체합니다.

## 사용 예제

```java
Map<String, Integer> salaries = new HashMap<>();
salaries.put("John", 30000);
salaries.put("Jane", 50000);

// getOrDefault 예제
int michaelSalary = salaries.getOrDefault("Michael", 0);
System.out.println("Michael's salary: " + michaelSalary); // 0

// computeIfPresent 예제 (급여 10% 인상)
salaries.computeIfPresent("John", (k, v) -> (int)(v * 1.1));
System.out.println("John's new salary: " + salaries.get("John")); // 33000

// computeIfAbsent 예제
salaries.computeIfAbsent("Michael", k -> 40000);
System.out.println("Michael's new salary: " + salaries.get("Michael")); // 40000

// merge 예제 (모든 직원에게 보너스 5000 추가)
salaries.forEach((name, salary) -> 
    salaries.merge(name, 5000, (oldSalary, bonus) -> oldSalary + bonus));
```

이러한 새로운 메소드들은 맵을 사용할 때 더 간결하고 함수형 프로그래밍 스타일의 코드를 작성할 수 있게 해주어 Java 8에서 중요한 개선 사항이었습니다.
