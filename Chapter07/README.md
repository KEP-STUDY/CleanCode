# 7장 오류처리

## 들어가기 전에

- 대다수의 코드 기반은 전적으로 오류 처리코드에 좌우된다.
- 하지만 오류 처리코드 때문에 비즈니스 로직이 더럽혀져서는 안된다.
- 우아하게 오류를 처리함으로써 깨끗하고 튼튼한 코드에 다가갈 수 있다.

## 오류 코드보다 예외를 사용하라

- 오류 플래그를 설정하거나 호출자에게 오류 코드를 반환하는 방법

```
public class DeviceController {
  public void sendShutDown() {
    DeviceHandle handle = getHandle(DEV1);
    // Check the state of the device
    if (handle != DeviceHandle.INVALID) {
    // Save the device status to the record field
    retrieveDeviceRecord(handle);
    // If not suspended, shut down
    if (record.getStatus() != DEVICE_SUSPENDED) {
      pauseDevice(handle);
      clearDeviceWorkQueue(handle);
      closeDevice(handle);
    } else {
      logger.log("Device suspended. Unable to shut down");
    }
    } else {
     logger.log("Invalid handle for: " + DEV1.toString());
    }
  }
```

- 예외를 던짐으로써 논리가 오류 처리 코드와 뒤섞이지 않는 방법

```
public class DeviceController {
  ...
  public void sendShutDown() {
    try {
      tryToShutDown();
    } catch (DeviceShutDownError e) {
      logger.log(e);
    }
  }

  private void tryToShutDown() throws DeviceShutDownError {
    DeviceHandle handle = getHandle(DEV1);
    DeviceRecord record = retrieveDeviceRecord(handle);
    pauseDevice(handle);
    clearDeviceWorkQueue(handle);
    closeDevice(handle);
  }

  private DeviceHandle getHandle(DeviceID id) {
    ...
    throw new DeviceShutDownError("Invalid handle for: " + id.toString());
    ...
  }
  ...
}
```

## Try-Catch-Fianlly 문부터 작성하라

- try 블록에서 무슨 일이 생기는지 catch 블록은 프로그램 상태를 일관성 있게 유지해야 한다.
- 먼저 강제로 예외를 일으키는 테스트 케이스를 작성한 후 테스트를 통과하게 코드를 작성하는 방법(TDD)을 권장한다.
- 그러면 자연스럽게 try 블록이 트랜잭션 범위부터 구현하게 되므로 범위 내에서 트랜잭션 본질을 유지하기 쉬워진다.

<br>

- 파일이 없으면 예외를 던지는지 알아보는 단위 테스트

```
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
  sectionStore.retrieveSection("invalid - file");
}
```

- 단위 테스트에 맞춰 다음 코드 구현

```
public List<RecordedGrip> retrieveSection(String sectionName) {
 // dummy return until we have a real implementation
 return new ArrayList<RecordedGrip>();
}
```

- 잘못된 파일 접근을 시도하게 구현을 변경

```
public List<RecordedGrip> retrieveSection(String sectionName) {
 try {
  FileInputStream stream = new FileInputStream(sectionName)
 } catch (Exception e) {
  throw new StorageException("retrieval error", e);
 }
 return new ArrayList<RecordedGrip>();
}
```

- catch 블록에서 예외 유형을 좁혀 실제로 FileInputStream 생성자가 던지는 FileNotFoundException을 잡아낸다.

```
public List<RecordedGrip> retrieveSection(String sectionName) {
 try {
  FileInputStream stream = new FileInputStream(sectionName);
  stream.close();
 } catch (FileNotFoundException e) {
  throw new StorageException("retrieval error”, e);
 }
 return new ArrayList<RecordedGrip>();
}
```

## 미확인(unchecked) 예외를 사용하라

- 확인된 예외는 OCP(Open Closed Principle)을 위반한다.
- 하위 단계에서 코드를 변경하면 상위 단계 메서드 선언부를 전부 고쳐야 한다. (연쇄적 수정)
- throws 경로에 위치하는 모든 함수가 최하위 함수에서 던지는 예외를 알아야하므로 캡슐화가 깨진다.

## 예외에 의미를 제공하라

- 예외를 던질 때는 전후 상황을 충분히 덧붙인다.
- 오류 메시지에 정보를 담아 예외와 함께 던진다.

## 호출자를 고려해 예외 클래스를 정의하라

- 애플리케이션에서 오류를 정의할 때 프로그래머에게 가장 중요한 관심사는 오류를 잡아내는 방법이 되어야 한다.

<br>

- 형편없이 분류한 사례

```
ACMEPort port = new ACMEPort(12);
try {
  port.open();
} catch (DeviceResponseException e) {
  reportPortError(e);
  logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
  reportPortError(e);
  logger.log("Unlock exception", e);
} catch (GMXError e) {
  reportPortError(e);
  logger.log("Device response exception");
} finally {
  …
}
```

- 호출하는 라이브러리 API를 감싸면서 예외 유형 하나를 반환

```
LocalPort port = new LocalPort(12);
try {
 port.open();
} catch (PortDeviceFailure e) {
 reportError(e);
 logger.log(e.getMessage(), e);
} finally {
 …
}
```

- LocalPort 클래스는 단순히 ACMEPort 클래스가 던지는 예외를 잡아 변환하는 Wrapper 클래스일 뿐이다.

```
public class LocalPort {
  private ACMEPort innerPort;

  public LocalPort(int portNumber) {
    innerPort = new ACMEPort(portNumber);
  }

  public void open() {
    try {
      innerPort.open();
    } catch (DeviceResponseException e) {
      throw new PortDeviceFailure(e);
    } catch (ATM1212UnlockedException e) {
      throw new PortDeviceFailure(e);
    } catch (GMXError e) {
      throw new PortDeviceFailure(e);
    }
  }
 …
}
```

## 정상 흐름을 정의하라

- catch 문에 넣은 필요없는 로직을 넣은 케이스

```
try {
 MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
 m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
 m_total += getMealPerDiem();
}
```

- 특수 사례 케이스 (Special Case Pattern): 예외적인 상황을 캡슐화로 처리

```
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```

```
public class PerDiemMealExpenses implements MealExpenses {
 public int getTotal() {
 // return the per diem default
 }
}
```

## null을 반환하지 마라

```
public void registerItem(Item item) {
 if (item != null) {
  ItemRegistry registry = peristentStore.getItemRegistry();
    if (registry != null) {
      Item existing = registry.getItem(item.getID());
      if (existing.getBillingPeriod().hasRetailOwner()) {
        existing.register(item);
      }
    }
  }
 }
```

```
List<Employee> employees = getEmployees();
if (employees != null) {
 for(Employee e : employees) {
 totalPay += e.getPay();
 }
}
```

```
List<Employee> employees = getEmployees();
for(Employee e : employees) {
 totalPay += e.getPay();
}
```

```
public List<Employee> getEmployees() {
 if( .. there are no employees .. )
 return Collections.emptyList();
}
```

## null을 전달하지 마라

```
public class MetricsCalculator
{
 public double xProjection(Point p1, Point p2) {
 return (p2.x – p1.x) * 1.5;
 }
 …
}
```

```
public class MetricsCalculator
{
 public double xProjection(Point p1, Point p2) {
  if (p1 == null || p2 == null) {
    throw InvalidArgumentException(
    "Invalid argument for MetricsCalculator.xProjection");
  }
  return (p2.x – p1.x) * 1.5;
 }
}
```

```
public class MetricsCalculator
{
 public double xProjection(Point p1, Point p2) {
 assert p1 != null : "p1 should not be null";
 assert p2 != null : "p2 should not be null";
 return (p2.x – p1.x) * 1.5;
 }
}
```

## 결론

- 깨끗한 코드는 읽기도 좋아야 하지만 안정성도 높아야 한다.
- 오류 처리를 프로그램 논리와 분리해 독자적인 사안으로 고려하면 튼튼하고 깨끗한 코드를 작성할 수 있다.
- 오류 처리를 프로그램 논리와 분리하면 독립적인 추론이 가능해지며 코드 유지보수성이 크게 높아진다.
