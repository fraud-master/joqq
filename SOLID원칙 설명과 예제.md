# SOLID 원칙 설명과 자바 예제 코드

SOLID는 객체 지향 프로그래밍에서 코드의 유지보수성과 확장성을 높이기 위한 5가지 설계 원칙입니다. 각 원칙에 대해 설명하고 자바 예제 코드를 제공하겠습니다.

## 1. 단일 책임 원칙 (Single Responsibility Principle, SRP)

### 설명
클래스는 단 하나의 책임만 가져야 합니다. 즉, 클래스를 변경해야 하는 이유는 오직 하나뿐이어야 합니다.

### 나쁜 예제
```java
public class Employee {
    private String name;
    private String email;
    private int salary;
    
    // 생성자, getter, setter 생략
    
    public void calculateTax() {
        // 세금 계산 로직
        System.out.println("세금 계산: " + (salary * 0.2));
    }
    
    public void saveToDatabase() {
        // 데이터베이스 저장 로직
        System.out.println(name + " 직원 정보를 데이터베이스에 저장");
    }
    
    public void generateReport() {
        // 보고서 생성 로직
        System.out.println(name + "의 보고서 생성");
    }
}
```

### 좋은 예제
```java
// 직원 정보만 담당
public class Employee {
    private String name;
    private String email;
    private int salary;
    
    // 생성자, getter, setter 생략
}

// 세금 계산만 담당
public class TaxCalculator {
    public void calculateTax(Employee employee) {
        // 세금 계산 로직
        System.out.println("세금 계산: " + (employee.getSalary() * 0.2));
    }
}

// 데이터베이스 저장만 담당
public class EmployeeRepository {
    public void save(Employee employee) {
        // 데이터베이스 저장 로직
        System.out.println(employee.getName() + " 직원 정보를 데이터베이스에 저장");
    }
}

// 보고서 생성만 담당
public class ReportGenerator {
    public void generateReport(Employee employee) {
        // 보고서 생성 로직
        System.out.println(employee.getName() + "의 보고서 생성");
    }
}
```

## 2. 개방-폐쇄 원칙 (Open-Closed Principle, OCP)

### 설명
소프트웨어 엔티티(클래스, 모듈, 함수 등)는 확장에는 열려 있어야 하고, 수정에는 닫혀 있어야 합니다.

### 나쁜 예제
```java
public class Rectangle {
    private double width;
    private double height;
    
    // 생성자, getter, setter 생략
}

public class Circle {
    private double radius;
    
    // 생성자, getter, setter 생략
}

public class AreaCalculator {
    public double calculateArea(Object shape) {
        if (shape instanceof Rectangle) {
            Rectangle rectangle = (Rectangle) shape;
            return rectangle.getWidth() * rectangle.getHeight();
        } else if (shape instanceof Circle) {
            Circle circle = (Circle) shape;
            return Math.PI * Math.pow(circle.getRadius(), 2);
        }
        // 새로운 도형이 추가되면 이 메서드를 수정해야 함
        return 0;
    }
}
```

### 좋은 예제
```java
public interface Shape {
    double calculateArea();
}

public class Rectangle implements Shape {
    private double width;
    private double height;
    
    // 생성자, getter, setter 생략
    
    @Override
    public double calculateArea() {
        return width * height;
    }
}

public class Circle implements Shape {
    private double radius;
    
    // 생성자, getter, setter 생략
    
    @Override
    public double calculateArea() {
        return Math.PI * Math.pow(radius, 2);
    }
}

public class Triangle implements Shape {
    private double base;
    private double height;
    
    // 생성자, getter, setter 생략
    
    @Override
    public double calculateArea() {
        return 0.5 * base * height;
    }
}

public class AreaCalculator {
    public double calculateArea(Shape shape) {
        // 새로운 도형이 추가되어도 이 메서드는 수정할 필요가 없음
        return shape.calculateArea();
    }
}
```

## 3. 리스코프 치환 원칙 (Liskov Substitution Principle, LSP)

### 설명
상위 타입의 객체를 하위 타입의 객체로 치환해도 프로그램의 정확성이 유지되어야 합니다.

### 나쁜 예제
```java
public class Rectangle {
    protected int width;
    protected int height;
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    public int getArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width; // 정사각형은 가로와 세로가 같아야 함
    }
    
    @Override
    public void setHeight(int height) {
        this.width = height; // 정사각형은 가로와 세로가 같아야 함
        this.height = height;
    }
}

// 클라이언트 코드
public void testRectangle(Rectangle rectangle) {
    rectangle.setWidth(5);
    rectangle.setHeight(4);
    // 직사각형이라면 area는 20이어야 함
    // 하지만 Square가 전달되면 area는 16이 됨
    assert rectangle.getArea() == 20; // Square의 경우 실패
}
```

### 좋은 예제
```java
public interface Shape {
    int getArea();
}

public class Rectangle implements Shape {
    private int width;
    private int height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    @Override
    public int getArea() {
        return width * height;
    }
}

public class Square implements Shape {
    private int side;
    
    public Square(int side) {
        this.side = side;
    }
    
    public void setSide(int side) {
        this.side = side;
    }
    
    @Override
    public int getArea() {
        return side * side;
    }
}

// 클라이언트 코드
public void testShapeArea(Shape shape) {
    // 공통 인터페이스 메서드만 사용
    int area = shape.getArea();
    // 각 구현체에 맞는 적절한 검증
}
```

## 4. 인터페이스 분리 원칙 (Interface Segregation Principle, ISP)

### 설명
클라이언트는 사용하지 않는 메서드에 의존하지 않아야 합니다. 큰 인터페이스보다 특정 클라이언트에 맞는 여러 개의 작은 인터페이스가 더 좋습니다.

### 나쁜 예제
```java
public interface Worker {
    void work();
    void eat();
    void sleep();
}

// 로봇은 eat과 sleep이 필요 없지만 구현해야 함
public class Robot implements Worker {
    @Override
    public void work() {
        System.out.println("로봇이 일합니다.");
    }
    
    @Override
    public void eat() {
        // 로봇은 먹지 않음 (필요 없는 구현)
        throw new UnsupportedOperationException("로봇은 먹지 않습니다.");
    }
    
    @Override
    public void sleep() {
        // 로봇은 자지 않음 (필요 없는 구현)
        throw new UnsupportedOperationException("로봇은 자지 않습니다.");
    }
}

// 일반 직원은 모든 메서드가 필요함
public class HumanWorker implements Worker {
    @Override
    public void work() {
        System.out.println("사람이 일합니다.");
    }
    
    @Override
    public void eat() {
        System.out.println("사람이 식사합니다.");
    }
    
    @Override
    public void sleep() {
        System.out.println("사람이 휴식합니다.");
    }
}
```

### 좋은 예제
```java
public interface Workable {
    void work();
}

public interface Eatable {
    void eat();
}

public interface Sleepable {
    void sleep();
}

// 로봇은 필요한 인터페이스만 구현
public class Robot implements Workable {
    @Override
    public void work() {
        System.out.println("로봇이 일합니다.");
    }
}

// 인간 직원은 모든 인터페이스 구현
public class HumanWorker implements Workable, Eatable, Sleepable {
    @Override
    public void work() {
        System.out.println("사람이 일합니다.");
    }
    
    @Override
    public void eat() {
        System.out.println("사람이 식사합니다.");
    }
    
    @Override
    public void sleep() {
        System.out.println("사람이 휴식합니다.");
    }
}

// 클라이언트 코드
public class WorkManager {
    public void manage(Workable worker) {
        worker.work(); // 작업 가능한 모든 객체를 사용할 수 있음
    }
}
```

## 5. 의존성 역전 원칙 (Dependency Inversion Principle, DIP)

### 설명
고수준 모듈은 저수준 모듈에 의존해서는 안됩니다. 둘 다 추상화에 의존해야 합니다. 추상화는 세부 사항에 의존해서는 안되며, 세부 사항이 추상화에 의존해야 합니다.

### 나쁜 예제
```java
// 저수준 모듈
public class MySQLDatabase {
    public void insert(String data) {
        System.out.println("MySQL에 데이터 삽입: " + data);
    }
}

// 고수준 모듈이 저수준 모듈에 직접 의존
public class UserService {
    private MySQLDatabase database;
    
    public UserService() {
        this.database = new MySQLDatabase(); // 직접 생성하여 의존
    }
    
    public void addUser(String userData) {
        database.insert(userData);
    }
}
```

### 좋은 예제
```java
// 추상화 (인터페이스)
public interface Database {
    void insert(String data);
}

// 저수준 모듈은 추상화를 구현
public class MySQLDatabase implements Database {
    @Override
    public void insert(String data) {
        System.out.println("MySQL에 데이터 삽입: " + data);
    }
}

public class OracleDatabase implements Database {
    @Override
    public void insert(String data) {
        System.out.println("Oracle에 데이터 삽입: " + data);
    }
}

// 고수준 모듈은 추상화에 의존
public class UserService {
    private Database database; // 인터페이스에 의존
    
    // 의존성 주입을 통해 구체적인 구현체를 받음
    public UserService(Database database) {
        this.database = database;
    }
    
    public void addUser(String userData) {
        database.insert(userData);
    }
}

// 클라이언트 코드
public class Main {
    public static void main(String[] args) {
        Database mysqlDb = new MySQLDatabase();
        UserService userService = new UserService(mysqlDb);
        userService.addUser("사용자1");
        
        // 다른 데이터베이스로 쉽게 전환 가능
        Database oracleDb = new OracleDatabase();
        UserService anotherService = new UserService(oracleDb);
        anotherService.addUser("사용자2");
    }
}
```

## SOLID 원칙 종합 예제

아래는 SOLID 원칙을 모두 적용한 간단한 급여 시스템 예제입니다:

```java
// 1. SRP: 각 클래스는 하나의 책임만 가짐
// 직원 인터페이스
interface Employee {
    double calculateSalary();
    String getEmployeeInfo();
}

// 2. OCP: 확장에는 열려있고 수정에는 닫혀있음
// 정규직 직원
class FullTimeEmployee implements Employee {
    private String name;
    private double monthlySalary;
    
    public FullTimeEmployee(String name, double monthlySalary) {
        this.name = name;
        this.monthlySalary = monthlySalary;
    }
    
    @Override
    public double calculateSalary() {
        return monthlySalary;
    }
    
    @Override
    public String getEmployeeInfo() {
        return "정규직: " + name;
    }
}

// 계약직 직원
class ContractEmployee implements Employee {
    private String name;
    private double hourlyRate;
    private int hoursWorked;
    
    public ContractEmployee(String name, double hourlyRate, int hoursWorked) {
        this.name = name;
        this.hourlyRate = hourlyRate;
        this.hoursWorked = hoursWorked;
    }
    
    @Override
    public double calculateSalary() {
        return hourlyRate * hoursWorked;
    }
    
    @Override
    public String getEmployeeInfo() {
        return "계약직: " + name;
    }
}

// 3. LSP: 하위 타입은 상위 타입을 대체할 수 있어야 함
// 4. ISP: 필요한 메서드만 포함하는 인터페이스

// 세금 계산 인터페이스
interface TaxCalculator {
    double calculateTax(double salary);
}

// 국내 세금 계산기
class DomesticTaxCalculator implements TaxCalculator {
    @Override
    public double calculateTax(double salary) {
        return salary * 0.2; // 20% 세금
    }
}

// 해외 세금 계산기
class ForeignTaxCalculator implements TaxCalculator {
    @Override
    public double calculateTax(double salary) {
        return salary * 0.15; // 15% 세금
    }
}

// 5. DIP: 추상화에 의존
class SalaryProcessor {
    private TaxCalculator taxCalculator;
    
    // 의존성 주입
    public SalaryProcessor(TaxCalculator taxCalculator) {
        this.taxCalculator = taxCalculator;
    }
    
    public double calculateNetSalary(Employee employee) {
        double grossSalary = employee.calculateSalary();
        double tax = taxCalculator.calculateTax(grossSalary);
        return grossSalary - tax;
    }
}

// 메인 클래스
public class SalarySystem {
    public static void main(String[] args) {
        // 직원 생성
        Employee fullTime = new FullTimeEmployee("홍길동", 5000000);
        Employee contract = new ContractEmployee("김철수", 20000, 160);
        
        // 세금 계산기 생성
        TaxCalculator domesticTax = new DomesticTaxCalculator();
        
        // 급여 처리기 생성
        SalaryProcessor processor = new SalaryProcessor(domesticTax);
        
        // 급여 계산
        System.out.println(fullTime.getEmployeeInfo() + "의 순 급여: " + processor.calculateNetSalary(fullTime));
        System.out.println(contract.getEmployeeInfo() + "의 순 급여: " + processor.calculateNetSalary(contract));
        
        // 세금 계산기 변경 (쉬운 확장)
        TaxCalculator foreignTax = new ForeignTaxCalculator();
        SalaryProcessor foreignProcessor = new SalaryProcessor(foreignTax);
        
        System.out.println(fullTime.getEmployeeInfo() + "의 해외 근무 순 급여: " + foreignProcessor.calculateNetSalary(fullTime));
    }
}
```

이 예제는 다음과 같이 SOLID 원칙을 적용했습니다:

1. **단일 책임 원칙(SRP)**: 각 클래스는 하나의 책임만 가집니다. Employee는 급여 정보, TaxCalculator는 세금 계산, SalaryProcessor는 급여 처리만 담당합니다.
2. **개방-폐쇄 원칙(OCP)**: 새로운 직원 유형이나 세금 계산 방식을 추가해도 기존 코드를 수정할 필요가 없습니다.
3. **리스코프 치환 원칙(LSP)**: FullTimeEmployee와 ContractEmployee는 Employee 인터페이스를 완전히 구현하며, 어디서든 Employee 타입으로 사용할 수 있습니다.
4. **인터페이스 분리 원칙(ISP)**: 각 인터페이스는 필요한 메서드만 포함합니다.
5. **의존성 역전 원칙(DIP)**: SalaryProcessor는 구체적인 TaxCalculator 구현체가 아닌 인터페이스에 의존합니다.
