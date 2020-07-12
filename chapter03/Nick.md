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
- 앞서 활용해본 구조를 적용
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
