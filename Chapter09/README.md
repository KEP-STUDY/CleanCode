## 9장 단위테스트

지난 10년간 우리는 눈부신 성장을 하였다.

과거에는 TDD라는 개념이 없었으며, 단위테스트란 그저 단지 돌아간다는 사실만 확인하는 용도에 지나지 않았다.

애자일과 TDD 덕에 단위 테스트를 자동화하는 프로그래머들이 많아졌고 점점 더 늘어나고 있다.

하지만 테스트를 추가하려고 급하게 서두르는 나머지 제대로된 테스트 케이스를 작성하는 사실을 놓쳐버리고 말았다.

## TDD의 법칙 세 가지

1. 실패하는 단위 테스트를 작성할 때 까지 실제 코드를 작성하지 않는다.

2. 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다

3. 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

이상적이지만 이렇게 일하면 매일 수십 개, 매년 수천 개에 달하는 테스트 케이스가 나온다.

사실상 전체를 테스트 하는 테스트 케이스가 나올 것이다.

하지만 실제 코드와 맞먹을 정도로 방대한 테스트 코드는 역으로 심각한 관리 문제를 부르기도 한다.

## 깨끗한 테스트 코드 유지하기

테스트 코드에 실제 코드와 같은 품질 기준을 적용해야할까? 답은 Yes다.

단순히 테스트 코드라고 해서 잘 설계하거나 돌아가기만 된다는 식의 테스트 코드는 좋지않다.

지저분한 테스트 코드 작성 -> 테스트 코드 복잡도 증가 -> 개발자는 테스트 코드를 짜는데 더 많은 시간을 할애 -> 테스트 코드 폐기

위의 과정이 일반적인 테스트 코드를 작성하는 팀에서 발생하는 딜레마이다.

하지만 이러한 테스트 코드가 실패한 이유는 **테스트 코드는 대충 짜도 괜찮다**라는 식의 결정 떄문이다.

테스트 코드는 실제 코드 못지 안헥 중요하다.

실제 코드 못지 않게 깨끗하게 짜야한다.

## 테스트는 유연성 유지보수성 재사용성을 제공한다.

테스트 코드를 깔끔하게 관리함으로써 코드 재사용성을 높이면 결과적으로 테스트 코드 작성에 들이는 시간을 줄일 수 있다.

그러면 깨끗한 테스트 코드를 만들려면 어떻게 해야할까?

단지 세 가지만 기억하라.

**가독성** **가독성** **가독성**

아래의 코드는 FitNess라는 라이브러리에서 가져온 테스트 코드이다.

문제점을 살펴보자.

```java
public void testGetPageHieratchyAsXml() throws Exception {
  crawler.addPage(root, PathParser.parse("PageOne"));
  crawler.addPage(root, PathParser.parse("PageOne.ChildOne"));
  crawler.addPage(root, PathParser.parse("PageTwo"));

  request.setResource("root");
  request.addInput("type", "pages");
  Responder responder = new SerializedPageResponder();
  SimpleResponse response =
    (SimpleResponse) responder.makeResponse(new FitNesseContext(root), request);
  String xml = response.getContent();

  assertEquals("text/xml", response.getContentType());
  assertSubString("<name>PageOne</name>", xml);
  assertSubString("<name>PageTwo</name>", xml);
  assertSubString("<name>ChildOne</name>", xml);
}
```

반복을 줄이면서 코드 재사용성을 높이는 것은 클린 코드의 기본이다.

```java
public void testGetPageHierarchyAsXml() throws Exception {
  makePages("PageOne", "PageOne.ChildOne", "PageTwo");

  submitRequest("root", "type:pages");

  assertResponseIsXML();
  assertResponseContains(
    "<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
}
```

## 도메인 특화 테스트 언어

도메인에 특화된 언어 주로 DSL ( Domain Specific Language )로 테스트 코드를 작성하는 경우도 있다.

아예 언어 자체가 테스트를 위한 언어가 되는 것이다.

## 이중 표준

테스트 API 코드에 적용하는 표준은 실제 코드에 적용하는 표준과는 **다르다**.

간결하고 표현력이 풍부해야하지만 실제 코드만큼 효율적일 필요는 없다.

실제 환경이 아니라 테스트 환경에서만 돌아가는 코드이기 때문이다.

```java
@Test
public void turnOnLoTempAlarmAtThreashold() throws Exception {
  hw.setTemp(WAY_TOO_COLD);
  controller.tic();
  assertTrue(hw.heaterState());
  assertTrue(hw.blowerState());
  assertFalse(hw.coolerState());
  assertFalse(hw.hiTempAlarm());
  assertTrue(hw.loTempAlarm());
}
```

저자가 쓴 코드를 보자.

tic이 뭔지 매우 궁금한데 신경쓸 필요가 없다고 한다.

대신 이 코드는 어쨌든 온도가 급강하 했을 때를 검증하는 테스트 코드이다.

```java
@Test
public void turnOnLoTempAlarmAtThreshold() throws Exception {
  wayTooCold();
  assertEquals("HBchL", hw.getState());
}
```

도메인을 아는 사람 입장에서야 HBchL이 뭔지 알겠지만 모르는 사람 입장에선 여전히 깔끔하지는 않아보인다.

다만 저자는 굉장히 자랑스러워 하고 있는데 역시 클린 코드는 주관적이라는 생각이 드는 대목이었다.

## 테스트 당 개념 하나, assert의 최소화

- 테스트 당 개념 하나를 놓는게 좋다.

- assert는 최소화 하고 코드 중복이나 과도한 방식을 통해 줄여야 한다면 차라리 여러 개를 쓰는게 낫다.

## F.I.R.S.T

깨끗한 테스트 코드는 다음 다섯 가지 규칙을 따른다.

Fast 빠르게 - 테스트는 빨라야한다. 테스트가 느리면 자주 돌릴 엄두를 못내니까.

Independent 독립적으로 - 테스트는 서로 의존해서는 안된다. 한 테스트가 다음 테스트가 실행될 환경을 준비해서는 안된다.

Repetable 반복가능하게 - 테스트는 어떤 환경에서도 반복 가능해야한다.

Self-Validating 자가 검증 하는 - 테스트는 bool 값으로 결과를 내야한다. 반드시 성공 or 실패만이 존재한다.

Timely 적시에 - 테스트는 적시에 작성해야한다. 단위 테스트는 테스트 하려는 실제 코드를 구현하기 직전에 구현한다.

## 결론

이 챕터에서 말하고 싶은 것은 결국 **깨끗한 테스트 코드**다.

테스트 코드는 직속적으로 깨끗하게 관리해주어야한다.

표현력을 높이고 간결하게 정리하자.

테스트 코드가 방치되어 망가지면 실제 코드도 망가진다. 테스트 코드를 꺠끗이 유지하자.
