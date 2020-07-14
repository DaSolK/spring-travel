# Chapter 03 : 템플릿
---
- **개방 폐쇄 원칙(OCP)** 다시 생각해보기
  - 코드에서 어떤 부분은 변경을 통해 그 기능이 다양해지고 확장하려는 성질이 있고, 어떤 부분은 고정되어 있고 변하지 않으려는 성질이 있음
  - ***변화의 특성이 다른 부분을 구분해주고, 각각 다른 목적과 다른 이유에 의해 다른 시점에 독립적으로 변경될 수 있는 효율적인 구조를 만들어 주는 것***
  - **템플릿** 이란, ***이렇게 바뀌는 성질이 다른 코드 중에서 변경이 거의 일어나지 않으며 일정한 패턴으로 유지되는 특성을 가진 부분을 자유롭게 변경되는 성질을 가진 부분으로 부터 독립시켜, 효과적으로 활용할 수 있도록 하는 방법***

## 다시 보는 초난감 DAO
- UserDao에 없는 것은 **예외상황** 에 대한 처리다.
### 예외처리 기능을 갖춘 DAO
- DB 커넥션 같이, 제한적인 리소스를 사용할 때는 반드시 예외처리가 필요하다.
#### JDBC 수정 기능의 예외처리 코드
```java
public void deleteAll() throws SQLException {
  Connection c = dataSource.getConnection();

  PreparedStatement ps = c.prepareStatement("delete from users");
  ps.executeUpdate();

  ps.close();
  c.close();
}
```
- 위 메소드에서 이슈가 있으면, close가 되지 않아, 리소스 반환이 안될 수 있다.
- **try/catch/finally** 를 적용한다.

```java
public void deleteAll() throws SQLException {
  Connection c = null;
  PreparedStatement ps = null;
  try {
    c = dataSource.getConnection();
    ps = c.prepareStatement("delete from users");
    ps.executeUpdate();
  } catch (SQLException e) {
    throw e;
  } finally {
    if (ps != null) {
      try {
        ps.close();
      } catch (SQLException e) {
      }
    }
    if (c != null) {
      try {
        c.close();
      } catch (SQLException e) {
      }
    }
  }
}
```
- catch 문에는 로그를 남기는 등의 처리를 하자.

#### JDBC 조회 기능의 예외처리
- getCount()

```java
public int getCount() throws SQLException {
  Connection c = null;
  PreparedStatement ps = null;
  ResultSet rs = null;

  try {
    c = dataSource.getConnection();
    ps = c.prepareStatement("select count(*) from users");
    rs = ps.executeQuery();
    rs.next();
    return rs.getInt(1);
  } catch (SQLException e) {
    throw e;
  } finally {
    if (rs != null) {
      try {
        rs.close();
      } catch (SQLException e) {
      }
    }
    if (ps != null) {
      try {
        ps.close();
      } catch (SQLException e) {
      }
    }
    if (c != null) {
      try {
        c.close();
      } catch (SQLException e){
      }
    }
  }
}
```
## 변하는 것과 변하지 않는 것
### JDBC try/catch/finally 코드의 문제점
- 메소드의 반복 / 복잡성을 해소해야 한다.
- **핵심은 변하지 않는, 그러나 많은 곳에서 중복되는 코드와 로직에 따라 자꾸 확장되고 자주 변하는 코드를 잘 분리해내는 작업**
- 1장에서 본 것과 비슷하며, 같은 방법으로 접근하면 된다.
- 다만, DAO와 DB의 연결 기능을 분리하는 것과는 다른 성격이므로, 해결 방법이 다르다.
### 분리와 재사용을 위한 디자인 패턴 적용
- 변하는 성격이 다른 것 찾기
- **변하지 않는 부분**
```java
Connection c = null;
		PreparedStatement ps = null;
		try {
			c = dataSource.getConnection();
```
- **변하는 부분**
```java
ps = c.prepareStatement("delete from users");
```
- **변하지 않는 부분**
``` java
			ps.executeUpdate();
		} catch (SQLException e) {
			throw e;
		} finally {
			if (ps != null) {
				try {
					ps.close();
				} catch (SQLException e) {
				}
			}
			if (c != null) {
				try {
					c.close();
				} catch (SQLException e) {
				}
			}
		}
```
#### 메소드 추출

```java
public void deleteAll() throws SQLException {
  Connection c = null;
  PreparedStatement ps = null;
  try {
    c = dataSource.getConnection();
    ps = makeStatement(c);
    ps.executeUpdate();
  } catch (SQLException e) {
    throw e;
  } finally {
    if (ps != null) {
      try {
        ps.close();
      } catch (SQLException e) {
      }
    }
    if (c != null) {
      try {
        c.close();
      } catch (SQLException e) {
      }
    }
  }
}

private PreparedStatement makeStatement(Connection c) throws SQLException {
  PreparedStatement ps;
  ps = c.prepareStatement("delete from users");
  return ps;
}
```
- 당장은 이득이 없어보임
  - ***분리시키고 남은 메소드가 재사용이 필요한 부분이지만, 반대로 되었기 때문***

#### 템플릿 메소드 패턴의 적용
- **템플릿 메소드 패턴**은 ***상속을 통해 기능을 확장해서 사용하는 부분***
- ***변하지 않는 부분은 슈퍼클래스에 두고 변하는 부분은 추상 메소드로 정의해둬서 서브클래스에서 오버라이드하여 새롭게 정의해 쓰도록 하는 것***

```java
public class UserDaoDeleteAll extends UserDao {
  protected PreparedStatement makeStatement(Connection c) throws SQLException {
    PreparedStatement ps = c.prepareStatement("delete from users");
    return ps;
  }
}
```
- 위 템플릿 메소드 패턴으로의 접근은 제한이 많다.
  - DAO 로직마다 상속을 통해 새로운 클래스를 만들어야 한다는 점
  - 상황마다 모두 서브클래스를 만들어야 한다.
- 확장구조가 이미 클래스르 설계하는 시점에서 고정되어 버림

#### 전략 패턴의 적용
- **개발 폐쇄 법칙(OCP)**을 잘 지키는 구조이면서도 템플릿 메소드 패턴보다 유연하고 확장성이 뛰어난 것이, 오브젝트를 아예 둘로 분리하고 클래스 레벨에서는 인터페이스를 통해서만 의존하도록 만드는 전략 패턴
- OCP 관점에 보면 **확장에 해당하는 변하는 부분을 별도의 클래스로 만들어 추상화된 인터페이스를 통해 위임하는 방식** 이다.
![image](https://user-images.githubusercontent.com/19977258/87046528-492d1280-c234-11ea-8265-93e3902a0e62.png)

- deleteAll 메소드에서 변하지 않는 부분이 contextMethod()
- deleteAll()은 JDBC를 이용해 DB를 업데이트하는 작업이라는 변하지 않는 **Context** 를 갖는다.
  - DB 커넥션 가져오기
  - PreparedStatement를 만들어줄 외부 기능 호출
  - 전달받은 PreparedStatement 실행
  - 예외가 발생하면 이를 다시 메소드 밖으로 던지기
  - 모든 경우에 만들어진 PreparedStatement와 Connection을 적절히 닫아주기
- interface 구현

```java
public interface StatementStrategy {
  PreparedStatement makePreparedStatement(Connection c) throws SQLException;
}
```
```java
public class DeleteAllStatement implements StatementStrategy {
  public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
    PreparedStatement ps = c.prepareStatement("delete from users");
    return ps;
  }
}
```
```java
public void deleteAll() throws SQLException {
  Connection c = null;
  PreparedStatement ps = null;
  try {
    c = dataSource.getConnection();
    StatementStrategy strategy = new DeleteAllStatement();
    ps = strategy.makePreparedStatement(c);
    ps.executeUpdate();
  } catch (SQLException e) {
    throw e;
  } finally {
    if (ps != null) {
      try {
        ps.close();
      } catch (SQLException e) {
      }
    }
    if (c != null) {
      try {
        c.close();
      } catch (SQLException e) {
      }
    }
  }
}
```
- 아직 해당 전략을 고정한다는 점이 OCP에도 어긋난다.

#### DI 적용을 위한 클라이언트/컨텍스트 분리
- 1,2장에서 활용해본 구조를 적용
```java
public void jdbcContextWithStatementStratgy(StatementStrategy stmt) throws SQLException {
  Connection c = null;
  PreparedStatement ps = null;
  try {
    c = dataSource.getConnection();
    ps = stmt.makePreparedStatement(c);
    ps.executeUpdate();
  } catch (SQLException e) {
    throw e;
  } finally {
    if (ps != null) { try { ps.close(); } catch (SQLException e) {}}
    if (c != null) { try { c.close(); } catch (SQLException e) {}}
  }
}
```
- 클라이언트
```java
public void deleteAll() throws SQLException {
  StatementStrategy st = new DeleteAllStatement();
  jdbcContextWithStatementStratgy(st);
}
```
## JDBC 전략 패턴의 최적화
### 전략 클래스의 추가 정보
- add() 만들기
```java
public class AddStatement implements StatementStrategy {
  User user;

  public AddStatement(User user){
    this.user = user;
  }

  public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
    PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
    ps.setString(1, user.getId());
    ps.setString(2, user.getName());
    ps.setString(3, user.getPassword());

    return ps;
	}
}
```
```java
public void add(User user) throws ClassNotFoundException, SQLException {
  StatementStrategy st = new AddStatement(user);
  jdbcContextWithStatementStratgy(st);
}
```

### 전략과 클라이언트의 동거
- 문제점
  - DAO 메소드마다 새로운 StatementStrategy 구현 클래스를 만들어야 함.
  - DAO 메소드에서 StatementStrategy에 전달할 User와 같은 부가정보는 오브젝트를 전달받는 생성자와 이를 저장해둘 인스턴스 변수를 만들어야 함.
  
#### 로컬 클래스
- 많아지는 클래스를 해결하기 위한 좋은 방법은 매번 독립적인 파일로 만드는 것이 아닌 **내부 클래스** 로 정의 하는 것
- 특정 메소드에서만 사용된다면, 로컬 클래스가 가능

```java
public void add(User user) throws ClassNotFoundException, SQLException {
  class AddStatement implements StatementStrategy {
    User user;

    public AddStatement(User user){
      this.user = user;
    }

    public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
      PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
      ps.setString(1, user.getId());
      ps.setString(2, user.getName());
      ps.setString(3, user.getPassword());

      return ps;
    }
  }

  StatementStrategy st = new AddStatement(user);
  jdbcContextWithStatementStratgy(st);
}
```
- **중첩 클래스 분류**
  - 대른 클래스 내부에 정의되는 클래스를 중첩 클래스라고 함
    - 스태틱 클래스(Static Class)
      - 독립적으로 오브젝트로 만들어질 수 있음
    - 내부 클래스 (Inner Class)
      - 자신이 정의된 클래스의 오브젝트 안에서만 만들어짐.
      - 내부 클래스(member inner class)
        - 멤버 필드처럼 오브젝트 레벨에 정의됨
      - 로컬 클래스(local class)
        - 메소드 레벨에 정의
      - 익명 내부 클래스(anonymous inner class)
        - 이름을 갖지 않음
- 내부 클래스에서 외부의 변수를 사용할 때는 final로 설정해야 하며, user 파라미터는 메소드 내부에서 변경될 일이 없으니, 무방함.
  - 훨씬 간결하게 표현 가능해짐

```java
public void add(final User user) throws ClassNotFoundException, SQLException {
  class AddStatement implements StatementStrategy {
    public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
      PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
      ps.setString(1, user.getId());
      ps.setString(2, user.getName());
      ps.setString(3, user.getPassword());

      return ps;
    }
  }

  StatementStrategy st = new AddStatement();
  jdbcContextWithStatementStratgy(st);
}
```
- 메소드마다 추가해야 했던 클래스 파일을 줄이고, 내부 클래스 특징을 통해 로컬 변수를 가져다 쓸 수 있게 됨
#### 익명 내부 클래스
- 이름을 갖지 않는 클래스
- 클래스 선언과 오브젝트 생성이 결합된 형태
- 상속할 클래스나 구현할 인터페이스를 생성자 대신 사용하여, 사용
- 클래스 재사용이 없고, 구현한 인터페이스 타입으로만 사용할 경우 적용
- `new 인터페이스이름() { 클래스 본문 }`
```java
public void add(final User user) throws ClassNotFoundException, SQLException {
  jdbcContextWithStatementStratgy(new StatementStrategy() {
    @Override
    public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
      PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
      ps.setString(1, user.getId());
      ps.setString(2, user.getName());
      ps.setString(3, user.getPassword());

      return ps;
    }
  });
}
```
```java
public void deleteAll() throws SQLException {
  jdbcContextWithStatementStratgy(
      new StatementStrategy() {
        @Override
        public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
          return c.prepareStatement("delete from users");
        }
      }
  );
}
```
## 컨텍스트와 DI
### JdbcContext의 분리
- jdbcContextWithStatementStrategy() 는 다른 DAO에서도 사용 가능

#### 클래스 분리
- JdbcContext라는 이름으로 분리
- UserDao에 있던 컨텍스트 메소드를 workWithStatementStrategy()라는 이름으로 옮김
- DataSource가 필요한 것은 UserDao가 아니라, JdbcContext가 되버리므로, DataSource 타입 빈을 DI 받도록 만듦

```java
public class JdbcContext {
	private DataSource dataSource;

	public void setDataSource(DataSource dataSource) {
		this.dataSource = dataSource;
	}

	public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException {
		Connection c = null;
		PreparedStatement ps = null;
		try {
			c = dataSource.getConnection();
			ps = stmt.makePreparedStatement(c);
			ps.executeUpdate();
		} catch (SQLException e) {
			throw e;
		} finally {
			if (ps != null) { try { ps.close(); } catch (SQLException e) {}}
			if (c != null) { try { c.close(); } catch (SQLException e) {}}
		}
	}
}
```

```java
public class UserDao {
	private JdbcContext jdbcContext;

	public void setJdbcContext(JdbcContext jdbcContext) {
		this.jdbcContext = jdbcContext;
	}

	public void add(final User user) throws ClassNotFoundException, SQLException {
		this.jdbcContext.workWithStatementStrategy(new StatementStrategy() {
			@Override
			public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
				PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values(?,?,?)");
				ps.setString(1, user.getId());
				ps.setString(2, user.getName());
				ps.setString(3, user.getPassword());

				return ps;
			}
		});
	}

	public void deleteAll() throws SQLException {
		this.jdbcContext.workWithStatementStrategy(
				new StatementStrategy() {
					@Override
					public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
						return c.prepareStatement("delete from users");
					}
				}
		);
	}
}

```
#### 빈 의존관계 변경
- UserDao가 JdbcContext에 의존함
- JdbcContext는 인터페이스인 DataSource와 달리, 구체 클래스
- 일반적으로 인터페이스를 만들어야 하나, JdbcContext는 그 자체로 독립적인 JDBC 컨텍스트를 제공해주는 서비스 오브젝트로서의 의미이므로 구현 방법이 바뀔 가능성은 없다.
- 스프링 빈 설정은 클래스 레벨이 아니라 런타임 시에 만들어지는 오브젝트 레벨의 의존관계에 따라 정의됨.
- 의존관계를 따라, XML 재정의

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="jdbcContext" class="com.example.demo.user.dao.JdbcContext">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <bean id="userDao" class="com.example.demo.user.dao.UserDao">
        <property name="dataSource" ref="dataSource"/>
        <property name="jdbcContext" ref="jdbcContext"/>
    </bean>

    <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
        <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost/tobi?serverTimezone=UTC"/>
        <property name="username" value="root"/>
        <property name="password" value="qlsfkeps`12"/>
    </bean>
</beans>
```
- 모든 dataSource를 userDao의 메소드에서 분리한게 아니니, 아직 남김.

### JdbcContext의 특별한 DI
#### 스프링 빈으로 DI
- 인터페이스를 사용하지 않은 DI는 문제가 있을까?
- DI의 기본 의도에 맞게 JdbcContext의 메소드를 인터페이스로 뽑아내 정의해두고, UserDao에서 사용하게 해도 되지만, 꼭 그럴 필요는 없음.
- ***기본적으로 인터페이스를 사용해야하지만(온전한 DI라고 볼수 없지만) 스프링의 DI는 넓게 보자면 객체의 생성과 관계설정에 대한 제어권한을 오브젝트에서 제거하고 외부로 위임했다는 IoC라는 개념을 포괄한다.***
- **JdbcContext를 UserDao와 DI 구조로 만들어야 할 이유**
  - JdbcContext가 스프링 컨테이너의 싱글톤 레지스트리에서 관리되는 싱글톤 빈이기 때문
    - JdbcContext는 그 자체로 변경되는 상태정보를 갖고 있지 않음
    - 내부에서 사용할 dataSource라는 인스턴스 변수지만 읽기 전용이다.
    - JDBC 컨텍스트 메소드를 제공해주는 일종의 서비스 오브젝트로서 의미가 있으며, 싱글톤으로 구현되어 공유하는 것이 이상적
  - **JdbcContext가 DI를 통해 다른 빈에 의존하고 있기 때문**
    - JdbcContext는 dataSource 프로퍼티를 통해 DataSource 오브젝트를 주입받음
    - DI를 위해서는 양쪽 모두 스프링 빈으로 등록되어야 함.
- **인터페이스를 사용하지 않은 이유**
  - UserDao와 JdbcContext가 매우 긴밀하다(***응집도가 높다***)
    - ORM을 사용할 경우는 이와 다르게 구현되어야 한다.
  - 이런 사항은 가장 마지막 단계에서 고려되어야 한다.

#### 코드를 이용하는 수동 DI
- 다른 방법으로는 **UserDao 내부에서 직접 DI를 적용하는 것**
- 단, 이 경우, 싱글톤으로 만들려는 것은 포기해야 한다.
  - DAO 메소드가 호출될 때마다 JdbcContext 오브젝트를 매번 만드는 것은 아니며, DAO마다 하나의 JdbcContext를 갖게 하는것.
  - JdbcContext의 제어권은 UserDao가 갖게 하며, 사용할 오브젝트를 만들고 초기화하는 전통적인 방법을 사용
- JdbcContext가 다른 빈을 인터페이스를 통해 의존하는 내용은 DataSource 타입 빈을 다이내믹하게 주입 받아서 사용하는 것.
  - 그러나, 스프링의 빈이 아니니, DI 컨테이너를 통해 주입받을 수 없다.
  - 이런 경우, **UserDao에게 DI까지 맡기는 방법을 적용**
- 스프링의 설정파일에 userDao와 dataSource 두 개만 빈으로 정의

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userDao" class="com.example.demo.user.dao.UserDao">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
        <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost/tobi?serverTimezone=UTC"/>
        <property name="username" value="root"/>
        <property name="password" value="qlsfkeps`12"/>
    </bean>
</beans>
```
```java
public class UserDao {
  private DataSource dataSource;
  private JdbcContext jdbcContext;

  public void setDataSource(DataSource dataSource) {
    // jdbcCOntext 생성(IoC)
    this.jdbcContext = new JdbcContext();
    // 의존 오브젝트 주입(DI)
    this.jdbcContext.setDataSource(dataSource);
    this.dataSource = dataSource;
  }
  ...
```
- **인터페이스 없이 DAO와 밀접한 관계를 갖는 클래스를 DI에 적용하는 방법 2가지**
- 빈으로 등록하는 방법
  - 장점은 인터페이스를 두지 않아도 될 만큼 긴밀한 관계를 갖는 DAO 클래스와 JdbcCOntext를 어색하게 따로 빈으로 구분하지 않고 내부에서 직접 만들어 사용하면서도 다른 오브젝트에 대한 DI를 적용할 수 있다는 점.
  - 단점은 DI의 근본적인 원칙에 부합하지 않는 구체적인 클래스와의 관계가 설정에 직접 노출된다는 단점이 있다.
- 코드를 이용해 수동으로 DI하는 방법
  - JdbcContext가 UserDao의 내부에서 만들어지고 사용되면서 그 관계를 외부에는 드러내지 않는 장점
  - JdbcCOntext를 여러 오브젝트가 사용하더라도, 싱글톤으로 만들 수 없고, DI 작업을 위한 부가적인 코드가 필요

## 템플릿과 콜백
- 익명 내부 클래스를 활용한 방식
- **템플릿/콜백 패턴** 전략 패턴의 컨텍스트를 **템플릿**이라고 부르고, 익명 내부 클래스로 만들어지는 오브젝트를 **콜백** 이라고 부른다.

### 템플릿/콜백의 동작원리
- **템플릿**은 고정된 작업 흐름을 가진 코드를 재사용한다는 의미
- **콜백**은 템플릿 안에서 호출되는 것이 목적인 오브젝트

#### 템플릿/콜백의 특징
- 템플릿/콜팩 패턴의 콜백은 보통 단일 메소드 인터페이스를 사용
  - 템플릿의 작업 흐름 중 특정 기능을 위해 한 번 호출되는 경우가 일반적이기 때문
- 메소드 레벨에서 이루어지는 DI방식의 전략 패턴 구조
- 일반적인 DI는 인스턴스 변수를 Setter로 받아서 사용하지만, 메번 메소드 단위로 사용할 오브젝트를 새롭게 전달 받는 것

#### JdbcContext에 적용된 템플릿/콜백
- 리턴 값이 없는 단순한 구조
- 조회 작업에서는 보통 템플릿의 작업 결과를 클라이언트에 리턴해준다.
- 템플릿의 작업 흐름이 좀 더 복잡한 경우에는 한 번 이상 콜백을 호출하기도 하고 여러 개의 콜백을 클라이언트로부터 받아서 사용하기도 한다.

### 편리한 콜백의 재활용
- DAO 메소드는 간결해지고 최소한의 데이터 액세스 로직만 갖고 있게 만들어짐
- 단, DAO 메소드에서 매번 익명 내부 클래스를 사용하기 때문에 상대적으로 코드를 작성하고 읽기가 조금 불편함

#### 콜백의 분리와 재활용
- makePreparedStatement의 쿼리부분만 파라미터로 받으면 형태는 같다.

```java
public void deleteAll() throws SQLException {
  executeSql("delete from users");
}

private void executeSql(final String query) throws SQLException {
  this.jdbcContext.workWithStatementStrategy(
      new StatementStrategy() {
        @Override
        public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
          return c.prepareStatement(query);
        }
      }
  );
}
```
#### 콜백과 템플릿의 결합
- DAO가 공유할 수 있는 템플릿 클래스 안으로 옮겨도 된다.

``` java
public class JdbcContext {
	private DataSource dataSource;

	public void setDataSource(DataSource dataSource) {
		this.dataSource = dataSource;
	}

	public void workWithStatementStrategy(StatementStrategy stmt) throws SQLException {
		Connection c = null;
		PreparedStatement ps = null;
		try {
			c = dataSource.getConnection();
			ps = stmt.makePreparedStatement(c);
			ps.executeUpdate();
		} catch (SQLException e) {
			throw e;
		} finally {
			if (ps != null) { try { ps.close(); } catch (SQLException e) {}}
			if (c != null) { try { c.close(); } catch (SQLException e) {}}
		}
	}

	public void executeSql(final String query) throws SQLException {
		workWithStatementStrategy(
				new StatementStrategy() {
					@Override
					public PreparedStatement makePreparedStatement(Connection c) throws SQLException {
						return c.prepareStatement(query);
					}
				}
		);
	}
}
```
``` java
public void deleteAll() throws SQLException {
  this.jdbcContext.executeSql("delete from users");
}
```
- 일반적으로는 성격이 다른 코드는 분리하지만, 이 경우는 응집력이 강한 코드들이기 때문에 한 군데 모여 있는 게 유리

### 템플릿/콜백의 응용
#### 테스트와 try/catch/finally
- 기존
```java
public class CalcSumTest {
  @Test
  public void sumOfNumbers() throws IOException {
    String path = getClass().getResource("").getPath() + "numbers.txt";

    Calculator calculator = new Calculator();
    int sum = calculator.calcSum(path);

    assertThat(sum, is(10));
  }
}
```
- 클래스 분리
```java
public class Calculator {
  public int calcSum(String path) throws IOException {
    BufferedReader br = null;
    Integer sum = 0;
    String line = null;

    br = new BufferedReader(new FileReader(path));

    while ((line = br.readLine()) != null) {
        sum += Integer.valueOf(line);
    }

    br.close
    return sum;
  }
}
```
- try/catch/finally 추가

```java
public class Calculator {
  public int calcSum(String path) throws IOException {
    BufferedReader br = null;
    Integer sum = 0;
    String line = null;

    try {
      br = new BufferedReader(new FileReader(path));
      while ((line = br.readLine()) != null) {
          sum += Integer.valueOf(line);
      }
    } catch (IOException e) {
      e.printStackTrace();
      throw e;
    } finally {
      if (br != null) {
        try {
          br.close();
        } catch (IOException e) {
          e.printStackTrace();
        }
      }
    }
    return sum;
  }
}
```
#### 중복의 제거와 템플릿/콜백 설계
- 새롭게 추가되는 내용을 적용하기 위해 템플릿/콜백으로 재사용성을 높임.

```java
public interface BufferedReaderCallback {
  Integer doSomethingWithReader(BufferedReader br) throws IOException;
}
```
```java
public class Calculator {
  public int calcSum(String filepath) throws IOException {
    BufferedReaderCallback sumCallback = new BufferedReaderCallback() {
      @Override
      public Integer doSomethingWithReader(BufferedReader br) throws IOException {
        Integer sum = 0;
        String line = null;

        while ((line = br.readLine()) != null) {
            sum += Integer.valueOf(line);
        }

        return sum;
      }
    };
    return fileReadTemplate(filepath, sumCallback);
}

  public Integer calcMultiple(String filepath) throws IOException {
    BufferedReaderCallback sumCallback = new BufferedReaderCallback() {
      @Override
      public Integer doSomethingWithReader(BufferedReader br) throws IOException {
        Integer multifly = 1;
        String line = null;

        while ((line = br.readLine()) != null) {
            multifly *= Integer.valueOf(line);
        }

        return multifly;
      }
    };
    return fileReadTemplate(filepath, sumCallback);
  }

  public Integer fileReadTemplate(String path, BufferedReaderCallback callback) throws IOException {
    BufferedReader br = null;

    try {
      br = new BufferedReader(new FileReader(path));
      int ret = callback.doSomethingWithReader(br);
      return ret;
    } catch (IOException e) {
      e.printStackTrace();
      throw e;
    } finally {
      if (br != null) {
        try {
          br.close();
        } catch (IOException e) {
          e.printStackTrace();
        }
      }
    }
  }
}
```
```java
public class CalcSumTest {
  Calculator calculator;
  String numFilePath;

  @Before
  public void setUp() {
    this.calculator = new Calculator();
    this.numFilePath = getClass().getResource("").getPath() + "numbers.txt";
  }

  @Test
  public void sumOfNumbers() throws IOException { 
    assertThat(calculator.calcSum(this.numFilePath), is(10));
  }

  @Test
  public void multipleOfNumbers() throws IOException {    
    assertThat(calculator.calcMultiple(this.numFilePath), is(24));
  }
}
```

#### 템플릿/콜백의 재설계
- calcSum과 calcMultiply의 공통 부분(공통 패턴)이 있으며, 라인별 작업을 콜백 인터페이스로 정의

```java
public interface LineCallback {
  Integer doSomethingWithLine(String line, Integer value);
}
```
```java
public class Calculator {
  public Integer lineReadTemplate(String path, LineCallback callback, int initVal) throws IOException {
    BufferedReader br = null;

    try {
      br = new BufferedReader(new FileReader(path));
      Integer res = initVal;
      String line = null;
      while ((line = br.readLine()) != null) {
          res = callback.doSomethingWithLine(line, res);
      }
      return res;
    } catch (IOException e) {
      e.printStackTrace();
      throw e;
    } finally {
      if (br != null) {
        try {
          br.close();
        } catch (IOException e) {
          e.printStackTrace();
        }
      }
    }
  }
}
```
```java
public class Calculator {
  public int calcSum(String filepath) throws IOException {
    LineCallback sumCallback = new LineCallback() {
      @Override
      public Integer doSomethingWithLine(String line, Integer value) {
        return value + Integer.valueOf(line);
      }
    };
    return lineReadTemplate(filepath, sumCallback, 0);
  }

  public Integer calcMultiple(String filepath) throws IOException {
    LineCallback multipleCallback = new LineCallback() {
      @Override
      public Integer doSomethingWithLine(String line, Integer value) {
        return value * Integer.valueOf(line);
      }
    };
    return lineReadTemplate(filepath, multipleCallback, 1);
  }

  public Integer lineReadTemplate(String path, LineCallback callback, int initVal) throws IOException {
    BufferedReader br = null;

    try {
      br = new BufferedReader(new FileReader(path));
      Integer res = initVal;
      String line = null;
      while ((line = br.readLine()) != null) {
        res = callback.doSomethingWithLine(line, res);
      }
      return res;
    } catch (IOException e) {
      e.printStackTrace();
      throw e;
    } finally {
      if (br != null) {
        try {
          br.close();
        } catch (IOException e) {
          e.printStackTrace();
        }
      }
    }
  }
}
```

#### 제네릭스를 이용한 콜백 인터페이스
- 자바 언어에 타입 파라미터라는 개념을 도입한 **Generics(제네릭스)** 를 이용
- 결과 타입을 다양하게 가져갈 수 있음
- LineCallback의 리턴 값과 파라미터 타입을 제네릭 타입 파라미터 T로 선언

```java
public interface LineCallback<T> {
  T doSomethingWithLine(String line, T value);
}
```
```java
public <T> T lineReadTemplate(String filepath, LineCallback<T> callback, T initVal) throws IOException {
  BufferedReader br = null;
  try {
    br = new BufferedReader(new FileReader(filepath));
    T res = initVal;
    String line = null;
    while((line = br.readLine()) != null) {
      res = callback.doSomethingWithLine(line, res);
    }
    return res;
  }
  catch(IOException e) {...}
  finally {...}
}
```
```java
public String concatenate(String filepath) throws IOException {
  LineCallback<String> concatCallback = new LineCallback<String>() {
    @Override
    public String doSomethingWithLine(String line, String value) {
      return value + line;
    }
  };

  return lineReadTemplate(filepath, concatCallback, "");
}
```

```java
@Test
public void concatenateNumbers() throws IOException {   
  assertThat(calculator.concatenate(this.numFilePath), is("1234"));
}
```

## 스프링의 JdbcTemplate
- 스프링에서 제공하는 템플릿/콜백 기술 알아보기
- JDBC 코드용 기본 템플릿 = JdbcTemplate
```java
public class UserDao {
  private DataSource dataSource;
  private JdbcTemplate jdbcTemplate;

  public void setDataSource(DataSource dataSource) {
    this.jdbcTemplate = new JdbcTemplate();
    this.dataSource = dataSource;
  }
  ...
```
### update()
- deleteAll에 적용

```java
public void deleteAll() throws SQLException {
  this.jdbcTemplate.update(new PreparedStatementCreator() {
    @Override
    public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
      return con.prepareStatement("delete from users");
    }
  });
}
```
- executeSql과 비슷하게 적용

```java
public void deleteAll() throws SQLException {
  this.jdbcTemplate.update("delete from users");
}
```
- add 함수 변환
  
```java
public void add(final User user) throws ClassNotFoundException, SQLException {
  this.jdbcTemplate.update("insert into users(id, name, password) values (?,?,?)", user.getId(), user.getName(), user.getPassword());
}
```

### queryForInt()

```java
public int getCount() throws SQLException {
  return this.jdbcTemplate.query(new PreparedStatementCreator() {
    @Override
    public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
      return con.prepareStatement("select count(*) from users");
    }
  }, new ResultSetExtractor<Integer>() {
    @Override
    public Integer extractData(ResultSet rs) throws SQLException, DataAccessException {
      rs.next();
      return rs.getInt(1);
    }
  });
}
```
- 콜백을 만들기 위해 두번이나 익명 내부 클래스가 등장하지만, 두 파라미터를 통해 결과 값을 가져올 수 있다.
- **ResultSetExtractor** 는 제네릭스 타입 파라미터를 갖는다.
- 위 내용을 1줄로 줄여서 사용이 가능했으나, **최근에 삭제**

```java
public int getCount() throws SQLException {
  return this.jdbcTemplate.queryForInt("select count(*) from users");
}
```

### queryForObject()
- get 메소드는 결과에서 User 오브젝트를 만들어 프로퍼티에 넣어주어야 한다.
- ResultSetExtractor 콜백 대신 **RowMapper** 콜백을 사용
- RowMapper는 ResultSet의 로우 하나를 매핑하기 위해 사용되기 때문에 여러번 호출될 수 있다.

```java
public User get(String id) {
  return this.jdbcTemplate.queryForObject("select * from users where id = ?",
      new Object[]{id}, new RowMapper<User>() {
    public User mapRow(ResultSet rs, int rowNum) throws SQLException {
      User user = new User();
      user.setId(rs.getString("id"));
      user.setName(rs.getString("name"));
      user.setPassword(rs.getString("password"));
      return user;
    }
  });
}
```
- queryForObject는 SQL 실행시, 한개의 로우만 얻을 것으로 고려한다.
- ResultSet의 next를 실행해 첫번째 로우로 이동, RowMapper 콜백을 호출
- 조회 결과가 없으면 EmptyResultDataAccessException을 던진다.

### query()
