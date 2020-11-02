명령행 인수의 구문 분석

소프트웨어는 과학보다는 공예에 가깝다는 사실이다.

깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 뒤에 정리해야 한다는 의미다.

인수 유형은 다양하지만 모두가 유사한 메서드를 제공하므로 클래스 하나로 통합

→ ArgumentMarshaler

매개변수 두개로 Args class 구성

```java
public static void main(String[] args) {
	try {
		Args arg = new Args("l,p#,d*", args); boolean logging = arg.getBoolean('l');
		int port = arg.getInt('p');
		String directory = arg.getString('d'); executeApplication(logging, port, directory);
	} catch (ArgsException e) {
		System.out.printf("Argument error: %s\n", e.errorMessage());
	}
}
```

첫 번째, 형식 또는 스키마를 지정하는 "l,p#,d"

두번째, 명령행 인수 배열 자체

```latex
 지난 수십여 년 동안 쌓아온 경험에서 얻은 교훈이라면, 프로그래밍은 과학보다 공예(craft)에 가깝다는 사실이다.
깨끗한 코드를 짜려면 먼저 지저분한 코드를 짠 뒤에 정리해야 한다는 의미다.
```

1차 코드의 문제점

- 구조화가 되어있지 않아서 새로운 argument type이 생길 때 마다 Args에 추가되어서 관리가 힘들다

새로운 argument를 추가하기 위해서 필요한 코드

- argument type에 해당하는 HashMap을 선택하기 위해 schema 요소의 구문 분석
- 명령행 argument에서 argument를 분석해 진짜 유형으로 변환
- gexXXX method를 구현해 호출자에게 해당하는 argument type형 반환

→ ArgumentMarsharler를 통해 관리

고쳐야 하는 부분

- parse
- get
- set

→ abstract class로 관리하여 기존의 기능들을 분산

문제점

- 여러 argument를 저장할 HashMap이 늘어남

해결책

- 하나의 ArgumentMarsharler HashMap으로 관리

문제점

- 명령행 인수 배열과 인덱스를 같이 넘겨주어야 함

해결책

- Iterator를 통해 관리

exception의 분리
