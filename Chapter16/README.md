# 16장 SerialDate 리팩토링

이번 장에서 우리는 JCommon 라이브러리의 date 패키지의 **SerialDate** 클래스를 리팩토링 해보려고 한다.

저자는 무려 10줄에 걸쳐서 **SerialDate**를 개발한 개발자인 데이비드 길버트를 샤라웃하며 까려는(?) 의도가 아님을 몇번이고 언급한다. (라곤 하지만 뒤에 가보면 신랄하게 코드를 까고 있다...)

우리는 다른 사람이 나의 코드에 대해서 해주는 조언을 감사히 여기고 반겨야할 행동이라는 것을 알아둘 필요가 있다.

의사도, 조종사들도 법류가들도 하는 행위이다. 개발자 또한 당연히 다른 사람의 코드를 보고 비판하고 더 좋은 코드로 만들어가기 위한 과정이 필요하다.

물론 이것이 가능하려면 팀원간의 강한 신뢰가 있어야 충돌할 수 있다. 그리고 다른 사람의 코드를 내 시간을 내서 리뷰해주는 것은 감사한 것이며 이는 일종의 헌신이다.

그말은 카카오 개발자가 가져야할 신충헌(신뢰 - 충돌 - 헌신)을 가장 잘 보여주는 예시는 코드 리뷰라고 생각한다.

내 코드를 다른 사람에게 보여주는 것을 절대 부끄러워하지 말자. 성장의 기회로 삼자.

자바에는 이미 `java.util.Date`와 `java.util.Calendar` (JAVA 8 이상에서는 LocalDate,LocalDateTime, ZonnedDateTime도 존재)가 있는데 그는 왜 굳이 `SerialDate`를 만들었을까?

과거에는 `SerialDate`가 시간 기반 클래스가 아닌 순수 날짜 클래스여서 과거에는 종종 사용을 했다고 한다.

이게 뭐하는 라이브러리인지 모르는 사람을 위해 아래의 테스트 코드를 보면 한방에 이해할 수 있다.

```java
public void testMondayNearest22Jan1970() {
    SerialDate jan22Y1970 = SerialDate.createInstance(22, MonthConstants.JANUARY, 1970);
    SerialDate mondayNearest = SerialDate.getNearestDayOfWeek(SerialDate.MONDAY, jan22Y1970);
    assertEquals(19, mondayNearest.getDayOfMonth());
}
```

테스트 코드의 또 다른 장점은 테스트 코드를 통해 해당 코드의 설명을 대신할 수 있다는 장점이 있다.

역시나 마찬가지로 나 또한 이 라이브러리를 이용하기 위해 테스트 코드를 보고 **아 이렇게 쓰는 거구나~~**하고 한방에 이해할 수 있었다.

일,월,년을 기준으로 SerialDate를 생성해서 날짜를 표현할 수도 있고, 해당 날짜의 가장 가까운 월요일을 찾아주는 기능에 대한 테스트도 존재한다.

결국 시간이 아닌 순수하게 날짜와 관련된 여러 간편한 기능들을 제공해주는 라이브러리임을 알 수 있다.

그러나 현재는 자바 기본 라이브러리의 날짜 라이브러리가 너무 성능이 좋고 지원되는 기능도 많아서 이제 JCommon의 SerialDate를 쓰는 경우는 **전혀** 없다.

그래서 다음과 같은 PR로 조롱을 당하기도 한다. [PR6](https://github.com/jfree/jcommon/pull/6)

PR6의 내용을 요약하면 **대충 JCommon 망한 것 같으니까 내 놀이터로 삼겠음 ㅎㅎ 개꿀**의 내용이다.

[SerialDate Code](https://github.com/jfree/jcommon/blob/master/src/main/java/org/jfree/date/SerialDate.java)

## 첫 번째, 돌려보자

[SerialDateTest Code](https://github.com/jfree/jcommon/blob/master/src/test/java/org/jfree/date/SerialDateTest.java)

- Test Code의 부재, SerialDate의 테스트 커버리지는 약 50%

- 부록에 존재하는 Bob 아저씨의 테스트 코드를 보면 테스트 코드를 작성하고 반복과 중복을 줄여서 리팩토링하는 것을 살펴볼 수 있다.

- 그는 SerialDate의 테스트 커버리지를 높이고 통과하는 것을 확인하여 SerialDate 라이브러리의 유효성을 검증했다.

## 두 번째, 고쳐보자

이제 밥아저씨의 테스트 코드와 함께 SerialDate를 리팩토링 해보자.

## 주석에 관하여

가장 먼저 해야할 일은 SerialDate 클래스의 주석 제거이다.

라이선스 정보, 저작권, 작성자, 변경 이력등이 보인다.

라이선스 정보나 저작권, 작성자는 보존하더라도 변경 이력은 버전관리 도구를 사용하는 현대에 맞지않는다.

```
 * Changes (from 11-Oct-2001)
 * --------------------------
 * 11-Oct-2001 : Re-organised the class and moved it to new package
 *               com.jrefinery.date (DG);
 * 05-Nov-2001 : Added a getDescription() method, and eliminated NotableDate
 *               class (DG);
 * 12-Nov-2001 : IBD requires setDescription() method, now that NotableDate
 *               class is gone (DG);  Changed getPreviousDayOfWeek(),
 *               getFollowingDayOfWeek() and getNearestDayOfWeek() to correct
 *               bugs (DG);
 * 05-Dec-2001 : Fixed bug in SpreadsheetDate class (DG);
 * 29-May-2002 : Moved the month constants into a separate interface
 *               (MonthConstants) (DG);
 * 27-Aug-2002 : Fixed bug in addMonths() method, thanks to N???levka Petr (DG);
 * 03-Oct-2002 : Fixed errors reported by Checkstyle (DG);
 * 13-Mar-2003 : Implemented Serializable (DG);
 * 29-May-2003 : Fixed bug in addMonths method (DG);
 * 04-Sep-2003 : Implemented Comparable.  Updated the isInRange javadocs (DG);
 * 05-Jan-2005 : Fixed bug in addYears() method (1096282) (DG);
 *
 */
```

## import에 관하여

37행부터 57행은 과감히 삭제해준다.

```java
import java.text.DateFormatSymbols;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.GregorianCalendar;
```

임포트문 또한 text와 util은 \*로 줄여주자.

## Javadoc에 관하여

```
/**
 *  An abstract class that defines our requirements for manipulating dates,
 *  without tying down a particular implementation.
 *  <P>
 *  Requirement 1 : match at least what Excel does for dates;
 *  Requirement 2 : the date represented by the class is immutable;
 *  <P>
 *  Why not just use java.util.Date?  We will, when it makes sense.  At times,
 *  java.util.Date can be *too* precise - it represents an instant in time,
 *  accurate to 1/1000th of a second (with the date itself depending on the
 *  time-zone).  Sometimes we just want to represent a particular day (e.g. 21
 *  January 2015) without concerning ourselves about the time of day, or the
 *  time-zone, or anything else.  That's what we've defined SerialDate for.
 *  <P>
 *  You can call getInstance() to get a concrete subclass of SerialDate,
 *  without worrying about the exact implementation.
 *
 * @author David Gilbert
 */
```

그 아래의 Javadoc 주석은 HTML 태그를 사용한다.

한 소스 코드에 여러 언어를 사용해서 밥 아저씨는 매우 빡치셨다.

이 주석 안에서는 영어, 자바, Javadoc, HTML이라는 네 가지 언어를 사용하고 있다.

그래서 이 주석 전체를 `<pre>` 태그로 감싸는 것을 밥 아저씨는 추천하고 있다.

## 이름에 관하여

```java
public abstract class SerialDate implements Comparable,
                                            Serializable,
                                            MonthConstants {
```

여기서 우리는 순수한 의문을 가질 수 있는데, 왜 이 라이브러리는 `Serial`이라는 이름이 붙은걸까?

그 이유는 Serial Number (일련 번호)를 이용해 클래스를 구현했기 때문이다.

여기서는 1899년 12월 30일을 기준으로 경과한 날짜 수를 사용한다.

여기서 밥 아저씨는 두 가지의 포인트에서 빡이 쳤다.

### 1. `일련번호`라는 용어는 정확하지 못하다

`Serial Number`는 사실 날짜보다는 제품 식별 번호로 더 적합하다.

그래서 좀 더 서술적인 용어인 `Ordinal`이 더 낫다고 얘기하고 있다.

### 2. SerialDate는 구현 클래스가 아니다

사실 이게 조금 더 중요한 이유인데, 애초에 SerialDate는 추상클래스다.

추상 클래스가 구현을 암시할 이유가 없다.

밥 아저씨는 차라리 `Date`로 짓는게 더 나았을거라고 조언한다.

근데 그러기엔 자바에 이미 `Date`라는 클래스가 너무 많다.

그래서 이 역시 적합하진 않지만 그나마 최선을 고르자면 `DayDate` 정도로 결정했다.

그래서 그는 이 `SerialDate` 추상 클래스를 `DayDate`로 부르기로 결정했다.

## 구현에 관하여

```java
public abstract class SerialDate implements Comparable,
                                            Serializable,
                                            MonthConstants {
```

`SeiralDate` 아니, `DayDate`는 Comparable, Serializable, MonthConstants를 구현하고 있다.

Comparable, Serializable는 그렇다치고 MonthConstants는 뭘까?

당장 이름만 봐도 뭔가 상수같아 보이는 인터페이스로 보인다.

맞다. MonthConstants는 사실 상수들로만 이루어진 인터페이스이다.

옛날 자바 프로그래머들이 많이 쓰는 기교인데 (라고 말하며 은근히 데이비드 길버트를 돌려까는 밥 아저씨...) 바람직한 방법은 아니다.

나 역시도 그렇게 생각하는데 상수의 집합이라면 구현보다는 `enum`으로 두는게 더 낫다.

## 변수 위의 주석

```java
    /** For serialization. */
    private static final long serialVersionUID = -293716040467423637L;
```

당연한 의미를 갖는 주석은 필요 없다.

제거한다.

```java
    /** The serial number for 1 January 1900. */
    public static final int SERIAL_LOWER_BOUND = 2;

    /** The serial number for 31 December 9999. */
    public static final int SERIAL_UPPER_BOUND = 2958465;
```

여기의 주석은 사실 최초 시작 시간과 최후 날짜를 의미한다.

이걸 주석으로 표현할 거라면 차라리 아래가 낫다.

```java
    public static final int SERIAL_LOWER_BOUND = 2; // 1/1/1900

    public static final int SERIAL_UPPER_BOUND = 2958465; // 12/31/9999
```

반드시 주석이 나쁜 것은 아니다.

하지만 최대한 간결하게 표현하는 것이 낫다.

근데 애당초 Serial.. 아니 DayDate 클래스는 추상 클래스여서 이런 정보를 가질 이유도 없다.

그리고 이 변수를 사용하는 유일한 클래스가 `SpreadsheetDate`라서 위 두 정보 또한 `SpreadsheetDate`로 옮겼다.

## 팩토리 패턴의 사용

```java
    /** The lowest year value supported by this date format. */
    public static final int MINIMUM_YEAR_SUPPORTED = 1900;

    /** The highest year value supported by this date format. */
    public static final int MAXIMUM_YEAR_SUPPORTED = 9999;
```

다음에 등장하는 변수들 또한 `SpreadsheetDate`로 옮길 옞어이었는데, `RelativeDayOfWeekRule`이라는 클래스도 이 두 변수를 사용한다는 사실이 드러났다.

이를 리팩토링 하려고 보니 DayDate 인스턴스를 생성하는 메소드들이 존재한다는 사실 또한 드러났다.

일반적으로는 기반 클래스(부모 클래스)가 파생 클래스(자식 클래스)를 몰라야 정상이다.

그런데 현재 구조는 부모 클래스가 자식 클래스의 정보에 의존하는 구조적인 문제가 있었다.

이를 위해 저자는 DayDateFactory라는 추상 팩토리 클래스를 만들고 최대 날짜와 최소 날짜와 같은 구현에 대한 정보를 가지게 했다.

그리고 SpreadsheetDateFactory 클래스가 DayDateFactory 클래스를 상속받게 하였다.

## 상수는 enum으로

```java
    /** Useful constant for Monday. Equivalent to java.util.Calendar.MONDAY. */
    public static final int MONDAY = Calendar.MONDAY;

    /**
     * Useful constant for Tuesday. Equivalent to java.util.Calendar.TUESDAY.
     */
    public static final int TUESDAY = Calendar.TUESDAY;

    /**
     * Useful constant for Wednesday. Equivalent to
     * java.util.Calendar.WEDNESDAY.
     */
    public static final int WEDNESDAY = Calendar.WEDNESDAY;

    /**
     * Useful constant for Thrusday. Equivalent to java.util.Calendar.THURSDAY.
     */
    public static final int THURSDAY = Calendar.THURSDAY;

    /** Useful constant for Friday. Equivalent to java.util.Calendar.FRIDAY. */
    public static final int FRIDAY = Calendar.FRIDAY;

    /**
     * Useful constant for Saturday. Equivalent to java.util.Calendar.SATURDAY.
     */
    public static final int SATURDAY = Calendar.SATURDAY;

    /** Useful constant for Sunday. Equivalent to java.util.Calendar.SUNDAY. */
    public static final int SUNDAY = Calendar.SUNDAY;
```

이러한 상수 값들은 역시 enum으로 리팩토링 해준다.

## 배열 설명 주석 제거

```java
    /** The number of days in each month in non leap years. */
    static final int[] LAST_DAY_OF_MONTH =
        {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};

    /** The number of days in a (non-leap) year up to the end of each month. */
    static final int[] AGGREGATE_DAYS_TO_END_OF_MONTH =
        {0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334, 365};

    /** The number of days in a year up to the end of the preceding month. */
    static final int[] AGGREGATE_DAYS_TO_END_OF_PRECEDING_MONTH =
        {0, 0, 31, 59, 90, 120, 151, 181, 212, 243, 273, 304, 334, 365};

    /** The number of days in a leap year up to the end of each month. */
    static final int[] LEAP_YEAR_AGGREGATE_DAYS_TO_END_OF_MONTH =
        {0, 31, 60, 91, 121, 152, 182, 213, 244, 274, 305, 335, 366};

    /**
     * The number of days in a leap year up to the end of the preceding month.
     */
    static final int[]
        LEAP_YEAR_AGGREGATE_DAYS_TO_END_OF_PRECEDING_MONTH =
            {0, 0, 31, 60, 91, 121, 152, 182, 213, 244, 274, 305, 335, 366};
```

각 배열을 설명하는 주석이 묘하게 비슷하다는 것을 알 수 있다.

변수 이름만으로도 이미 의미가 명확하므로 주석을 제거해준다.

그리고 충격적이게도 `AGGREGATE_DAYS_TO_END_OF_MONTH` 변수와 `LEAP_YEAR_AGGREGATE_DAYS_TO_END_OF_MONTH` 변수는 JCommon 프레임워크 어디에서도 사용하지 않는다고 한다. 이런 변수들은 지워주자. (요즘은 IDE 성능이 좋아서 이런 변수들은 IDE UI에서 비활성화 처리되어서 한눈에 알기 쉬워졌다.)

```java
    /** A useful constant for referring to the first week in a month. */
    public static final int FIRST_WEEK_IN_MONTH = 1;

    /** A useful constant for referring to the second week in a month. */
    public static final int SECOND_WEEK_IN_MONTH = 2;

    /** A useful constant for referring to the third week in a month. */
    public static final int THIRD_WEEK_IN_MONTH = 3;

    /** A useful constant for referring to the fourth week in a month. */
    public static final int FOURTH_WEEK_IN_MONTH = 4;

    /** A useful constant for referring to the last week in a month. */
    public static final int LAST_WEEK_IN_MONTH = 0;
```

이런 상수들 또한 enum으로 변경해주자.

## 이름의 모호성

```java
    /** Useful range constant. */
    public static final int INCLUDE_NONE = 0;

    /** Useful range constant. */
    public static final int INCLUDE_FIRST = 1;

    /** Useful range constant. */
    public static final int INCLUDE_SECOND = 2;

    /** Useful range constant. */
    public static final int INCLUDE_BOTH = 3;
```

이 변수들은 범위의 끝 날짜를 범위에 포함할지에 대한 여부를 표현하는 상수들이다.

수학적으로 개구간(open interval), 반개구간(half interval), 폐구간(closed interval)이라는 개념이다.

그래서 이들의 이름은 CLOSED, CLOSED_LEFT, CLOSED_RIGHT, OPEN이라 보는게 더 적합하다.

```java

    /**
     * Useful constant for specifying a day of the week relative to a fixed
     * date.
     */
    public static final int PRECEDING = -1;

    /**
     * Useful constant for specifying a day of the week relative to a fixed
     * date.
     */
    public static final int NEAREST = 0;

    /**
     * Useful constant for specifying a day of the week relative to a fixed
     * date.
     */
    public static final int FOLLOWING = 1;
```

이 상수들은 주어진 날짜를 기준으로 특정 요일을 계산할 때 사용한다.

각각 직전 요일, 다음 요일, 가장 가까운 요일을 의미한다.

저자는 `WeekDayRange`라는 이름으로 enum 타입을 정의하고

각 값은 LAST, NEXT, NEAREST로 정의했다.

## 기본 생성자

```java
    /**
     * Default constructor.
     */
    protected SerialDate() {
    }
```

기본 생성자 역시 컴파일러가 만들어주므로 제거해주자.

## final의 제거

밥 아저씨는 인수와 변수의 선언에 `final` 키워드를 모두 제거했다.

실질적인 가치는 없으면서 코드만 복잡하게 만든다고 판단했기 때문이다.

그는 자신이 final 키워드가 정신 싸납다고 느끼는 이유가 자신의 테스트 코드가 final 키워드로 잡아낼 오류를 미리 잡았기 떄문이라고 생각해서 일지도 모른다고 했다. (갑자기 자기자랑을?)

## if문 리팩토링

### Before

```java
    public static int stringToWeekdayCode(String s) {

        final String[] shortWeekdayNames
            = DATE_FORMAT_SYMBOLS.getShortWeekdays();
        final String[] weekDayNames = DATE_FORMAT_SYMBOLS.getWeekdays();

        int result = -1;
        s = s.trim();
        for (int i = 0; i < weekDayNames.length; i++) {
            if (s.equals(shortWeekdayNames[i])) {
                result = i;
                break;
            }
            if (s.equals(weekDayNames[i])) {
                result = i;
                break;
            }
        }
        return result;

    }
```

### After

```java
    public static int stringToWeekdayCode(String s) {

        final String[] shortWeekdayNames
            = DATE_FORMAT_SYMBOLS.getShortWeekdays();
        final String[] weekDayNames = DATE_FORMAT_SYMBOLS.getWeekdays();

        int result = -1;
        s = s.trim();
        for (int i = 0; i < weekDayNames.length; i++) {
            if (s.equals(shortWeekdayNames[i]) || s.equals(weekDayNames[i])) {
                result = i;
                break;
            }
        }
        return result;

    }
```

그리고 그는 Day enum 클래스를 만들었는데 위 메소드가 굳이 DayDate에 속할 필요가 없다고 판단해서 Day enum 클래스로 옮겼다. (객체지향!)

## 메소드가 메소드를 부르는 경우

```java
    /**
     * Returns an array of month names.
     *
     * @return an array of month names.
     */
    public static String[] getMonths() {

        return getMonths(false);

    }

    /**
     * Returns an array of month names.
     *
     * @param shortened  a flag indicating that shortened month names should
     *                   be returned.
     *
     * @return an array of month names.
     */
    public static String[] getMonths(final boolean shortened) {

        if (shortened) {
            return DATE_FORMAT_SYMBOLS.getShortMonths();
        }
        else {
            return DATE_FORMAT_SYMBOLS.getMonths();
        }

    }
```

getMonth는 결국 getMonth를 리턴하는 아주 띠용한 구조이다.

하나로 합치고 이름을 조금 더 서술적으로 변경했다.

```java
    public static String[] getMonthNames() {

        return dateFormatSymbols.getMonths();
    }
```

dateFormatSymbols는 위에서 리팩토링한 enum 클래스이다.

```java
    /**
     * Returns a string representing the supplied month.
     * <P>
     * The string returned is the long form of the month name taken from the
     * default locale.
     *
     * @param month  the month.
     *
     * @return a string representing the supplied month.
     */
    public static String monthCodeToString(final int month) {

        return monthCodeToString(month, false);

    }

    /**
     * Returns a string representing the supplied month.
     * <P>
     * The string returned is the long or short form of the month name taken
     * from the default locale.
     *
     * @param month  the month.
     * @param shortened  if <code>true</code> return the abbreviation of the
     *                   month.
     *
     * @return a string representing the supplied month.
     */
    public static String monthCodeToString(final int month,
                                           final boolean shortened) {

        // check arguments...
        if (!isValidMonthCode(month)) {
            throw new IllegalArgumentException(
                "SerialDate.monthCodeToString: month outside valid range.");
        }

        final String[] months;

        if (shortened) {
            months = DATE_FORMAT_SYMBOLS.getShortMonths();
        }
        else {
            months = DATE_FORMAT_SYMBOLS.getMonths();
        }

        return months[month - 1];

    }
```

위와 비슷한 패턴의 메서드가 또 보인다.

일반적으로 메서드 인수로 플래그를 넣는 것은 바람직하지 않다.

특히 출력 형식을 선택하는 플래그는 가급적 피하는 편이 좋다.

차라리 다음과 같이 함수로 쪼개고 enum으로 넘겼다.

```java
public String toString() {
    return dateFormatSymbols.getMonths()[index - 1];
}

public String toShortString() {
    return dateFormatSymbols.getShortMonths()[index - 1];
}
```

## 서술적 표현의 중요성

```java
    /**
     * Determines whether or not the specified year is a leap year.
     *
     * @param yyyy  the year (in the range 1900 to 9999).
     *
     * @return <code>true</code> if the specified year is a leap year.
     */
    public static boolean isLeapYear(final int yyyy) {

        if ((yyyy % 4) != 0) {
            return false;
        }
        else if ((yyyy % 400) == 0) {
            return true;
        }
        else if ((yyyy % 100) == 0) {
            return false;
        }
        else {
            return true;
        }

    }
```

위 코드는 if, else 문이 반복되면서 다소 코드를 이해하기가 어렵다.

저자는 **서술적인 표현**을 강조하며 가독성을 높였다.

```java
public static boolean isLeapYear(int year) {
    boolean fourth = year % 4 == 0;
    boolean hundredth = year % 100 == 0;
    boolean fourHundredth = year % 400 == 0;
    return fourth && ( !hundredth || fourHundredth);
}
```

이러한 서술적 표현의 중요성은 코드 가독성을 높이고 읽는 사람으로 하여금 코드의 이해도를 높이는 효과가 있다.

## addDays 메서드

```java
    public static SerialDate addDays(final int days, final SerialDate base) {

        final int serialDayNumber = base.toSerial() + days;
        return SerialDate.createInstance(serialDayNumber);

    }
```

이 메서드는 온갖 DayDate 변수를 사용하므로 static이어서는 안된다.

인스턴스 메서드로 변경하고, 호출하는 toSerial 메서드를 toOrinal로 변경하였다.

```java
public DayDate addDyas(int days) {
    return DayDateFctory.makeDate(toOrdinal() + days);
}
```

코드가 훨씬 깔끔해지고 명확해진 것을 알 수 있다.

```java
    public static SerialDate addMonths(final int months,
                                       final SerialDate base) {

        final int yy = (12 * base.getYYYY() + base.getMonth() + months - 1)
                       / 12;
        final int mm = (12 * base.getYYYY() + base.getMonth() + months - 1)
                       % 12 + 1;
        final int dd = Math.min(
            base.getDayOfMonth(), SerialDate.lastDayOfMonth(mm, yy)
        );
        return SerialDate.createInstance(dd, mm, yy);

    }
```

addMonth 또한 마찬가지다.

인스턴스 메서드로 수정하고 알고리즘이 복잡하므로 임시 변수 설명을 사용해 좀 더 읽기 쉽게 수정하였다.

```java
public DayDate addMonths(int months) {
    int thisMonthAsOrdinal = 12 * getYear() + getMonth().index -1;
    int resultMonthAsOrdinal = thisMonthAsOrdinal + months;
    int resultYear = resultMonthAsOrdinal / 12;
    Month resultMonth = Month.make(resultMonthAsOrdinal % 12 + 1);

    int lastDayofResultMonth = lastDayofResultMonth(resultMonth, resultYear);
    int resultDay = Math.min(getDayOfMonth(), lastDayOfResultMonth);
    return DayDateFactory.makeDate(resultDay, resultMonth, resultYear);
}
```

getYYY라는 이름도 getYear로 고치고 조금 더 직관적으로 수정했다.

```java
    public static SerialDate addYears(final int years, final SerialDate base) {

        final int baseY = base.getYYYY();
        final int baseM = base.getMonth();
        final int baseD = base.getDayOfMonth();

        final int targetY = baseY + years;
        final int targetD = Math.min(
            baseD, SerialDate.lastDayOfMonth(baseM, targetY)
        );

        return SerialDate.createInstance(targetD, baseM, targetY);

    }
```

addYears 또한 수정할 수 있다.

```java
public DayDate plusYears(int years){
    int resultYear = getYear() + years;
    int lastDayOfMonthInResultYear = lastDayOfMonth(getMonth(), resultYear);
    int resultDay = Math.min(getDayOfMonth(), lastDayOfMonthInResultYear);
    return DayDateFactory.makeDate(resultDay, getMonth(), resultYear);
}
```

위 메서드에서 date.addDyas(5)라는 표현이 date 객체를 변경하지 않고 새 DayDate를 반환한다는 사실이 과연 명확하게 드러날 수 있을까?

아니면 date 객체에 5일을 더한다고 오해할 소지는 없을까? 그래서 저자는 addDays보다는 `plusDays`, `plusMonth`로 이름을 변경하였다.

코드를 읽는 사람은 addDays가 date 객체의 상태를 변경한다고 생각한다.

## 중복 인스턴스 메서드 제거

```java
    public static SerialDate getPreviousDayOfWeek(final int targetWeekday,
                                                  final SerialDate base) {

        // check arguments...
        if (!SerialDate.isValidWeekdayCode(targetWeekday)) {
            throw new IllegalArgumentException(
                "Invalid day-of-the-week code."
            );
        }

        // find the date...
        final int adjust;
        final int baseDOW = base.getDayOfWeek();
        if (baseDOW > targetWeekday) {
            adjust = Math.min(0, targetWeekday - baseDOW);
        }
        else {
            adjust = -7 + Math.max(0, targetWeekday - baseDOW);
        }

        return SerialDate.addDays(adjust, base);

    }
```

getPreviousDayOfWeek는 올바로 동작은 하지만 로직이 어쩐지 복잡하다.

임시 변수 설명을 사용해 읽기 쉽게 변경하고 인스턴스 메서드로 변경했다.

또한 Math와 관련된 중복된 인스턴스 메서드를 제거했다.

```java
public DayDate getPreviousDayOfWeek(Day tagrgetDayOfWeek) {
    int offsetToTarget = targetDayOfWeek.index - getDayOfWeek().index;
    if(offsetToTarget >= 0)
        offsetToTarget -= 7;
    return plusDays(offsetToTarget);
}
```

임시 변수 설명을 통해 우리는 알고리즘을 조금 더 명확하게 표현해낼 수 있다는 사실을 알게 되었다.

```java
    public static SerialDate getFollowingDayOfWeek(final int targetWeekday,
                                                   final SerialDate base) {

        // check arguments...
        if (!SerialDate.isValidWeekdayCode(targetWeekday)) {
            throw new IllegalArgumentException(
                "Invalid day-of-the-week code."
            );
        }

        // find the date...
        final int adjust;
        final int baseDOW = base.getDayOfWeek();
        if (baseDOW > targetWeekday) {
            adjust = 7 + Math.min(0, targetWeekday - baseDOW);
        }
        else {
            adjust = Math.max(0, targetWeekday - baseDOW);
        }

        return SerialDate.addDays(adjust, base);
    }
```

이 메소드도 마찬가지로 위와 같이 리팩토링한다.

```java
public DayDate getFloowingDayOfWeek(Day tagrgetDayOfWeek) {
    int offsetToTarget = targetDayOfWeek.index - getDayOfWeek().index;
    if(offsetToTarget >= 0)
        offsetToTarget += 7;
    return plusDays(offsetToTarget);
}
```

훨씬 명확해지고 간단해졌다.

```java
    public static SerialDate getNearestDayOfWeek(final int targetDOW,
                                                 final SerialDate base) {

        // check arguments...
        if (!SerialDate.isValidWeekdayCode(targetDOW)) {
            throw new IllegalArgumentException(
                "Invalid day-of-the-week code."
            );
        }

        // find the date...
        final int baseDOW = base.getDayOfWeek();
        int adjust = -Math.abs(targetDOW - baseDOW);
        if (adjust >= 4) {
            adjust = 7 - adjust;
        }
        if (adjust <= -4) {
            adjust = 7 + adjust;
        }
        return SerialDate.addDays(adjust, base);

    }
```

getNearestDayOfWeek 메서드 역시 직전 두 메서드와 마찬가지로 패턴을 맞추고 임시 변수 설명을 사용해 알고리즘을 표현하였다.

```java
public DayDate getNearestDayOfWeek(final Day targetDay) {
    int offsetToThisWeeksTarget = targetDay.index - getDayOfWeek().index;
    int offsetToFutureTarget = (offsetToThisWeeksTarget + 7) % 7;
    int offsetToPreviousTarget = offsetToFutureTarget - 7;
    if(offsetToFutureTarget > 3)
        return plusDays(offsetToPreviousTarget);
    else
        return plusDays(offsetToFutureTarget);
}
```

## 인스턴스 메서드의 사용

```java
    public SerialDate getEndOfCurrentMonth(final SerialDate base) {
        final int last = SerialDate.lastDayOfMonth(
            base.getMonth(), base.getYYYY()
        );
        return SerialDate.createInstance(last, base.getMonth(), base.getYYYY());
    }
```

getEndOfCurrentMonth는 어쩐지 이상하다.

DayDate 인스턴스를 인수로 받아 거기서 메서드를 호출하고 있다.

인스턴스 메서드로 바꾸고 이름을 수정하였다.

```java
public DayDate getEndOfMonth() {
    Month month = getMonth();
    int year = getYear();
    int lastDay = lastDayOfMonth(month, year);
    return DayDateFactory.makeDate(lastDay, month, year);
}
```

놀랍도록 코드는 간결하고 직관적이게 되었다.

## 정리

이제 지금까지 밥 아저씨가 리팩토링을 한 작업들을 정리해보자.

### 1. 처음에 나오는 주석은 너무 오래되었다. 개선

### 2. enum을 모두 독자적인 소스 파일로 옮겼다

### 3. 정적 변수와 정적 메서드를 DateUtil이라는 새 클래스로 옮겼다

### 4. 일부 추상 메서드를 DayDate 클래스로 끌어올렸다

### 5. Month.make를 Month.fromInt로 변경했다. 다른 enum 또한 toInt() 접근자를 생성하고 index 필드를 private로 변경했다

### 6. plusYears와 plusMonths에 중복이 있었다. correctLastDayOfMonth라는 새로운 메서드를 정의하므로써 중복을 없애고 세 메서드를 명확하게 하였다

### 7.팔방 미인으로 사용하던 숫자 1을 제거했다

흥미롭게도 테스트 커버리지는 84.9%로 오히려 감소했다.

테스트하는 코드가 줄어서라기 보다는, 클래스 크기 자체가 줄어들면서 테스트하지 않는 코드의 비중이 커졌기 때문이다.

그러나 테스트하지 않는 코드는 너무 사소해 테스트할 필요도 없는 코드들이다.

## 결론

보이스카우트 규칙을 기억하라.

우리가 코드를 수정하고 지나온 길은 반드시 처음보다 깨끗해야한다.

리팩토링을 통해 테스트 커버리지가 증가했고, 버그를 고쳤으며, 코드 크기가 줄었고, 코드가 명확해졌다.

다음 사람은 우리보다 코드를 조금 더 쉽고 명확하게 이해할 수 있게 될 것이다.
