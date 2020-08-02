# 4장 주석

## 토론해봅시다

1. 좋은 주석과 나쁜 주석에 대해 이야기해보자.

2. 본인은 어떨 때 주석을 작성하는지 이야기해보자.

3. 주석이 있어서 도움이 되었던 경험이 있는지 이야기해보자.

<br>
<hr>

## 들어가기 전에

> 우리는 코드로 의도를 표현하지 못해서, 즉 실패를 만회하기 위해 주석을 사용한다.  
> 주석 없이는 자신을 표현할 방법을 찾지 못해 할 수 없이 주석을 사용한다.  
> 주석을 달 때마다 자신에게 표현력이 없다는 사실을 푸념해야 마땅하다.

<hr>

## 코드로 의도를 표현하라!

```
if ((employee.flags && HOURLY_FLAG) && (employee.age > 65))
```

```
if (employee.isEligibleForFullBenefits())
```

## 좋은 주석

- 법적인 주석

```
// Copyright (C) 2003,2004,2005 by Object Mentor, Inc. All rights reserved.
// GNU General Public License 버전 2 이상을 따르는 조건으로 배포한다.
```

- 정보를 제공하는 주석

```
// 테스트 중인 Responder 인스턴스를 반환한다.
protected abstract Responder responderInstance();
```

```
protected abstact Responder getResponderBeingTested()
```

```
// kk:mm:ss EEE, MMM dd, yyyy 형식
Pattern timeMatcher = Pattern.compile("\\d*:\\d*:\\d* \\w*, \\w* \\d* \\d*")
```

- 의도를 설명하는 주석

```
public int compareTo(Object o)
{
  if(o instanseof WikiPagePath)
  {
    WikiPagePath p = (WikiPagePath) o;
    String compressedName = StringUtil.join(names, "");
    String compressedArgumentName = StringUtil.join(p.names, "");
    String compressedName.compareTo(compressedArgumentName);
  }
  return 1; // 오른쪽 유형이므로 정렬 순위가 더 높다.
}
```

```
public void testConcurrentAddWidgets() throws Exception {
  WidgetBuilder widgetBuilder =
    new WidgetBuilder(new Class[]{BoldWidget.class});
  String text = "'''bold text'''";
  ParentWidget parent =
    new BoldWidget(new MockWidgetRoot(), "'''bold text'''");
  AtomicBoolean failFlag = new AtomicBoolean();
  failFlag.set(false);

  // 쓰레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다.
  for (int i = 0; i < 25000; i++) {
    WidgetBuilderThread widgetBuilderThread =
    new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
    Thread thread = new Thread(widgetBuilderThread);
    thread.start();
  }
  assertEquals(false, failFlag.get());
}
```

- 의미를 명료하게 밝히는 주석

```
public void testCompareTo() throws Exception
{
  WikiPagePath a = PathParser.parse("PageA");
  WikiPagePath ab = PathParser.parse("PageA.PageB");
  WikiPagePath b = PathParser.parse("PageB");
  WikiPagePath aa = PathParser.parse("PageA.PageA");
  WikiPagePath bb = PathParser.parse("PageB.PageB");
  WikiPagePath ba = PathParser.parse("PageB.PageA");
  assertTrue(a.compareTo(a) == 0); // a == a
  assertTrue(a.compareTo(b) != 0); // a != b
  assertTrue(ab.compareTo(ab) == 0); // ab == ab
  assertTrue(a.compareTo(b) == -1); // a < b
  assertTrue(aa.compareTo(ab) == -1); // aa < ab
  assertTrue(ba.compareTo(bb) == -1); // ba < bb
  assertTrue(b.compareTo(a) == 1); // b > a
  assertTrue(ab.compareTo(aa) == 1); // ab > aa
  assertTrue(bb.compareTo(ba) == 1); // bb > ba
}
```

- 결과를 경고하는 주석

```
// 여유 시간이 충분하지 않다면 실행하지 마십시오.
public void _testWithReallyBigFile()
{
  writeLinesToFile(10000000);
  response.setBody(testFile);
  response.readyToSend(this);
  String responseString = output.toString();
  assertSubString("Content-Length: 1000000000", responseString);
  assertTrue(bytesSent > 1000000000);
}
```

```
public static SimpleDateFormat makeStandardHttpDateFormat()
{
  // SimpleDateFormat은 스레드에 안전하지 못하다.
  // 따라서 각 인스턴스를 독립적으로 생성해야 한다.
  SimpleDateFormat df = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm:ss z");
  df.setTimeZone(TimeZone.getTimeZone("GMT"));
  return df;
}
```

- TODO 주석

```
// TODO-MdM 현재 필요하지 않다.
// 체크아웃 모델을 도입하면 함수가 필요없다.
protected VersionInfo makeVersion() throws Exception
{
  return null;
}
```

- 중요성을 강조하는 주석

```
String listItemContent = match.group(3).trim();
// 여기서 trim은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
// 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 때문이다.
new ListItemWidget(this, listItemContent, this.level + 1);
return buildList(text.substring(match.end()));
```

## 나쁜 주석

- 주절거리는 주석

```
public void loadProperties()
{
  try
  {
    String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE;
    FileInputStream propertiesStream = new FileInputStream(propertiesPath);
    loadedProperties.load(propertiesStream);
  }
  catch(IOException e)
  {
    // 속성 파일이 없다면 기본값을 모두 메모리로 읽어 들였다는 의미이다.
  }
}
```

- 같은 이야기를 중복하는 주석

```
// this.closed가 true일 때 반환되는 유틸리티 메서드이다.
// 타임아웃에 도달하면 예외를 던진다.
public synchronized void waitForClose(final long timeoutMillis)
throws Exception
{
  if(!closed)
  {
    wait(timeoutMillis);
    if(!closed)
     throw new Exception("MockResponseSender could not be closed");
  }
}
```

```
public abstract class ContainerBase
 implements Container, Lifecycle, Pipeline,
 MBeanRegistration, Serializable {

 /**
 * The processor delay for this component.
 */
 protected int backgroundProcessorDelay = -1;

 /**
 * The lifecycle event support for this component.
 */
 protected LifecycleSupport lifecycle =
 new LifecycleSupport(this);

 /**
 * The container event listeners for this Container.
 */
 protected ArrayList listeners = new ArrayList();

 /**
 * The Loader implementation with which this Container is
 * associated.
 */
 protected Loader loader = null;

 /**
 * The Logger implementation with which this Container is
 * associated.
 */
 protected Log logger = null;

 /**
 * Associated logger name.
 */
 protected String logName = null;

 /**
 * The Manager implementation with which this Container is
 * associated.
 */
 protected Manager manager = null;

 /**
 * The cluster with which this Container is associated.
 */
 protected Cluster cluster = null;

 /**
 * The human-readable name of this Container.
 */
 protected String name = null;

 /**
 * The parent Container to which this Container is a child.
 */
 protected Container parent = null;

 /**
 * The parent class loader to be configured when we install a
 * Loader.
 */
 protected ClassLoader parentClassLoader = null;

 /**
 * The Pipeline object with which this Container is
 * associated.
 */
 protected Pipeline pipeline = new StandardPipeline(this);

 /**
 * The Realm with which this Container is associated.
 */
 protected Realm realm = null;

 /**
 * The resources DirContext object with which this Container
 * is associated.
 */
 protected DirContext resources = null;
```

- 오해할 여지가 있는 주석

```
// this.closed가 true일 때 반환되는 유틸리티 메서드이다.
// 타임아웃에 도달하면 예외를 던진다.
public synchronized void waitForClose(final long timeoutMillis)
throws Exception
{
  if(!closed)
  {
    wait(timeoutMillis);
    if(!closed)
     throw new Exception("MockResponseSender could not be closed");
  }
}
```

- 의무적으로 다는 주석

```
 /**
 *
 * @param title The title of the CD
 * @param author The author of the CD
 * @param tracks The number of tracks on the CD
 * @param durationInMinutes The duration of the CD in minutes
 */
 public void addCD(String title, String author,
                   int tracks, int durationInMinutes) {
  CD cd = new CD();
  cd.title = title;
  cd.author = author;
  cd.tracks = tracks;
  cd.duration = duration;
  cdList.add(cd)
 }
```

- 이력을 기록하는 주석

```
 * Changes (from 11-Oct-2001)
 * --------------------------
 * 11-Oct-2001 : Re-organised the class and moved it to new package
 * com.jrefinery.date (DG);
 * 05-Nov-2001 : Added a getDescription() method, and eliminated NotableDate
 * class (DG);
 * 12-Nov-2001 : IBD requires setDescription() method, now that NotableDate
 * class is gone (DG); Changed getPreviousDayOfWeek(),
 * getFollowingDayOfWeek() and getNearestDayOfWeek() to correct
 * bugs (DG);
 * 05-Dec-2001 : Fixed bug in SpreadsheetDate class (DG);
 * 29-May-2002 : Moved the month constants into a separate interface
 * (MonthConstants) (DG);
 * 27-Aug-2002 : Fixed bug in addMonths() method, thanks to N???levka Petr (DG);
 * 03-Oct-2002 : Fixed errors reported by Checkstyle (DG);
 * 13-Mar-2003 : Implemented Serializable (DG);
 * 29-May-2003 : Fixed bug in addMonths method (DG);
 * 04-Sep-2003 : Implemented Comparable. Updated the isInRange javadocs (DG);
```

- 있으나 마나 한 주석

```
/**
 * 기본 생성자
 */
protected AnnualDateRule() {
}

/** 월 중 일자 */
 private int dayOfMonth;

/**
 * 월 중 일자를 반환한다.
 *
 * @return 월 중 일자
 */
public int getDayOfMonth() {
 return dayOfMonth;
}
```

- 함수나 변수로 표현할 수 있다면 주석을 달지 마라

```
// 전역 목록 <smoudle>에 속하는 모듈이 우리가 속한 하위 시스템에 의존하는가?
if (smodule.getDependSubsystems().contains(subSysMod.getSubSystem()))
```

```
ArrayList moduleDependees = smodule.getDependSubsystems();
String ourSubSystem = subSysMod.getSubSystem();
if (moduleDependees.contains(ourSubSystem))
```

- 위치를 표시하는 주석

```
// Actions ////////////////////////////
```

- 닫는 괄호에 다는 주석

```
public class wc {
 public static void main(String[] args) {
 BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
 String line;
 int lineCount = 0;
 int charCount = 0;
 int wordCount = 0;
 try {
    while ((line = in.readLine()) != null) {
      lineCount++;
      charCount += line.length();
      String words[] = line.split("\\W");
      wordCount += words.length;
    } //while
    System.out.println("wordCount = " + wordCount);
    System.out.println("lineCount = " + lineCount);
    System.out.println("charCount = " + charCount);
  } // try
  catch (IOException e) {
    System.err.println("Error:" + e.getMessage());
  } //catch
 }
```

- 공로를 돌리거나 저자를 표시하는 주석

- 주석으로 처리한 코드

- HTML 주석

- 전역 정보

```
/**
 * Port on which fitnesse would run. Defaults to <b>8082</b>.
 *
 * @param fitnessePort
 */
public void setFitnessePort(int fitnessePort)
{
  this.fitnessePort = fitnessePort;
}
```

- 너무 많은 정보

- 모호한 관계

```
/*
 * 모든 픽셀을 담을 만큼 충분한 배열로 시작한다. (여기에 필터 바이트를 더한다.)
 * 그리고 헤더 정보를 위해 200바이트를 더한다.
 */
 this.pngBytes = new byte[((this.width + 1) * this.height * 3) + 200];
```

- 함수 헤더

## 예제

```
/**
 * This class Generates prime numbers up to a user specified
 * maximum. The algorithm used is the Sieve of Eratosthenes.
 * <p>
 * Eratosthenes of Cyrene, b. c. 276 BC, Cyrene, Libya --
 * d. c. 194, Alexandria. The first man to calculate the
 * circumference of the Earth. Also known for working on
 * calendars with leap years and ran the library at Alexandria.
 * <p>
 * The algorithm is quite simple. Given an array of integers
 * starting at 2. Cross out all multiples of 2. Find the next
 * uncrossed integer, and cross out all of its multiples.
 * Repeat untilyou have passed the square root of the maximum
 * value.
 *
 * @author Alphonse
 * @version 13 Feb 2002 atp
 */
import java.util.*;

public class GeneratePrimes
{
 /**
 * @param maxValue is the generation limit.
 */
 public static int[] generatePrimes(int maxValue)
 {
  if (maxValue >= 2) // the only valid case
  {
    // declarations
    int s = maxValue + 1; // size of array
    boolean[] f = new boolean[s];
    int i;

    // initialize array to true.
    for (i = 0; i < s; i++)
    f[i] = true;

    // get rid of known non-primes
    f[0] = f[1] = false;

    // sieve
    int j;
    for (i = 2; i < Math.sqrt(s) + 1; i++)
    {
      if (f[i]) // if i is uncrossed, cross its multiples.
      {
        for (j = 2 * i; j < s; j += i)
        f[j] = false; // multiple is not prime
      }
    }
    // how many primes are there?
    int count = 0;
    for (i = 0; i < s; i++)
    {
      if (f[i])
      count++; // bump count.
    }
    int[] primes = new int[count];
    // move the primes into the result
    for (i = 0, j = 0; i < s; i++)
    {
      if (f[i]) // if prime
      primes[j++] = i;
    }
    return primes; // return the primes
  }
  else // maxValue < 2
    return new int[0]; // return null array if bad input.
 }
}
```

```
/**
 * This class Generates prime numbers up to a user specified
 * maximum. The algorithm used is the Sieve of Eratosthenes.
 * Given an array of integers starting at 2:
 * Find the first uncrossed integer, and cross out all its
 * multiples. Repeat until there are no more multiples
 * in the array.
 */
public class PrimeGenerator
{
  private static boolean[] crossedOut;
  private static int[] result;

  public static int[] generatePrimes(int maxValue)
  {
    if (maxValue < 2)
      return new int[0];
    else
    {
      uncrossIntegersUpTo(maxValue);
      crossOutMultiples();
      putUncrossedIntegersIntoResult();
      return result;
    }
  }

  private static void uncrossIntegersUpTo(int maxValue)
  {
    crossedOut = new boolean[maxValue + 1];
    for (int i = 2; i < crossedOut.length; i++)
    crossedOut[i] = false;
  }

  private static void crossOutMultiples()
  {
    int limit = determineIterationLimit();
    for (int i = 2; i <= limit; i++)
    if (notCrossed(i))
    crossOutMultiplesOf(i);
  }

  private static int determineIterationLimit()
  {
    // Every multiple in the array has a prime factor that
    // is less than or equal to the root of the array size,
    // so we don't have to cross out multiples of numbers
    // larger than that root.
    double iterationLimit = Math.sqrt(crossedOut.length);
    return (int) iterationLimit;
  }

  private static void crossOutMultiplesOf(int i)
  {
    for (int multiple = 2*i;
    multiple < crossedOut.length;
    multiple += i)
    crossedOut[multiple] = true;
  }

  private static boolean notCrossed(int i)
  {
    return crossedOut[i] == false;
  }

  private static void putUncrossedIntegersIntoResult()
  {
    result = new int[numberOfUncrossedIntegers()];
    for (int j = 0, i = 2; i < crossedOut.length; i++)
     if (notCrossed(i))
       result[j++] = i;
  }

  private static int numberOfUncrossedIntegers()
  {
    int count = 0;
    for (int i = 2; i < crossedOut.length; i++)
      if (notCrossed(i))
        count++;

    return count;
  }
}
```

## 결론 (개인적 견해)

- 주석의 필요성을 느꼈다면 코드 표현력이 부족하지 않았는지 생각해본다.
- util성 클래스 또는 메소드는 주석이 유용할 때도 있지만 서비스에서는 최대한 안 쓰는게 좋은 것 같다.
- 생각하기 귀찮으니깐 그냥 왠만하면 주석 쓰지 말자. ㅋ
