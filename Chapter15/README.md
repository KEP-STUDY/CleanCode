# 15장 JUnit 들여다보기

1. 조건문 캡슐화 + 긍정문

```
public String compact(String message) {
  if (canBeCompacted()) {
    findCommonPrefix();
    findCommonSuffix();
    String compactExpected = compactString(expected);
    String compactActual = compactString(actual);
    return Assert.format(message, compactExpected, compactActual);
  } else {
    return Assert.format(message, expected, actual);
  }
}

private boolean canBeCompacted() {
  return expected != null && actual != null && !areStringsEqual();
}
```

2. 함수명 명확화 + 함수 분리

```
public String formatCompactedComparison(String message) {
  if (canBeCompacted()) {
    compactExpectedAndActual();
    return Assert.format(message, compactExpected, compactActual);
  } else {
    return Assert.format(message, expected, actual);
  }
}

private void compactExpectedAndActual() {
  findCommonPrefix();
  findCommonSuffix();
  compactExpected = compactString(expected);
  compactActual = compactString(actual);
}

```

3. 함수 사용 일관화 + 시간 결합 외부 노출

```
private void compactExpectedAndActual() {
  prefixIndex = findCommonPrefix();
  suffixIndex = findCommonSuffix(prefixIndex);
  compactExpected = compactString(expected);
  compactActual = compactString(actual);
}

private int findCommonPrefix() {
  int prefixIndex = 0;
  int end = Math.min(expected.length(), actual.length());
  for (; prefixIndex < end; prefixIndex++) {
    if (expected.charAt(prefixIndex) != actual.charAt(prefixIndex)) break;
  }
  return prefixIndex;
}

private int findCommonSuffix(int prefixIndex) {
  int expectedSuffix = expected.length() - 1;
  int actualSuffix = actual.length() - 1;
  for (; actualSuffix >= prefixIndex && expectedSuffix >= prefixIndex;
    actualSuffix--, expectedSuffix--) {
    if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix)) break;
  }
  return expected.length() - expectedSuffix;
}

```

4. 불필요한 중간 변수 제거

```
private void compactExpectedAndActual() {
  findCommonPrefixAndSuffix();
  compactExpected = compactString(expected);
  compactActual = compactString(actual);
}

private int findCommonPrefixAndSuffix() {
  findCommonPrefix();
  int expectedSuffix = expected.length() - 1;
  int actualSuffix = actual.length() - 1;
  for (; actualSuffix >= prefixIndex && expectedSuffix >= prefixIndex;
    actualSuffix--, expectedSuffix--) {
    if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix)) break;
  }
  return expected.length() - expectedSuffix;
}

private void findCommonPrefix() {
  prefixIndex = 0;
  int end = Math.min(expected.length(), actual.length());
  for (; prefixIndex < end; prefixIndex++)
    if (expected.charAt(prefixIndex) != actual.charAt(prefixIndex))
      break;
}

```

5. 함수 정리

```
private void findCommonPrefixAndSuffix() {
  findCommonPrefix();
  suffixLength = 0;
  for (; !suffixOverlapsPrefix(suffixLength); suffixLength++) {
    if (charFromEnd(expected, suffixLength) !=
        charFromEnd(actual, suffixLength))
      break;
  }
}

private char charFromEnd(String s, int i) {
  return s.charAt(s.length() - i - 1);
}

private boolean suffixOverlapsPrefix(int suffixLength) {
  return actual.length() - suffixLength <= prefixLength ||
    expected.length() - suffixLength <= prefixLength;
}
```

6. 불필요한 if문 제거

```
private String compactString(String source) {
  return
    computeCommonPrefix() +
    DELTA_START +
    source.substring(prefixLength, source.length() - suffixLength) +
    DELTA_END +
    computeCommonSuffix();
}
```

7. StringBuilder 활용

```
private String compact(String s) {
  return new StringBuilder()
    .append(startingEllipsis())
    .append(startingContext())
    .append(DELTA_START)
    .append(delta(s))
    .append(DELTA_END)
    .append(endingContext())
    .append(endingEllipsis())
    .toString();
}

private String startingEllipsis() {
  return prefixLength > contextLength ? ELLIPSIS : "";
}

private String startingContext() {
  int contextStart = Math.max(0, prefixLength - contextLength);
  int contextEnd = prefixLength;
  return expected.substring(contextStart, contextEnd);
}

private String delta(String s) {
  int deltaStart = prefixLength;
  int deltaEnd = s.length() - suffixLength;
  return s.substring(deltaStart, deltaEnd);
}

private String endingContext() {
  int contextStart = expected.length() - suffixLength;
  int contextEnd = Math.min(contextStart + contextLength, expected.length());
  return expected.substring(contextStart, contextEnd);
}

private String endingEllipsis() {
  return (suffixLength > contextLength ? ELLIPSIS : "");
}
```

## 결론

**_세상에 개선이 불필요한 모듈은 없다._**
