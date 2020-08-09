## 제 5장 형식 맞추기

## 토론 해보기

- 코드를 볼 때 난해하다고 느낀 경험이 있는지?

- 하나의 파일의 줄 수가 500줄 이상 넘어가면 어떤 느낌이 드는가?

- 프로젝트를 열어서 내부 모듈을 살펴 볼 때 이 함수 저 함수 오가며 소스 파일을 위-아래로 뒤져본 경험이 있는지?

- 클래스 내에서만 사용되는 private 함수의 적절한 위치는 어디에 두는게 좋은가?

- 클래스의 프로퍼티들은 어디에 위치시키는 것이 좋을까?

## 이 장에 들어가기에 앞서

```
밥 아저씨의 바램

뚜껑을 열었을 때 독자들이 코드가 깔끔하고, 일관적이며, 꼼꼼하다고 감탄하면 좋겠다.

질서 정연하다고 탄복하면 좋겠다.

모듈을 읽으며 눈이 휘둥그래 놀랐으면 좋겠다.

전문가가 짰다는 인상을 심어주면 좋겠다.

그 대신 술취한 애런 한 무리가 술게임 하자고 하듯이 어수선하다면, 

사람들은 그 프로젝트의 다른 측면도 무성의하게 처리했을 것이라고 생각할 것이다.

프로그래머라면 형식을 깔끔하게 맞춰 코드를 짜야한다.

코드 형식을 맞추기 위해 간단한 규칙을 정하고 그 규칙을 확실히 따라야 한다.

팀으로 일한다면 팀이 합의해 규칙을 정하고 모두가 그 규칙을 따라야 한다.

필요하다면 규칙을 자동으로 생성하는 도구를 사용하라
```

## 형식을 맞추는 목적

- 코드 형식은 **매우 Fuxxing 중요하다**

- 코드 형식은 의사소통이요 의사소통은 개발자의 의무다

- 돌아가는 코드에 만족하지 마라

- 오늘의 코드는 내일 바뀐다

- 처음 잡아놓은 구현 스타일과 가독성 수준은 이후의 유지보수와 확장성에 많은 영향을 미친다

## 적절한 행 길이

![5_2fig_martin](https://user-images.githubusercontent.com/43809168/89733856-520f3f00-da93-11ea-97bc-d576a4bfc530.jpg)

- 파일의 세로 길이는 짧으면 짧을 수록 좋다

- JUnit, FitNesse, Time and Money는 최대 500줄 넘어가는 파일이 없음

- 일반적으로는 큰 파일보다 작은 파일이 이해하기 쉽다

## 신문 기사 처럼 작성하라 (말이 쉽지)

- 최상단에 의도를 드러내라

- 이름은 간단하면서도 설명이 가능하게 짓는다 (어렵다니까?)

- 코드는 위에서 아래로 읽힌다는 사실을 기억하라

- 의도는 상단에, 구체적인 내용은 하단에 위치시켜라

## 개념은 빈 행으로 분리하라

```kotlin
package com.kakaoenterprise.kakaowork.commando.cms.controller

import com.kakaoenterprise.kakaowork.commando.cms.enum.PropertyType
import com.kakaoenterprise.kakaowork.commando.cms.model.api.request.RequiredIds
import com.kakaoenterprise.kakaowork.commando.cms.model.api.request.TypeRequest
```

- 일반적인 자바 코드에서 package와 import는 공백으로 구분된다.

- 빈 행을 넣는 기준은 **새로운 개념**이 시작 될 때

## 세로 밀집도

```kotlin
class Test {
    /**
    * 저번 장에서
    */
    val className: String

    /**
    * 주석 쓰지 말랬는데
    */
    val properties: List<Property>

    /**
    * 주석 쓰네 뒤질라고
    */
    fun addProperty() {
        ...
    }
}
```

```kotlin
class Test {
    val className: String
    val properties: List<Property>

    fun addProperty() {
        ...
    }
}
```

- 서로 밀접한 코드는 가까이에 놓자

- 주석은 자제

## 수직 거리

- 서로 밀접한 개념은 세로로 가까이 두는게 맞다

```java
private static void readPreferences() {
    InputStream is = null;
    try {
        is = new FileInputStream(getPrefrenceFile());
        ...
        ...
        ...
    } catch (IOException e) {
        try {
            ...
            ...

        } catch() {
            ...
        }
    }
}
```

- 위 코드는 JUnit에 사용된 코드인데 최상단에 지역변수가 선언되었다

- 변수간에는 세로로 거리를 두지 않는다

- 인스턴스 변수의 위치는 논쟁이 분분하다 C++의 경우 인스턴스 변수를 클래스 마지막에 선언함

- 어느쪽이든 상관 없으나 잘 알려진 위치에 인스턴스 변수를 모아야함

### 종속 함수

- A 함수가 B 함수를 호출한다면 A 함수를 더 먼저 배치

- A 함수와 B 함수는 서로 가까운 위치에 선언

### 개념적 유사성

![jeGj8sI](https://user-images.githubusercontent.com/43809168/89734172-75d38480-da95-11ea-9cb7-d210017fd1dd.jpg)

- 친화도가 높은 코드는 서로를 끌어당긴다

- 비슷한 동작을 수행하는 코드들도 가까이에 배치시키면 가독성이 높아진다

```java
public class Assertions {
	/**
	 * <em>Assert</em> that the supplied {@code condition} is {@code true}.
	 * <p>Fails with the supplied failure {@code message}.
	 */
	public static void assertTrue(boolean condition, String message) {
		AssertTrue.assertTrue(condition, message);
	}

	/**
	 * <em>Assert</em> that the boolean condition supplied by {@code booleanSupplier} is {@code true}.
	 * <p>If necessary, the failure message will be retrieved lazily from the supplied {@code messageSupplier}.
	 */
	public static void assertTrue(BooleanSupplier booleanSupplier, Supplier<String> messageSupplier) {
		AssertTrue.assertTrue(booleanSupplier, messageSupplier);
	}

	// --- assertFalse ---------------------------------------------------------

	/**
	 * <em>Assert</em> that the supplied {@code condition} is {@code false}.
	 */
	public static void assertFalse(boolean condition) {
		AssertFalse.assertFalse(condition);
	}

	/**
	 * <em>Assert</em> that the supplied {@code condition} is {@code false}.
	 * <p>Fails with the supplied failure {@code message}.
	 */
	public static void assertFalse(boolean condition, String message) {
		AssertFalse.assertFalse(condition, message);
	}

}
```

실제 JUnit의 Assert 클래스의 메소드들을 보면 비슷한 기능을 하는 메소드들 끼리 붙어있다.

꼭 서로가 서로를 호출하는 함수가 아니더라도 가까이 배치하는 경우의 예가 딱 위의 코드이다.

## 세로 순서

- 함수 호출 종속성은 아래 방향으로 한다

- 호출되는 함수가 호출하는 함수보다 아래에 위치시킨다.

```kotlin
fun caller() {
    callee()
}

private fun callee() {
    println("Ex Girlfriend : Fu** out")
}
```

## 가로 형식 맞추기

- 저자가 조사한 바에 따르면 일반적인 프로그램에서 평균적인 가로행의 길이는 대략 20-60자 사이

- 더 길어도 상관없다 요즘 젊은 것들은 모니터도 크고 눈도 좋아서 200자도 쌉가능

- 그러나 저자의 추천은 대략 120자

## 가로 공백과 밀집도

```kotlin
fun test(name: String) {
    var count = 0
    call(count, name)
}
```

- 함수 이름과 이어지는 괄호 사이에는 공백을 두지 않았는데 이는 함수와 인수가 밀접하기 때문

- 할당 연산자 강조를 위해 앞 뒤에 공백을 둠

- call 인자가 쉼표 뒤에 공백이 있음 쉼표를 강조해 인수가 별개라는 의미를 갖게 함

## 팀 규칙

- 좋은 소프트웨어 시스템은 읽기 쉬운 문서로 이루어진다

- 스타일은 일관적이며 매끄러워야 한다

- 한 소스 파일에서 본 형식이 다른 소스 파일에도 쓰이리라는 신뢰를 줘야함

- 온갖 스타일을 뒤 섞어 소스 코드를 필요 이상으로 복잡하게 만들지 말 것

## 밥 아저씨의 형식 규칙

```java
public class CodeAnalyzer implements JavaFileAnalysis { 
	private int lineCount;
	private int maxLineWidth;
	private int widestLineNumber;
	private LineWidthHistogram lineWidthHistogram; 
	private int totalChars;
	
	public CodeAnalyzer() {
		lineWidthHistogram = new LineWidthHistogram();
	}
	
	public static List<File> findJavaFiles(File parentDirectory) { 
		List<File> files = new ArrayList<File>(); 
		findJavaFiles(parentDirectory, files);
		return files;
	}
	
	private static void findJavaFiles(File parentDirectory, List<File> files) {
		for (File file : parentDirectory.listFiles()) {
			if (file.getName().endsWith(".java")) 
				files.add(file);
			else if (file.isDirectory()) 
				findJavaFiles(file, files);
		} 
	}
	
	public void analyzeFile(File javaFile) throws Exception { 
		BufferedReader br = new BufferedReader(new FileReader(javaFile)); 
		String line;
		while ((line = br.readLine()) != null)
			measureLine(line); 
	}
	
	private void measureLine(String line) { 
		lineCount++;
		int lineSize = line.length();
		totalChars += lineSize; 
		lineWidthHistogram.addLine(lineSize, lineCount);
		recordWidestLine(lineSize);
	}
	
	private void recordWidestLine(int lineSize) { 
		if (lineSize > maxLineWidth) {
			maxLineWidth = lineSize;
			widestLineNumber = lineCount; 
		}
	}

	public int getLineCount() { 
		return lineCount;
	}

	public int getMaxLineWidth() { 
		return maxLineWidth;
	}

	public int getWidestLineNumber() { 
		return widestLineNumber;
	}

	public LineWidthHistogram getLineWidthHistogram() {
		return lineWidthHistogram;
	}
	
	public double getMeanLineWidth() { 
		return (double)totalChars/lineCount;
	}

	public int getMedianLineWidth() {
		Integer[] sortedWidths = getSortedWidths(); 
		int cumulativeLineCount = 0;
		for (int width : sortedWidths) {
			cumulativeLineCount += lineCountForWidth(width); 
			if (cumulativeLineCount > lineCount/2)
				return width;
		}
		throw new Error("Cannot get here"); 
	}
	
	private int lineCountForWidth(int width) {
		return lineWidthHistogram.getLinesforWidth(width).size();
	}
	
	private Integer[] getSortedWidths() {
		Set<Integer> widths = lineWidthHistogram.getWidths(); 
		Integer[] sortedWidths = (widths.toArray(new Integer[0])); 
		Arrays.sort(sortedWidths);
		return sortedWidths;
	} 
}
```

마지막으로 저자의 코드를 보며 감탄 & 비판 해보도록 합시다

## 끗