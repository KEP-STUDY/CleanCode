# 11장 시스템

## 시스템 제작과 시스템 사용을 분리하라

- 소프트웨어 시스템은 (애프리케이션 객체를 제작하고 의존성을 서로 연결하는)준비 과정과 (준비 과정 이후에 이어지는) 런타임 로직을 분리해야 한다.

- 준비 과정 코드와 런타임 로직이 섞인 전형적인 예

```
public Service getService() {
  if(service == null)
    service = new MyServiceImpl(...);
  return service;
}
```

### 초기화 지연

- 장점
  - 실제로 필요할 때까지 객체 생성하지 않으므로 부하가 적다. 그만큼 애플리케이션 시작이 빨라진다.
  - 어떤 경우에도 null 포인터를 반환하지 않는다.
- 단점
  - getService 메서드가 MyServiceImpl과 생성자 인수에 명시적으로 의존한다.
  - 단위 테스트에서 getService 메서드를 호출하기 전에 **테스트 전용 객체**를 service 필드에 할당해야 한다.
  - 일반 런타임 로직에다 객체 생성 로직을 섞어놓은 탓에 모든 실행 경로도 테스트해야 한다. (SRP 위반)

### Main 분리

- 생성관 관련된 코드는 모두 main이나 main이 호출하는 모듈로 옮기고 나머지 시스템은 모든 객체가 생성되었고 모든 의존성이 연결되었다고 가정한다.
- main 함수에서 시스템에 필요한 객체를 생성한 후 이를 애플리케이션에 넘긴다. 애플리케이션은 그저 객체를 사용할 뿐
- 즉 애플리케이션은 main이나 객체가 생성되는 과정을 전혀 모른다.

### 팩토리

- **ABSTRACT FACTORY PATTERN** : 서로 관련이 있는 객체들을 통째로 묶어서 팩토리 클래스를 만들고, 이들 팩토리를 조건에 따라 생성하도록 다시 팩토리를 만들어서 객체를 생성하는 패턴 [링크](https://victorydntmd.tistory.com/300)
- FACTORY METHOD PATTERN : 조건에 따른 객체 생성을 팩토리 클래스로 위임하여, 팩토리 클래스에서 객체를 생성하는 패턴 [링크](https://victorydntmd.tistory.com/299)
- 추상 팩토리 패턴은 어떻게 보면, 팩토리 메서드 패턴을 좀 더 캡슐화한 방식

<img width="734" alt="abstract factory" src="https://user-images.githubusercontent.com/33617083/93022266-5c3cd400-f623-11ea-87f3-61edfe1d1cb5.png">

- OrderProcessing 애플리케이션은 LineItem이 생성되는 구체적인 방법을 모르지만, LineItem 생성되는 시점을 완벽하게 통제하며, 필요하다면 OrderProcessing 애플리케이션에서만 사용하는 생성자 인수도 넘길 수 있다.

### 의존성 주입

- 사용과 제작을 분리하는 강력한 메커니즘
- 제어 역전(Inversion of Control, IoC)을 의존성 관리에 적용한 메커니즘
- 제어 역전에서는 한 객체가 맡은 보조 책임을 새로운 객체에게 전적으로 떠넘긴다. 새로운 객체는 넘겨받은 책임만 맡으므로 SRP 성립
- 의존성 관리 맥락에서는 객체는 의존성 자체를 인스턴스로 만드는 책임을 지지 않는다. 이를 책임을 전적으로 담당하는 특수 **컨테이너**를 사용한다.

```
MyService myService = (MyService)(jndiContext.lookup(“NameOfMyService”));
```

- JNDI 검색은 의존성 주입을 부분적으로 구현한 기능이다.
- DI 컨테이너는 (대개 요청이 들어올 때마다) 필요한 객체의 인스턴스를 만든 후 생성자 인수나 설정자 메서드를 사용해 의존성을 설정한다.

### 확장

> 소프트웨어 시스템은 물리적인 시스템과 다르다. 관심사를 적절히 분리해 관리한다면 소프트웨어 아키텍처는 점진적으로 발전할 수 있다.

<br>

- Bank EJB용 EJB2 지역 인터페이스

```
package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public interface BankLocal extends java.ejb.EJBLocalObject {
 String getStreetAddr1() throws EJBException;
 String getStreetAddr2() throws EJBException;
 String getCity() throws EJBException;
 String getState() throws EJBException;
 String getZipCode() throws EJBException;
 void setStreetAddr1(String street1) throws EJBException;
 void setStreetAddr2(String street2) throws EJBException;
 void setCity(String city) throws EJBException;
 void setState(String state) throws EJBException;
 void setZipCode(String zip) throws EJBException;
 Collection getAccounts() throws EJBException;
 void setAccounts(Collection accounts) throws EJBException;
 void addAccount(AccountDTO accountDTO) throws EJBException;
}
```

- 상응하는 EJB2 엔티티 빈 구현

```
package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public abstract class Bank implements javax.ejb.EntityBean {
 // Business logic...
 public abstract String getStreetAddr1();
 public abstract String getStreetAddr2();
 public abstract String getCity();
 public abstract String getState();
 public abstract String getZipCode();
 public abstract void setStreetAddr1(String street1);
 public abstract void setStreetAddr2(String street2);
 public abstract void setCity(String city);
 public abstract void setState(String state);
 public abstract void setZipCode(String zip);
 public abstract Collection getAccounts();
 public abstract void setAccounts(Collection accounts);
 public void addAccount(AccountDTO accountDTO) {
  InitialContext context = new InitialContext();
  AccountHomeLocal accountHome = context.lookup("AccountHomeLocal");
  AccountLocal account = accountHome.create(accountDTO);
  Collection accounts = getAccounts();
  accounts.add(account);
 }

 // EJB container logic
 public abstract void setId(Integer id);
 public abstract Integer getId();
 public Integer ejbCreate(Integer id) { ... }
 public void ejbPostCreate(Integer id) { ... }

 // The rest had to be implemented but were usually empty:
 public void setEntityContext(EntityContext ctx) {}
 public void unsetEntityContext() {}
 public void ejbActivate() {}
 public void ejbPassivate() {}
 public void ejbLoad() {}
 public void ejbStore() {}
 public void ejbRemove() {}
}
```

- 비즈니스 논리가 컨테이너와 밀접하게 결합된 탓에 독자적인 단위 테스트가 어렵다.
- 컨테이너를 흉내 내거나 많은 시간을 낭비하며 EJB와 테스트를 실제 서버에 배치해야 한다.
- 따라서 EJB2 코드는 프레임워크 밖에서 재사용하기란 사실상 불가능
- 일반적으로 EJB2 빈은 DTO를 정의한다. DTO에는 메서드가 없으며 사실상 구조체다. 즉 동일한 정보를 저장하는 자료 유형이 두 개라는 의미이다. 그래서 한 객체에서 다른 객체로 자료를 복사하는 반복적인 규격 코드가 필요하다.

### 횡단(cross-cutting) 관심사

- 영속성과 같은 **관심사**는 애플리케이션의 자연스러운 객체 경계를 넘나드는 경향이 있다. 모든 객체가 전반적으로 동일한 방식을 이용하게 만들어야 한다.
- AOP에서 **관점**이라는 모듈 구성 개념은 "특정 관심사를 지원하려면 시스템에서 특정 지점들이 동작하는 방식을 일관성 있게 바꿔야 한다"라고 명시한다.

## 자바 프록시

- 자바 프록시는 단순한 상황에 적합하다. 개별 객체나 클래스에서 메서드 호출을 감싸는 경우가 좋은 예이다.
- 하지만, JDK에서 제공하는 동적 프록시는 인터페이스만 지원한다.

```
// Bank.java (suppressing package names...)
import java.utils.*;

// The abstraction of a bank.
public interface Bank {
 Collection<Account> getAccounts();
 void setAccounts(Collection<Account> accounts);
}

// BankImpl.java
import java.utils.*;

// The “Plain Old Java Object” (POJO) implementing the abstraction.
public class BankImpl implements Bank {
 private List<Account> accounts;

 public Collection<Account> getAccounts() {
  return accounts;
 }

 public void setAccounts(Collection<Account> accounts) {
  this.accounts = new ArrayList<Account>();
  for (Account account: accounts) {
    this.accounts.add(account);
  }
 }
}

// BankProxyHandler.java
import java.lang.reflect.*;
import java.util.*;

// “InvocationHandler” required by the proxy API.
public class BankProxyHandler implements InvocationHandler {
 private Bank bank;

 public BankHandler (Bank bank) {
  this.bank = bank;
 }

 // Method defined in InvocationHandler
 public Object invoke(Object proxy, Method method, Object[] args)
    throws Throwable {
  String methodName = method.getName();
  if (methodName.equals("getAccounts")) {
    bank.setAccounts(getAccountsFromDatabase());
    return bank.getAccounts();
  } else if (methodName.equals("setAccounts")) {
    bank.setAccounts((Collection<Account>) args[0]);
    setAccountsToDatabase(bank.getAccounts());
    return null;
  } else {
    ...
  }
 }

 // Lots of details here:
 protected Collection<Account> getAccountsFromDatabase() { ... }
 protected void setAccountsToDatabase(Collection<Account> accounts) { ... }
}

// Somewhere else...
Bank bank = (Bank) Proxy.newProxyInstance(
 Bank.class.getClassLoader(),
 new Class[] { Bank.class },
 new BankProxyHandler(new BankImpl()));
```

- 프록시 API에는 InvocationHandler를 넘겨 줘야 한다. 넘긴 InvocationHandler는 프록시에 호출되는 Bank 메서드를 구현하는 데 사용된다. BankProxyHandler는 자바 리플렉션 API를 사용해 제네릭스 메서드를 상응하는 BankImpl 메서드로 매핑한다.

## 순수 자바 AOP 프레임워크

- POJO는 순수하게 도메인에 초점을 맞춘다.
- 엔터프라이즈 프레임워크(그리고 다른 도메인)에 의존하지 않는다. 따라서 테스트가 개념적으로 더 쉽고 간단하다.
- POJO는 순수하게 도메인에 초점을 맞추고 엔터프라이즈 프레임워크에 의존하지 않는다.

<img width="782" alt="Decorator" src="https://user-images.githubusercontent.com/33617083/93099096-991bd000-f6e2-11ea-8bae-95bd8284735d.png">

- Bank 도메인 객체는 DAO로 프록시 되었으며, DAO는 JDBC 드라이버 자료 소스로 프록시되었다.
- 클라이언트는 Bank 객체에서 getAccounts()를 호출한다고 믿지만 실제로는 Bank POJO의 기본 동작을 확장한 중첩 Decorator 객체 집합의 가장 외곽과 통신한다.

```
XmlBeanFactory bf =
  new XmlBeanFactory(new ClassPathResource("app.xml", getClass()));
Bank bank = (Bank) bf.getBean("bank");
```

- 애플리케이션에서 DI 컨테이너에게 시스템 내 최상위 객체를 요청하기 위한 코드
- 스프링 관련 자바 코드가 거의 필요 없으므로 애플리케이션 **사실상 스프링과 독립적**이다.

```
package com.example.banking.model;
import javax.persistence.*;
import java.util.ArrayList;
import java.util.Collection;

@Entity
@Table(name = "BANKS")
public class Bank implements java.io.Serializable {
  @Id @GeneratedValue(strategy=GenerationType.AUTO)
  private int id;

  @Embeddable // An object “inlined” in Bank’s DB row
  public class Address {
    protected String streetAddr1;
    protected String streetAddr2;
    protected String city;
    protected String state;
    protected String zipCode;
  }

  @Embedded
  private Address address;

  @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER, mappedBy="bank")
  private Collection<Account> accounts = new ArrayList<Account>();

  public int getId() {
    return id;
  }

  public void setId(int id) {
    this.id = id;
  }

  public void addAccount(Account account) {
    account.setBank(this); accounts.add(account);
  }

  public Collection<Account> getAccounts() {
    return accounts;
  }

  public void setAccounts(Collection<Account> accounts) {
    this.accounts = accounts;
  }
}

```

## 테스트 주도 시스템 아키텍처 구축

- 코드 수준에서 아키텍처 관심사를 분리할 수 있다면, 진정한 **테스트 주도** 아키텍처 구축이 가능해진다.
- 최신의 시스템 구조는 각기 POJO 객체로 구현되는 모듈화된 관심사 영역으로 구성된다. 이렇게 서로 다른 영역은 해당 영역 코드에 최소한의 영향을 미치는 관점이나 유사한 도구를 사용해 통합한다. 이런 구조 역시 코드와 마찬가지로 테스트 주도 기법을 적용할 수 있다.

## 의사 결정을 최적화하라

- 관심사를 모듈로 분리한 POJO 시스템은 기민함을 제공한다. 이런 기민함 덕택에 최신 정보에 기반해 최선의 시점에 최적의 결정을 내리기가 쉬워진다. 또한 결정의 복잡성도 줄어든다.

## 명백한 가치가 있을 때 표준을 현명하게 사용하라

- 표준을 사용하면 아이디어와 컴포넌트를 재사용하기 쉽고, 적절한 경험을 가진 사람을 구하기 쉬우며, 좋은 아이디어를 캡슐화하기 쉽고, 컴포넌트를 엮기 쉽다. 하지만 때로는 표준을 만드는 시간이 너무 오래 걸려 업계가 기다리지 못한다. 어떤 표준은 원래 표준을 제정한 목적을 잊어버리기도 한다.

## 결론

- 시스템 역시 깨끗해야 한다. 깨끗하지 못한 아키텍처는 도메인 논리를 흐리며 기민성을 떨어뜨린다.
- POJO를 작성하고 관점 혹은 관점과 유사한 메커니즘을 사용해 각 구현 관심사를 분리해야 한다.
- 시스템을 설계하든 개별 모듈을 설계하든, 실제로 돌아가는 가장 단순한 수단을 사용하자
