깨끗한 클래스란?

역할이란?

협력이란?

책임이란?

soap 서비스란?

# 0908

깨끗한 클래스란?

- 동작하는 클래스와는 반대로 알아보기 쉬운 클래스
- 하는 일이 간단하고 명확한 클래스
- 작은 클래스

(객체지향적인 관점에서)
역할이란?

- 그 클래스가 처리할 수 있는 일
- 클래스의 존재 의미
- 해당 클래스가 수행하는 행위를 공통으로 만들어둔 것

협력이란?

- 서로의 상태를 모른체 자신이 완료한 일을 넘기는 것
- 객체간의 메시지를 주고받는 행위
- 작은 일을 분리하여 큰 결과물을 내는 것

책임이란?

- 그 클래스가 행하고자 하는 바를 이루는 것
- 객체가 수행해야 할 행위를 정해놓은 것
- 클래스가 각각의 역할을 충실히 수행하는 것

## 클래스 체계

표준 자바 관례

- 먼저, 변수목록
  - static constant > public constant > static private variable > private instance variable순
- 그 다음, public function > private function
  - 비공개 함수는 자신을 호출하는 공개 함수 직후에 넣는다.

이처럼, 추상화 단계가 순차적으로 내려간다

### 캡슐화

- 변수와 유틸리티 함수는 가능한 공개하지 않는 편이 낫지만 반드시 숨겨야 한다는 법칙도 없다.
- protected 로 선언해 테스트 코드에 접근을 허용할 수 있음
- 캡슐화를 풀어주는 결정은 제일 마지막으로 고려

### 클래스는 작아야 한다!

- 클래스를 만드는 규칙
  1. 크기
  2. 크기

→ 최대한 작게 만들기 ( '작게' 만드는 것이 기본 규칙 )

- 그렇다면 얼마나 작아야 할까?

  - 함수 → 행 수로 크기를 측정
  - 클래스 → 맡은 책임을 측정

- 클래스의 크기를 줄이는 방법
  - 작명
    - 클래스의 이름은 해당 클래스의 책임을 기술

### 단일 책임 원칙 (Single Responsibility Principle)

→ 클래스나 모듈을 변경할 이유가 하나뿐이어야 한다는 원칙

소프트웨어를 돌아가게 만드는 활동 vs 소프트웨어를 깨끗하게 만드는 활동

복잡성을 다루기 위한 체계적인 정리 필요

몇 개의 큰 클래스보다 작은 클래스 여럿으로 이뤄진 시스템이 더 바람직하다.

작은 클래스

→ 각자 맡은 책임이 하나

→ 변경할 이유가 하나

→ 다른 작은 클래스와 협력해 시스템에 필요한 동작 수행

### 응집도 (Cohesion)

- 클래스는 인스턴스 변수 수가 작아야 한다
- 클래스 메서드에서 변수를 많이 사용할수록, 메서드와 클래스의 응집도는 높아진다.
- 응집도가 높다

  → 클래스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 의미

- '함수를 작게, 매개변수 목록을 짧게' 가지는 전략을 취하면

  → 소수의 메서드만이 사용하는 인스턴스 변수가 많아진다

  → 새로운 클래스로 나누어 주어야한다.

### 응집도를 유지하면 작은 클래스 여럿이 나온다.

## 변경하기 쉬운 클래스

클래스 일부에서만 사용되는 비공개 메서드는 코드를 개선할 잠재적인 여지를 시사한다.

→ 그러나, 실제로 개선에 뛰어드는 계기는 시스템이 변할때이다.

바람직한 구조

→ 새 기능을 수정하거나 기존 기능을 변경할 때 건드릴 코드가 최소인 시스템 구조

→ 새 기능을 추가할 때, 시스템을 확장할 뿐 기존 코드를 변경하지 않아야 됨

### 변경으로부터 격리

- concrete class vs abstract class
- concrete class → 구현된 메서드 / abstract class → 구현되지 않은 메서드
- 인터페이스와 추상 클래스를 사용해 구현에 미치는 영향을 격리시킨다.

### 결합도가 낮다

→ 각 시스템 요소가 다른 요소로부터 그리고 변경으로부터 잘 격리되어 있다.

→ 또한, 시스템 요소가 서로 잘 격리되어 있으면 각 요소를 이해하기도 쉽다.

### DIP(Dependency Inversion Principle)
