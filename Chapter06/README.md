# 6장 객체와 자료 구조

### 의존성이란?
- 연관성 있는 것들이 서로 조합하는 것
- 그거 없이는 이게 안되는 것
- 한 객체가 다른 객체를 참조하고 있는 경우

### 인터페이스란?
- 다른 것을 사용하기 위한 도구
- 어떤 일을 하는지 간략하게 보여주고 규격화하기 위한 문법?
- 사용자가 사용할 수 있는 가장 최앞단에 위치한 것

### 추상화란?
- 미사용 클래스로써 확장성이 잠재되어있는 클래스
- abstraction으로 가려놓는 것
- 공통적인 특성을 뽑아내는 것

### 객체란?
- 역할,책임,협력을 하는 것
- 객체지향 프로그래밍에서 역할,책임,행위를 갖고 능동적이고 자율적인 구성원
- 행동을 하는 단위

### 구조체
- 알고리즘에 사용되어 지는 것
- prmitive 타입을 조합하여 새로운 타입을 정의하는 것
- 프로그래밍 세계에서 일어나는 문제를 해결하기 위해 규격화 해놓은 도구

## 자료 추상화

- 구현을 감추기 위해서는 추상화가 필요하다
- 추상 인터페이스를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 의미가 있다

## 자료/객체 비대칭

- 객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다.
- 자료 구조는 자료를 그대로 공개하며 별다른 함수는 제공하지 않는다.

```java
// Procedure Programming
public class Square {
 public Point topLeft;
 public double side;
}

public class Rectangle {
 public Point topLeft;
 public double height;
 public double width;
}

public class Circle {
 public Point center;
 public double radius;
}

public class Geometry {
 public final double PI = 3.141592653589793;

 public double area(Object shape) throws NoSuchShapeException {
  if (shape instanceof Square) {
   Square s = (Square)shape;
   return s.side * s.side;
  }
  else if (shape instanceof Rectangle) {
   Rectangle r = (Rectangle)shape;
   return r.height * r.width;
  }
  else if (shape instanceof Circle) {
   Circle c = (Circle)shape;
   return PI * c.radius * c.radius;
  }
  throw new NoSuchShapeException();
 }
}

// Object Oriented Programming
public class Square implements Shape {
 private Point topLeft;
 private double side;

 public double area() {
  return side*side;
 }
}

public class Rectangle implements Shape {
 private Point topLeft;
 private double height;
 private double width;

 public double area() {
  return height * width;
 }
}

public class Circle implements Shape {
 private Point center;
 private double radius;
 public final double PI = 3.141592653589793;

 public double area() {
  return PI * radius * radius;
 }
}
```

- 객체 지향의 경우
    - 다형(polymorphic) 메서드라서 새 도형을 추가해도 기존 함수(area())를 재사용할 수 있다.
    - 그러나 새로운 기능이 추가되면 도형 클래스 전부를 고쳐야됨
- 절차 지향일 경우
    - geometry에 속한 함수를 모두 고쳐야 한다.
    - geometry에 새로운 기능이 추가되어도 다른 클래스들이 영향을 받지 않는다

- 절차적인 코드는 기존 자료 구조를 변경하지 않으며 새 함수를 추가하기 쉽다.
    - but 새로운 자료 구조를 추가하기 위해서는 모든 함수를 고쳐야 한다.
- 객체 지향 코드는 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다.
    - but 새로운 함수를 추가하려면 모든 클래스를 고쳐야 한다.

## 디미터 법칙

- 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙

## 기차 충돌

- 함수가 반환하는 객체의 함수를 호출하는 현상

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();

Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

- 객체라면 내부 구조를 숨겨야 하므로 디미터 법칙의 위배
- 자료 구조라면 내부 구조를  노출하므로 디미터 법칙이 적용되지 않음
- 자료구조에도 조회 함수와 설정 함수를 정의하라 요구하는 프레임워크와 표준(ex. Bean)이 존재한다.

## 잡종 구조

- 절반은 객체 / 절반은 자료 구조
- 이런 잡종 구조는 새로운 함수와 자료 구조를 추가하기 어렵다.

## 구조체 감추기

- 목적을 드러내는 함수 만들기

## 자료 전달 객체

- 자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스
- DTO (Data Transfer Object)
- Bean 구조

## 활성 레코드

- 공개 변수가 있거나 비공개 변수에 조회/설정 함수가 있는 자료 구조이면서 탐색 함수도 제공한다.
- 데이터베이스 테이블이나 다른 소스에서 자료를 직접 변환한 결과
- 비즈니스 규칙을 추가해 자료 구조를 객체로 취급하는 것은 바람직하지 않다.
- 자료 구조로 취급하고 비즈니스 로직을 가진 객체를 따로 생성

## 결론

- 객체
    - 동작을 공개하고 자료를 숨김
    - 기존 동작을 유지하면서 새 객체를 추가하기 쉬움
    - 기존 객체에 새 동작을 추가하기 어려움
- 자료구조
    - 자료를 노출
    - 새 동작을 추가하기 쉬움
    - 기존 함수에 새 자료 구조를 추가하기 어려움
- 새로운 자료 타입을 추가하는 유연성이 필요할 경우 객체
- 새로운 동작을 추가하는 유연성이 필요할 경우 자료 구조