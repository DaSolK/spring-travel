# 토비의 스프링 Vol.1
## 스프링이란?
---
### 정의
- 자바 엔터프라이즈 애플리케이션 개발에 사용되는 애플리케이션 프레임워크
- 개발을 위한 **틀** / **공통 프로그래밍 모델** / **기술 API** 제공
#### 틀 - 스프링 컨테이너
- **스프링 컨테이너 / 애플리케이션 컨텍스트**라고 불리는 **스프링 런타임 엔진**을 제공
- 컨테이너가 설정 정보를 참고하여 애플리케이션을 구성하는 오브젝트를 생성/관리
- 독립적으로 동작할 수 있으나, 보통 웹 모듈에서 동작하는 서비스나 서블릿으로 등록하여 사용
- **<u>스프링 컨테이너를 다룰 줄 알며, 스프링 컨테이너가 애플리케이션 오브젝트를 이용할 수 있게 설정정보를 등록하는 법을 알아야 한다.</u>**
#### 공통 프로그래님 모델 - IoC/DI, 서비스 추상화, AOP
- **애플리케이션 코드가 어떻게 작성되어야 하는가에 대한 기준을 제시** *<u>(== 프로그래밍 모델)</u>*
  - IoC/DI
    - 오브젝트의 생명주기 / 의존관계에 대한 프로그래밍 모델
    - 스프링이 직접 제공하는 기술과 API, 컨테이너도 IoC/DI 방식으로 작성되어 있음.
  - 서비스 추상화
    - 환경/서버, 특정 기술에 종속되지 않고 이식성이 뛰어나며 유연하게 애플리케이션을 만들 수 있게 해주는 것.
  - AOP
    - 애플리케이션 코드에 산재해서 나타는 부가적인 기능들을 독립적으로 모듈화하는 프로그래밍 모델
    - 다양한 엔터프라이즈 서비스를 적용하면서, 깔끔한 코드를 유지할 수 있게 도움을 줌.
#### 기술 API
- 개발의 다양한 영역에 바로 활용할 수 있는 방대한 양의 기술 API를 제공
- UI 작성, 웹 프레젠테이션 계층, 비즈니스 서비스 계층, 기반 서비스 계층, 도메인 계층, 데이터 액세스 계층 등에서 필요한 주요 기술을 일관성있게 사용할 수 있게 지원해주는 기능 및 전략 클래스를 제공
- 모든 기술은 JavaEE (표준 자바 엔터프라이즈 플랫폼)이 기반
### 목표
- **위 세가지 요소를 적극 활용하여, 클래스는 스프링 컨테이너 위에서 오브젝트로 만들어져 동작하며, 코드는 스프링의 프로그래밍 모델에 따라 작성되고, 엔터프라이즈 기술을 사용할 때 스프링이 제공하는 기술API와 서비스를 활용하도록 개발한다.**

### 가치
#### 단순함 (Simplicity)
  - EJB(Enterpize JavaBeans) 의 복잡성을 해소하며, POJO 프로그래밍을 통해 가장 단순한 객체지향 개발 모델을 지향한다.
#### 유연성 (Flexibility)
  - 다양한 서드파티 프레임워크의 지원을 받듯이, 유연성/확장성의 강점을 두어 Glue 프레임워크 형태의 발전을 이룸.
---
# Chapter 01 : 오브젝트와 의존관계
## 초난감 DAO
**DAO (Data Access Object)** <u>DB를 이용해 데이터를 조회/조작하는 기능을 전담하는 오브젝트</u>
사용자 정보 저장/조회 DAO 만들기
### User
- 사용자 정보는 자바빈 규약을 따르는 오브젝트를 이용하면 편리
```java
public class User {
	String id;
	String name;
	String password;

	public String getId() {
		return id;
	}

	public void setId(String id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}
}
```
- **JAVA BEAN (자바빈)** 
  - 디폴트 생성자 : 자바빈은 파라미터가 없는 디폴트 생성자를 갖고 있어야 한다.
  - 프로퍼티 : 자바빈이 노출하는 이름을 가진 속성을 프로퍼티라고 한다. 프로퍼티는 set으로 시작하는 수정자메소드(setter)와 get으로 시작하는 접근자 메소드(getter)를 이용해 수정/조회 할 수 있다.
### UserDao
```java
package springbook.user.dao;

import springbook.user.domain.User;

import java.sql.*;

public class UserDao {
	public void add(User user) throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(
				"jdbc:mysql://localhost/springbook", "spring", "book");

		PreparedStatement ps = c.prepareStatement(
				"insert into users(id, user, password) value(?,?,?);");
		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());

		ps.executeUpdate();

		ps.close();
		c.close();
	}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.jdbc.Driver");
		Connection c = DriverManager.getConnection(
				"jdbc:mysql://localhost/springbook", "spring", "book");

		PreparedStatement ps = c.prepareStatement(
				"select * from users where id = ?");
		ps.setString(1, id);

		ResultSet rs = ps.executeQuery();
		rs.next();
		User user = new User();
		user.setId(rs.getString("id"));
		user.setName(rs.getString("name"));
		user.setPassword(rs.getString("password"));

		rs.close();
		ps.close();
		c.close();

		return user;
	}
}

```

### main() 을 활용한 DAO 테스트 코드
- 가장 간단한 검증 방법은 오브젝트 스스로 자신을 검증하는 것
- 모든 클래스에는 자신을 엔트리 포인트로 설정해 직접 실행이 가능하게 해주는 main 메소드가 있다.

```java
public static void main(String[] args) throws ClassNotFoundException, SQLException {
	UserDao dao = new UserDao();

	User user = new User();
	user.setId("whiteship");
	user.setName("백기선");
	user.setPassword("married");

	dao.add(user);

	System.out.println(user.getId() + " 등록 성공");

	User user2 = dao.get(user.getId());
	System.out.println(user2.getName());
	System.out.println(user2.getPassword());

	System.out.println(user2.getId() + " 조회 성공");
}
```
- **생각할 점**
  - 왜 이 코드에 문제가 많다고 할까?
  - 잘 동작하는 코드를 굳이 수정하고 개선해야 하는 이유?
  - DAO 코드를 개선했을 때의 장점?
  - 당장/미래에 주는 유익?
  - 객체지향 설계의 원칙과는 무슨 상관일까?
  - DAO를 개선하는 경우와 그대로 사용하는 경우, 스프링을 사용하는 개발에서 무슨 차이가 있을까?

## DAO의 분리
### 관심사 분리
- 객체 지향은 흔히 실세계를 최대한 가깝게 모델링해낼 수 있다는 의미가 있다고 여겨지나, 그보다 객체지향 기술이 만들어내는 **가상의 추상세계 자체를 효과적으로 구성할 수 있고, 이를 자유롭고 편리하게 변경/발전/확장시킬 수 있다는 것**에 더 의미가 있다.
- 변경이 일어날 때 필요한 작업을 최소화하고, 그 변경이 다른 곳에 문제를 일으키지 않게 하자 (**분리와 확장**)
- 모든 변화는 한번에 일어나지 않으며, 한번에 한 가지 관심에 집중되어 일어난다면, 우리는 한 가지 관심을 한 군데로 집중시키는 것.
- 관심이 같은 것끼리 모으고, 관심이 다른 것은 따로 떨어트린다. (**관심사의 분리[Separation of Concerns]**)
### 커넥션 만들기의 추출
#### UserDao의 관심사항
- DB와 연결을 위한 커넥션을 어떻게 가져올까
- 사용자 등록을 위해 DB에 보낼 SQL 문장을 담을 Statement를 만들고 실행
- 작업이 끝나면, 사용한 리소스를 닫고, 시스템에 돌려주는 것
#### 중복 코드의 메소드 추출
- 커넥션을 가져오는 중복 코드 분리
```java
public class UserDao {
	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c =getConnection();
		
		PreparedStatement ps = c.prepareStatement(
				"insert into users(id, name, password) value(?,?,?);");
		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());

		ps.executeUpdate();

		ps.close();
		c.close();
	}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c =getConnection();

		PreparedStatement ps = c.prepareStatement(
				"select * from users where id = ?");
		ps.setString(1, id);

		ResultSet rs = ps.executeQuery();
		rs.next();
		User user = new User();
		user.setId(rs.getString("id"));
		user.setName(rs.getString("name"));
		user.setPassword(rs.getString("password"));

		rs.close();
		ps.close();
		c.close();

		return user;
	}

	private Connection getConnection() throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.cj.jdbc.Driver");
		Connection c = DriverManager.Connection c = DriverManager.getConnection(
				"jdbc:mysql://localhost/springbook", "spring", "book");
		return c;
	}
}
```
 - 관심이 다른 코드가 있는 메소드에는 영향을 주지 않으며, 관심 내용이 독립적으로 존재하므로 수정이 간단해졌다.
 - 이제 N 사, D 사에서 추상클래스인 UserDao를 구입해, NUserDao, DUserDao 라는 서브클래스를 만든다.
 - 그럼 UserDao에서 추상 메소드로 선언한 getConnection()을 원하는 방식대로 구현이 가능하다.
 - 즉, UserDao 소스코드를 제공하지 않아도, getConnection() 메소드를 원하는 방식으로 확장해 UserDao 기능을 사용할 수 있다.
 - **DB 커넥션의 관심을 상속을 통해 서브클래스로 분리한 것**

```java
public abstract class UserDao {
	...
	public abstract Connection getConnection() throws ClassNotFoundException, SQLException;
}
```

```java
public class NUserDao extends UserDao {
	public Connection getConnection() throws ClassNotFoundException, SQLException {
		// N 사 DB connection 생성코드
	};
}

public class DUserDao extends UserDao {
	public Connection getConnection() throws ClassNotFoundException, SQLException {
		// D 사 DB connection 생성코드
	}
}
```

- DAO의 핵심 기능인 **어떻게 데이터를 등록하고 가져올 것인가(SQL 작성, 파라미터 바인딩, 쿼리 진행, 검색정보 전달)** 라는 관심을 담당하는 ***UserDao***
- **DB 연결 방법은 어떻게 할 것인가**라는 관심을 담고 있는 NUserDao,DUserDao가 <u>**클래스 레벨로 구분**</u>.
  - 클래스 계층구조를 통해 두 개의 관심이 독립적으로 분리되어 **변경** 작업이 용이
  - UserDao 코드 수정 없이, DB 연결 기능을 새롭게 정의한 클래스를 생성 가능 (**확장** 가능)
- 슈퍼클래스에 기본적인 로직의 흐름(커넥션 가져오기, SQL 생성, 실행, 반환)을 만들고, 그 기능의 일부를 추상 메소드나 오버라이딩이 가능한 protected 메소드 등으로 만든 뒤 서브클래스에서 필요에 맞게 구현하여 사용하도록 하는 방법을 **템플릿 메소드 패턴(template method pattern)** 이라고 한다.
- getConnection() 메소드는, 어떤 Connection 클래스의 오브젝트를 어떻게 생성할 것인지를 결정한다. 이렇게 서브클래스에서 구체적인 오브젝트 생성 방법을 결정하는 것을 **팩토리 메소드 패턴(factory method pattern)** 이라고 한다.
- 단, 상속은 간단하고 편리하게 느껴지지만 한계점이 있다.
  - **다중 상속을 허용하지 않는다.**
    - 이미 상속을 하고 있었다면?
  - **생각보다 상하위 클래스 관계가 밀접하다.**
    - 서브클래스는 슈퍼클래스를 사용할 수 있으며, 슈퍼클래스가 바뀔 때, 서브클래스도 수정이 필요할 수 있다.
  - **확장된 기능인 DB 커넥션을 생성하는 코드를 다른 DAO 클래스에 적용할 수 없다.**
    - 다른 DAO 클래스에도 getConnection()구현코드가 중복되어진다.

## DAO의 확장
- 관심사에 따라 분리한 오브젝트들은 제각기 독특한 변화를 가진다.
  - 데이터 액세스 로직
  - DB 연결
- 위 두가지 관심사는 변화의 성격이 다르다.
  - 변화의 이유
  - 시기
  - 주기
- 추상 클래스를 만들고 이를 상속한 서브클래스에서 변화가 필요한 부분을 바꿔서 쓸 수 있게 만든 이유는 ***변화의 성격이 다른 것을 분리해, 서로 영향을 주지 않은 채로 각각 필요한 시점에 독립적으로 변경할 수 있게 하기 위해서*** 이다.
- **다만, 상속을 사용했다는 사실이 불편하다.**

### 클래스의 분리
- 두 개의 관심사를 독립시키며 동시에 손쉽게 확장할 수 있는 방법
  - DB 커넥션과 관련된 부분을 별도 클래스로 제작
  - 해당 클래스를 UserDao가 이용

```java
public class UserDao {
	private SimpleConnectionMaker simpleConnectionMaker;

	public UserDao() {
		simpleConnectionMaker = new SimpleConnectionMaker();
	}

	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = simpleConnectionMaker.makeNewConnection();

		PreparedStatement ps = c.prepareStatement(
				"insert into users(id, name, password) value(?,?,?);");
		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());

		ps.executeUpdate();

		ps.close();
		c.close();
	}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = simpleConnectionMaker.makeNewConnection();

		PreparedStatement ps = c.prepareStatement(
				"select * from users where id = ?");
		ps.setString(1, id);

		ResultSet rs = ps.executeQuery();
		rs.next();
		User user = new User();
		user.setId(rs.getString("id"));
		user.setName(rs.getString("name"));
		user.setPassword(rs.getString("password"));

		rs.close();
		ps.close();
		c.close();

		return user;
	}
}
```

```java
public class SimpleConnectionMaker {
	public Connection makeNewConnection() throws ClassNotFoundException, SQLException {
		Class.forName("com.mysql.cj.jdbc.Driver");
		Connection c = DriverManager.getConnection(
				"jdbc:mysql://localhost/springbook", "spring", "book");
		return c;
	}
}
```
- 테스트를 통해 수정 전/후가 동일한지 확인
- 성격이 다른 코드를 분리는 하였으나, UserDao의 코드가 SimpleConnectioNMaker라는 특정 클래스에 종속이되어, 상속했을 때 처럼 **UserDao 코드 수정없이 DB 커넥션 생성 기능을 변경할 수가 없다.**
- 자유로운 확장을 위해서는 다음 문제가 해결되어야 한다.
  - SimpleConnectionMaker 자체
    - 현재 makeNewConnection으로 가져오는 코드를 N사 나 D사에서 다르게 활용하면 모두 새롭게 변경해주어야 한다.
    - 모든 함수에 전부 변경이 들어가야 한다.
      - `Connection c = simpleConnectionMake.openConnection();`
  - DB 커넥션을 제공하는 클래스가 어떤 것인지, UserDao가 구체적으로 알고 있어야 한다.
    - N사나 D사에서 다른 클래스를 구현하면 어쩔 수 없이 UserDao 자체 수정이 일어난다.
- 근본적인 원인은 **UserDao가 바뀔 수 있는 정보, 즉 DB 커넥션을 가져오는 클래스에 대해 너무 많이 알고 있다.** 는 것이다.
  - 어떤 클래스가 쓰일 지
  - 커넥션을 가져오는 메소드는 무엇인지

### 인터페이스 도입
- 두 클래스가 서로 긴밀하지 않도록, 추상적인 느슨한 연결고리를 도입
  - **추상화** : ***어떤 것들의 공통적인 성격을 뽑아내어 이를 따로 분리하는 것***
- 오브젝트를 만들려면 구체적인 클래스 하나를 선택해야하지만, 인터페이스로 추상화해놓은 최소한의 통로를 통해 접근하는 쪽에서는 **오브젝트를 만들 때 사용할 클래스가 무엇인지 몰라도 된다.**
- 인터페이스를 통해 접근하면 **실제 구현 클래스를 바꿔도 신경 쓸 일이 없다.**
- **인터페이스**는 ***어떤 일을 하겠다는 기능만 정의한 것***

```java
public interface ConnectionMaker {
	public Connection makeConnection() throws ClassNotFoundException, SQLException;
}
```
```java
public class DConnectionMaker implements ConnectionMaker {
	public Connection makeConnection() throws ClassNotFoundException, SQLException {
		// D 사의 독자적인 방법으로 Connection을 생성하는 코드
	}
}
```
```java
public class UserDao {
	private ConnectionMaker connectionMaker;

	public UserDao() {
		connectionMaker = new DConnectionMaker();
	}

	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = connectionMaker.makeConnection();

		PreparedStatement ps = c.prepareStatement(
				"insert into users(id, name, password) value(?,?,?);");
		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());

		ps.executeUpdate();

		ps.close();
		c.close();
	}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = connectionMaker.makeConnection();

		PreparedStatement ps = c.prepareStatement(
				"select * from users where id = ?");
		ps.setString(1, id);

		ResultSet rs = ps.executeQuery();
		rs.next();
		User user = new User();
		user.setId(rs.getString("id"));
		user.setName(rs.getString("name"));
		user.setPassword(rs.getString("password"));

		rs.close();
		ps.close();
		c.close();

		return user;
	}
}
```
- 인터페이스를 통해 매우 잘 분리한 듯 싶지만, 여전히 UserDao 내에는 DConnection이라는 클래스가 남아있게 되었다.
`connectionMaker = new DConnectionMaker();`

### 관계설정 책임의 분리
- UserDao 내의 분리되지 않은 관심사항을 찾아야 한다.
  - **UserDao가 어떤 ConnectionMaker 구현 클래스의 오브젝트를 이용하게 할지 결정하는 것**
  - **UserDao와 UserDao가 사용할 ConnectionMaker의 특정 구현 클래스 사이의 관계를 설정해주는 것에 관한 관심**
- **UserDao의 클라이언트** (**<u>UserDao를 사용하는 곳</u>**)가 UserDao와 ConnectionMaker 사이의 관계를 분리해서 두기에 적절하다.
  - 오브젝트 사이의 관계가 만들어지려면, 일단 만들어진 오브젝트가 있어야 하는데, 직접 생성자를 호출해 오브젝트를 만들 수도 있지만 **외부에서 만들어 준 것**을 가져올 수도 있다.
- 해당 인터페이스 타입의 오브젝트라면 파라미터 전달이 가능하고, 어떤 클래스로부터 만들어졌는지 몰라도, 정의된 메소드를 이용하면 신경을 쓰지 않는다.

```java
public UserDao(ConnectionMaker connectionMaker) {
	this.connectionMaker = connectionMaker;
}
```
- main은 **UserDao의 클라이언트** 로써, DConnectionMaker를 사용해 진행한다.

```java
public static void main(String[] args) throws ClassNotFoundException, SQLException {
	ConnectionMaker connectionMaker = new DConnectionMaker();
	UserDao dao = new UserDao(connectionMaker);

	User user = new User();
	user.setId("whiteship");
	user.setName("백기선");
	user.setPassword("married");

	dao.add(user);

	System.out.println(user.getId() + " 등록 성공");

	User user2 = dao.get(user.getId());
	System.out.println(user2.getName());
	System.out.println(user2.getPassword());

	System.out.println(user2.getId() + " 조회 성공");
}
```

### 원칙과 패턴
- 객체지향 설계 원칙
- **SOLID**
  - SRP(The Single Responsibility Princile) 단일 책임 원칙
  - OCP(The Open Closed Principle) 개방 폐쇄 원칙
  - LSP(The Liskov Subsitution Principle) 리스코프 치환 법칙
  - ISP(The Interface Segregation Principle) 인터페이스 분리 법칙
  - DIP(The Dependency Inversion Principle) 의존관계 역전 법칙
   
#### 개방 폐쇄 원칙 (OCP, Open-Closed Principle)
- 클래스나 모듈은 확장에는 열려있어야 하며, 변경에는 닫혀 있어야 한다.
  - UserDao는 DB 연결 방법에 대해 기능 확장이 열려있다.
  - UserDao 자신의 핵심 기능을 구현한 소스는 변화에 영향을 받지 않아 닫혀있다.
#### 높은 응집도와 낮은 결합도
##### 높은 응집도
- 응집도가 높다는 건 하나의 모듈, 클래스가 하나의 책임 또는 관심사에만 집중되어 있다는 것
- 변경이 일어날 때 모듈의 많은 부분이 함께 바뀌는 경우
  - 일부분만 일어나도 된다면, 모듈 전체에서 어떤 부분이 바뀌어야 하는지 파악이 필요
  - 바뀌지 않는 부분에 다른 영향을 미치지 않는 지 확인 필요
##### 낮은 결합도
- 느슨한 연결은 관계를 유지하는 데 꼭 필요한 최소한의 방법만 간접적인 형태로 제공, 나머지는 서로 독립적이고 알 필요도 없게 만들어 주는 것
- 변화에 대응하는 속도가 높아지고, 구성이 깔끔, 확장에 편리
- **결합도** : <u>하나의 오브젝트가 변경이 일어날 때에 관계를 맺고 있는 다른 오브젝트에게 변화를 요구하는 정도</u>
- 하나의 변경이 발생할 때 마치 파문이 이는 것처럼 여타 모듈과 객체로 변경에 대한 요구가 전파되지 않는 상태

#### 전략 패턴
- UserDaoTest - UesrDao - ConnectionMaker 는 디자인 패턴의 시각으로 **전략 패턴(Strategy Pattern)** 에 해당
- ***자신의 기능 맥락(Context)에서 필요에 따라 변경이 필요한 알고리즘(독립적인 책임으로 분리가 가능한 기능)을 인터페이스를 통해 통째로 분리시키고, 이를 구현한 구체적인 알고리즘 클래스를 필요에 따라 바꿔서 사용할 수 있게 하는 디자인 패턴***

## 제어의 역전(Inversion of Control)
### 오브젝트 팩토리
- UserDaoTest는 클라이언트(UserDao의 기능 테스트 목적)지만, UserDao가 ConnectionMaker 인터페이스를 구현한 특정 클래스로부터 완벽히 독립할 수 있도록 하는 기능을 추가로 맡고 있다.
  - UserDao와 ConnectionMaker 구현 클래스의 오브젝트 만들기
  - 두 개의 오브젝트가 연결돼서 사용될 수 있도록 관계를 맺어주기

#### 팩토리
- **팩토리(Factory)** : 객체의 생성 방법을 결정하고 만들어진 오브젝트를 돌려주는 기능을 하는 클래스
  - 추상 팩토리 패턴 / 팩토리 메소드 패턴과 다르다
- 오브젝트를 생성하는 쪽과 생성된 오브젝트를 사용하는 쪽의 역할과 책임을 깔끔하게 분리하려는 목적

```java
public class DaoFactory {
	public UserDao userDao() {
		ConnectionMaker connectionMaker = new DConnectionMaker();
		UserDao userDao = new UserDao(connectionMaker);
		return userDao;
	}
}
```

```java
public static void main(String[] args) throws ClassNotFoundException, SQLException {
		ConnectionMaker connectionMaker = new DConnectionMaker();
		UserDao dao = new DaoFactory().userDao();
		...
```

#### 설계도에서의 팩토리
- UserDao와 ConnectionMaker는 각각 애플리케이션의 핵심적인 데이터 로직과 기술 로직을 담당
  - 실질적인 로직을 담당하는 컴포넌트
- DaoFactory는 이런 애플리케이션의 오브젝트들을 구성하고 그 관계를 정의하는 책임을 맡고 있음.
  - 애플리케이션을 구성하는 컴포넌트의 구조와 관계를 정의한 설계도

### 오브젝트 팩토리의 활용
- AccountDao, MessageDao가 생겼을 때, ConnectionMaker 구현 클래스의 오브젝트를 생성하는 코드가 메소드마다 반복되게 된다. (중복)
- 분리가 필요

```java
public UserDao userDao() {
	return new UserDao(new DConnectionMaker());
}

public AccountDao accountDao() {
	return new AccountDao(new DConnectionMaker());
}

public MessageDao messageDao() {
	return new MessageDao(new DConnectionMaker());
}
```

```java
public class DaoFactory {
	public UserDao userDao() {
		return new UserDao(connectionMaker());
	}
	public AccountDao accountDao() {
		return new AccountDao(connectionMaker());
	}
	public MessageDao messageDao() {
		return new MessageDao(connectionMaker());
	}

	public ConnectionMaker connectionMaker() {
		return new DConnectionMaker();
	}
}
```
### 제어권의 이전을 통한 제어관계 역전
- ***프로그램의 제어 흐름 구조가 뒤바뀌는 것***
- 자신이 사용할 오브젝트를 선택하거나 생성하지 않는다.
- 자신도 어떻게 만들어지고 어디서 사용되는지 알 수 없다.
- 엔트리 포인트인 main()과 같은 경우를 제외하고, 모든 오브젝트는 **위임받은 제어 권한을 갖는 특별한 오브젝트에 의해 결정되고 만들어짐.**
- 라이브러리 / 프레임워크
  - 라이브러리 : 애플리케이션 흐름을 직접 제어, 동작 중 필요한 기능이 있을 때 능동적으로 라이브러리를 사용
  - 프레임워크 : 애플리케이션 코드가 프레임워크에 의해 사용됨.

## 스프링의 IoC
### 오브젝트 팩토리를 이용한 스프링 IoC
#### 애플리케이션 컨텍스트와 설정정보
- **빈(Bean)** : <u>*스프링이 제어권을 가지고 직접 만들고 관계를 부여하는 오브젝트*</u>
  - 자바빈 또는 엔터프라이즈 자바빈(EJB)에서 말하는 빈과 비슷한 오브젝트 단위의 애플리케이션 컴포넌트
  - 스프링 빈은 스프링 컨테이너가 생성과 관계설정, 사용 등을 제어해주는 제어의 역전이 적용된 오브젝트
- **빈 팩토리(Bean Factory)** : <u>*빈의 생성과 관계 설정 같은 제어를 담당*</u>
  - 보통 빈 팩토리보다, 좀 더 확장한 **애플리케이션 컨텍스트(Application Context)**를 사용
  
	#### DaoFactory를 사용하는 애플리케이션 컨텍스트
	- ***@Configuration*** : 빈 팩토리를 위한 오브젝트 설정을 담당하는 클래스라고 인식
	- ***@Bean*** : 오브젝트를 만들어주는 메소드
  
```java
@Configuration
public class DaoFactory {
	@Bean
	public UserDao userDao() {
		return new UserDao(connectionMaker());
	}

	@Bean
	public ConnectionMaker connectionMaker() {
		return new DConnectionMaker();
	}
}
```

```java
public static void main(String[] args) throws ClassNotFoundException, SQLException {

	ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
	UserDao dao = context.getBean("userDao", UserDao.class);

	User user = new User();
	user.setId("whiteship");
	user.setName("백기선");
	user.setPassword("married");

	dao.add(user);

	System.out.println(user.getId() + " 등록 성공");

	User user2 = dao.get(user.getId());
	System.out.println(user2.getName());
	System.out.println(user2.getPassword());

	System.out.println(user2.getId() + " 조회 성공");
}
```

- ***getBean()*** : ApplicationContext가 관리하는 오브젝트를 요청하는 메소드

### 애플리케이션 컨텍스트의 동작방식
- 앞서 만들었던 오브젝트 팩토리에 대응됨.
- DaoFactory는 UserDao를 비롯한 DAO 오브젝트를 생성하고 DB 생성 오브젝트와 관계를 맺어주는 제한적인 역할
- **애플리케이션 컨텍스트**는 ***애플리케이션에서 IoC를 적용해서 관리할 모든 오브젝트에 대한 생성과 관계설정을 담당.***
- 다만, 직접 오브젝트 생성과 관계를 맺는 코드가 없고, 별도 설정정보로 얻는다.
  - ***@Configuration*** 이 붙은 DaoFactory는 이 애플리케이션 컨텍스트가 활용하는 IoC 설정정보
  - 내부적으로는 애플리케이션 컨텍스트가 DaoFactory의 userDao() 메소드를 호출해서 오브젝트를 가져온 것을 클라이언트가 getBean()으로 요청할 때 전달
- 애플리케이션 컨텍스트는 DaoFactory 클래스를 설정정보로 등록해두고 ***@Bean*** 이 붙은 메소드의 이름을 가져와 빈 목록을 만든다.
- 클라이언트가 getBean()메소드를 호출하면, 빈 목록에서 요청한 이름을 찾고, 있다면 빈을 생성하는 메소드를 호출해서 오브젝트를 생성 / 클라이언트에 전달
- <u>오브젝트 팩토리와 비교해 애플리케이션 컨텍스트를 사용할 때의 장점</u>
  - ***클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다.***
  - ***애플리케이션 컨텍스트는 종합 IoC 서비스를 제공해준다***
  - ***애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공한다.***

### 스프링 IoC의 용어 정리
- **빈(Bean)**
  - 스프링이 IoC 방식으로 관리하는 오브젝트
  - 관리되는 오브젝트(Managed Object)
  - 스프링 애플리케이션에서 만들어지는 모든 오브젝트가 다 빈은 아니다.
    - <u>스프링이 직접 그 생성과 제어를 담당하는 오브젝트</u>만 빈이라고 한다.
- **빈 팩토리(Bean Factory)**
  - 스프링의 IoC를 담당하는 핵심 컨테이너
  - 빈 등록, 생성, 조회 등 관리
- **애플리케이션 컨텍스트(Application Context)**
  - 빈 팩토리를 확장한 IoC 컨테이너
  - 빈 팩토리 + 스프링이 제공하는 각종 부가서비스를 추가로 제공
- **설정정보/설정 메타정보(Configuration metadata)**
  - 애플리케이션 컨텍스트 또는 빈 팩토리가 IoC를 저용하기 위해 사용하는 메타정보
  - IoC 컨테이너에 의해 관리되는 애플리케이션 오브젝트를 생성하고 구성할 떄 사용
  - 애플리케이션의 형상 정보 혹은 blueprints(청사진)이라고도 한다.
- **컨테이너(Container) 또는 IoC 컨테이너**
  - IoC 방식으로 빈을 관리한다는 의미에서 애플리케이션 컨텍스트나 빈 팩토리를 컨테이너 또는 IoC 컨테이너라고도 한다.
- **스프링 프레임워크**
  - 스프링이 제공하는 모든 기능을 통틀어 말할 때 사용
## 싱글톤 레지스트리와 오브젝트 스코프
- DaoFactory의 userDao()를 여러번 호출했을 때 오브젝트는 동일한가?
- 애플리케이션 컨텍스트에는 어떠한가?

```java
DaoFactory factory = new DaoFactory();
UserDao dao1 = factory.userDao();
UserDao dao2 = factory.userDao();

System.out.println(dao1);
System.out.println(dao2);
```
`com.example.demo.user.dao.UserDao@30ee2816`
`com.example.demo.user.dao.UserDao@31d7b7bf`

```java
ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
UserDao dao1 = context.getBean("userDao", UserDao.class);
UserDao dao2 = context.getBean("userDao", UserDao.class);

System.out.println(dao1);
System.out.println(dao2);
```
`com.example.demo.user.dao.UserDao@1a6c1270`
`com.example.demo.user.dao.UserDao@1a6c1270`

### 싱글톤 레지스트리로서의 애플리케이션 컨텍스트
- 애플리케이션 컨텍스트는 ***싱글톤을 저장하고 관리하는 싱글톤 레지스트리(Singleton Registry)*** 이기도 하다.
  - 기본적으로 내부에서 생성하는 빈 오브젝트를 모두 싱글톤으로 만든다.
  - 디자인 패턴의 싱글톤 패턴과 비슷하지만, 그 구현 방법은 다르다.

#### 서버 애플리케이션과 싱글톤
- ***왜 싱글톤으로 빈을 만드는가?***
  - *<u>스프링이 주로 적용되는 대상이 자바 엔터프라이즈 기술을 사용하는 서버환경이기 때문</u>*
  - 요청 당 5개의 오브젝트, 초당 500개의 요청 = 초당 2500 오브젝트
    - 사용자 요청을 담당하는 여러 스레드에서 하나의 오브젝트를 공유

#### 싱글톤 패턴의 한계
- 클래스 밖에서 오브젝트를 생성하지 못하도록 생성자를 **private**으로 설정한다.
- 생성된 싱글톤 오브젝트를 저장할 수 있는 자신과 같은 타입의 **private static** 필드를 정의한다.
- Static factory method인 getInstance를 만들고, 이 메소드가 최초로 호출되는 시점에만 생성자를 호출해서 오브젝트를 생성시킨다. 생성된 오브젝트는 private static 필드에 저장된다. (또는 private ststic 필드의 초기값으로 오브젝트를 미리 만들어 둘 수 있다.)
- 오브젝트가 생성된 이후에는 getInstance 메소드를 통해 이미 만들어져 있는 private static 필드에 저장해둔 오브젝트를 넘겨준다.

```java
public class UserDao {
	private static UserDao INSTANCE;
	...
	
	private UserDao(ConnectionMaker connectionMaker) {
		this.connectionMaker = connectionMaker;
	}
	
	public static synchronized UserDao getInstance() {
		if (INSTANCE == null) INSTANCE = new UserDao(???);
		return INSTANCE;
	}
	...
}
```

- 싱글톤 패턴 구현 방식의 문제
  - private 생성자를 갖고 있어, 상속할 수 없다. (다형성 적용을 못한다.)
    - static 필드와 메소드를 사용하는 것도 동일하다.
  - 싱글톤은 테스트가 어렵다.
    - 만들어지는 방식이 제한적이라, 목 오브젝트 등으로 대체하기가 힘들다.
    - 초기화 과정에서 사용할 오브젝트를 Dynamic하게 주입하기도 어렵다.
  - 서버환경에서는 싱글톤이 하나만 만들어지는 것을 보장하지 못한다.
    - 서버에서 클래스 로더를 어떻게 구성하느냐에 따라, 하나 이상의 오브젝트가 생성 가능하다.
    - 여러 개의 JVM에 분산돼서 설치가 되는 경우에서도 싱글톤의 가치가 떨어진다.
  - 싱글톤의 사용은 전역 상태를 만들 수 있어 바람직하지 못하다.
    - 사용하는 클라이언트가 정해져 있지 않기 때문에, Global State가 된다.

#### 싱글톤 레지스트리
- **싱글톤 레지스트리(Singleton Registry)** 스프링은 직접 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공
  - 스태틱 메소드와 private 생성자를 사용해야 하는 비정상적인 클래스가 아니라 평범한 자바 클래스를 싱글톤으로 활용하게 해준다.
  - 평범한 자바 클래스라도 IoC 방식의 컨테이너를 사용해서 생성과 관계설정, 사용 등에 대한 제어권을 컨테이너에게 넘기면 손쉽게 싱글톤 방식으로 만들어져 관리되게 할 수 있다.
    - 테스트 환경에서도 자유롭게 오브젝트를 만들 수 있고, 테스트를 목적으로 한 목 오브젝트로 대체도 가능하다.
  - 싱글톤 패턴과 달리 스프링이 지지하는 객체지향적인 설계 방식과 원칙, 디자인 패턴(싱글톤 제외) 등을 적용하는데 아무 제약이 없다.

### 싱글톤과 오브젝트의 상태
- 멀티스레드 환경에서 여러 스레드가 동시에 접근할 수 있어, 상태 관리에 주의 필요
  - Stateless 방식으로 만들어져야 한다.
  - 파라미터와 로컬 변수, 리턴 값들을 이용한다.
    - 메소드 파라미터나 메소드 안에서 생성되는 로컬 변수는 매번 새로운 값을 저장할 독립적인 공간이 만들어지므로, 싱글톤이라고 해도 여러 스레드가 변수의 값을 덮어쓰지 않는다.
  - 읽기전용의 속성을 가진 정보로 활용한다.
### 스프링 빈의 스코프
- 빈이 생성되고, 존재하고, 적용되는 범위를 **스코프(Scope)**라고 한다.
- 기본 스코프는 싱글톤
- 프로토타입(ProtoType)스코프
  - 빈 요청 시, 매번 새로운 오브젝트를 반환
- 요청(Request) 스코프
  - 웹을 통해 HTTP 요청이 생길 때마다 생성
- 세션(Session) 스코프
  - 웹의 세션과 유사

## 의존관계 주입(DI)
### 제어의 역전(IoC)과 의존관계 주입
- 스프링 IoC 기능의 대표적인 동작원리는 주로 ***의존관계 주입(Dependency Injection)*** 이라고 불린다.
  - 오브젝트 레퍼런스를 외부로부터 제공(주입)받고 이를 통해 여타 오브젝트와 다이내믹하게 의존관계가 만들어지는 게 핵심
### 런타임 의존관계 설정
#### 의존관계
![image](https://user-images.githubusercontent.com/19977258/86368360-0c06d480-bcb8-11ea-99fe-5598a679ca4f.png)
- ***<u>A가 B에 의존하고 있음</u>***
- 의존의 의미
  - B가 변하면 A에 영향을 미친다.
  - B의 기능이 추가/변경 등이 되면 그 영향이 A로 전달된다.
  - EX> A에서 B에 정의된 메소드를 호출해서 사용하는 경우
- 의존관계에는 방향성이 있어, ***A가 B에 의존하고 있지만, B는 A에 의존하지 않는다.***
  - B는 A의 변화에 영향을 받지 않는다.
#### UserDao의 의존관계
- UserDao가 ConnectionMaker에 의존하고 있다.
  - UserDao가 ConnectionMaker 인터페이스를 사용하는 것
  - ConnectionMaker의 인터페이스가 변하면 UserDao가 직접적으로 영향을 받는다.
- ConnectionMaker 인터페이스를 구현한 클래스(DConnectionMaker)는 다른 것으로 바뀌거나 그 내부에서 사용하는 메소드에 변화가 생겨도 UserDao에 영향을 주지 않는다.
- 인터페이스에 대해서만 의존관계를 만들면, 결합도가 낮아 변화에 영향을 덜 받는다.
- 의존관계란 한쪽의 변화가 다른 쪽에 영향을 주는 것이니, ***인터페이스를 통해 의존관계에 제한을 주면 그만큼 변경이 자유로워진다.***
- 모델이나 코드에서 클래스와 인터페이스를 통해 드러나는 의존관계가 아닌, 런타임 시에 오브젝트 사이에서 만들어지는 의존관계도 있다.
  - 런타임 의존관계 또는 오브젝트 의존관계로, **설계 시점의 의존관계가 실체화된 것**
- 프로그램이 시작되고 <u>UserDao 오브젝트가 만들어지고 나서 런타임 시에 의존관계를 맺는 대상, 즉 실제 사용대상인 오브젝트</u>를 ***의존 오브젝트(Dependent Object)*** 라고 한다.
- **의존관계 주입**은 ***구체적인 의존 오브젝트와 그것을 사용할 주체, 보통 클라이언트라고 부르는 오브젝트를 런타임 시에 연결해주는 작업*** 을 말한다.
- 의존관계 주입의 조건
  - 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 그러기 위해서는 인터페이스에만 의존하고 있어야 한다.
  - 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제 3의 존재가 결정한다
  - 의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.

#### UserDao의 의존관계 주입
- UserDao를 만드는 시점에서 DConnectionMaker 오브젝트의 레퍼런스가 생성자 파라미터에 의해 전달된다.
- 두 오브젝트 간에 런타임 의존관계가 만들어지고, UserDao 오브젝트는 주입받은 DConnectionMaker 오브젝트를 언제든지 사용가능하게 된다.
- DI는 자신이 사용할 오브젝트에 대한 선택과 생성 제어권을 외부로 넘기고, 자신은 수동적으로 주입받는 오브젝트를 사용한다는 점에서 IoC의 개념에 잘 맞는다.

### 의존관계 검색과 주입
- 의존관계를 맺는 방법이 외부로부터의 주입이 아니라 스스로 검색을 이용하기 때문에 **의존관계 검색(Dependency lookup)** 이라고 불리는 것도 있다.
  - 자신이 필요한 의존 오브젝트를 능동적으로 찾는다.
  - 어떤 클래스의 오브젝트를 이용할지를 결정하지는 않는다.
  - 런타임 시 의존관계를 맺을 오브젝트를 결정하는 것과 오브젝트의 생성 작업은 외부 컨테이너에게 IoC로 맡기지만, 이를 가져올 떄는 메소드나 생성자를 통한 주입 대신 ***스스로 컨테이너에게 요청하는 방법*** 을 사용한다.
```java
public UserDao(){
	DaoFactory daoFactory = new DaoFactory();
	this.connectionMaker = daoFactory.connectionMaker();
}
```
- UserDao는 어떤 ConnectionMaker 오브젝트를 사용할지 알지 못한다.
  - 의존대상은 ConnectionMaker 인터페이스 뿐이다.
- 그러나 적용 방법은 외부로부터의 주입이 아니라, 스스로 IoC 컨테이너인 DaoFactory에게 요청하는 것이다.
- 애플리케이션 컨텍스트에서는 ***getBean()***이라는 메소드가 의존관계 검색에 사용된다.
```java
public UserDao(){
	ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
	this.connectionMaker = context.getBean("connectionMaker", ConnectionMaker.class);
}
```
- 의존관계 주입이 의존관계 검색보다 훨씬 단순하고 깔끔하다.
  - 의존관계 검색은 코드 안에 오브젝트 팩토리 클래스나, 스프링 API가 나타난다.
  - 애플리케이션 컴포넌트가 컨테이너와 같이 성격이 다른 오브젝트에 의존하게 되는 것이라 바람직하지 않다.
- 다만, 의존관계 검색을 사용해야 할 때가 있는데, **Static 메소드인 main()에서는 DI를 이용해 오브젝트를 주입받을 방법이 없으므로, 의존관계 검색을 사용해야 한다.**
- 서버에서도 main()과 같은 기동 메소드는 없지만, 사용자 요청을 받을 때마다  main() 메소드와 비슷한 역할을 하는 서블릿에서 스프링 컨테이너에 담긴 오브젝트를 사용하려면 한 번은 의존관계 검색 방식을 통해 오브젝트를 가져와야 한다.
  - 서블릿은 스프링이 미리 만들어서 제공하기 때문에 직접 구현하지 않아도 된다.
- 의존관계 검색과 의존관계 주입의 중요 차이점
  - ***의존관계 검색에서는 검색하는 오브젝트가 스프링 빈일 필요는 없다.***
    - UserDao에 getBean()을 사용하였을 때, UserDao가 굳이 Bean일 필요는 없다. (ConnectionMaker만 스프링 빈이기만 하면 된다.)
  - ***의존관계 주입에서는 UserDao와 ConnectionMaker 사이에 DI가 적용되려면 UserDao도 반드시 컨테이너가 만드는 빈 오브젝트여야 한다.***
    - 컨테이너가 UserDao에 ConnectionMaker 오브젝트를 주입해주려면  ***UserDao에 대한 생성과 초기화 권한을 갖고 있어야 하기 때문*** 이다.
### 의존관계 주입의 응용
#### 기능 구현의 교환
- 바라보는 DB 서버의 변경등의 상황에서 ConnectionMaker 생성 코드를 변경하는 것만으로도 전체 변경이 가능하다.
  - new ConnectionMaker()를 사용하는 모든 곳에 변경 적용하는 짓을 하지 않아도 된다.
```java
@Bean
public ConnectionMaker connectionMaker() {
	return new LocalDBConnectionMaker();
}

@Bean
public ConnectionMaker connectionMaker() {
	return new ProdDBConnectionMaker();
}
```
#### 부가기능 추가
- DB Connection Pool Open/Close 문제
  - 매번 Count 소스를 추가하고 삭제할 수 없다.
  - DI를 통해 Count 하는 오브젝트를 추가한다.
```java
public class CountingConnectionMaker implements ConnectionMaker {
	int counter = 0;
	private ConnectioNmaker realConnectioNmaker;

	public CountingConnectionMaker(ConnectionMaker realConnectionMaker) {
		this.realConnectionMaker = realConnectionMaker;
	}

	public Connection makerConnection() throws ClassNotFoundException, SQLException {
		this.counter++;
		return realConnectioMaker.makeConnection();
	}

	public int getCounter() {
		return this.counter;
	}
}
```
```java
@Configuration
public class CountingDaoFactory {
	@Bean
	public UserDao userDao() {
		return new UserDao(connectionMaker());
	}

	@Bean
	public ConnectionMaker connectionMaker() {
		return COuntingConnectionMaker(realConnectioNMaker());
	}
	
	@Bean
	public ConnectionMaker realConnectionMaker() {
		return new DConnectionMaker();
	}
}
```

### 메소드를 이용한 의존관계 주입
- 생성자가 아닌 일반 메소드를 이용해 의존 오브젝트와의 관계를 주입해주는 2가지 방법
  - **수정자 메소드(Setter)를 이용한 주입**
    - 외부로부터 제공받은 오브젝트 레퍼런스를 저장해뒀다가 내부의 메소드에서 사용하게 하는 DI 방식에서 활용하기에 적당하다.
  - **일반 메소드를 이용한 주입**
    - 한번에 여러 개의 파라미터를 받을 수 있지만, 그만큼 실수하기 쉽다.

```java
public void setConnectionMaker(ConnectionMaker connectionMaker) {
	this.connectionMaker = connectionMaker;
}
```
## XML을 이용한 설정
- 범용 DI 컨테이너를 사용하게 되면, 오브젝트 사이의 의존정보는 일잉이 자바 코드로 만들어주면 번거롭게 된다.
- 대부분 틀에 박힌 구조가 반복되며, 또한 DI 구성이 바뀔 때마다 자바 코드를 수정하고 자바 클래스를 다시 컴파일하는 것도 번거로운 일이다.
- DI 의존관계 설정정보를 만드는 일 중 가장 대표적인 것이 **XML** 이다.
  - 단순 텍스트 파일이기 때문에 다루기 쉽다.
  - 쉽게 이해할 수 있고, 컴파일과 같은 별도의 빌드 작업이 없다.
  - 환경이 달라져서 오브젝트의 관계가 바뀌는 경우에도 빠르게 변경사항 반영 가능
  - 스키마나 DTD를 이용해서 정해진 포맷을 따라 작성됐는지 손쉽게 확인 가능

### XML 설정
- DI 정보가 담긴 XML 파일은 **\<beans\>** 를 루트 엘리먼트로 사용한다.
- **\<beans\>** 안에는 여러개의  **\<bean\>** 을 정의할 수 있다.
  - **@Configuration** =  **\<beans\>**
  - **@Bean** = **\<bean\>**
- 하나의 @Bean 메소드를 통해 얻을 수 있는 빈의 DI 정보
  - 빈의 이름
    - @Bean 메소드 이름이 빈의 이름
    - getBean()에서 사용
  - 빈의 클래스
    - 빈 오브젝트를 어떤 클래스를 이용해서 만들지 정의
  - 빈의 의존 오브젝트
    - 빈의 생성자나 수정자 메소드를 통해 의존 오브젝트를 넣음.
    - 의존 오브젝트도 하나의 빈이므로 이름이 있고, 그 이름에 해당하는 메소드를 호출해서의종 오브젝트를 가져온다.
    - 의존 오브젝트는 하나 이상일 수도 있다.

#### connectionMaker() 전환
- DI 정의 세 가지 중에서 빈의 이름과 빈 클래스(이름) 두 가지를 **\<bean\>** 태그의 **id**와 **class** 애트리뷰트를 이용해 정의할 수 있다.
  - class 애트리뷰트에 지정하는 것은 자바 메소드에서 오브젝트를 만들 때 사용하는 클래스 이름 (**리턴타입이 아니며, XML에서는 리턴타입을 설정하지 않음**)
#### userDao() 전환
- DI 정의 세 가지 모두 존재한다.
- 수정자 메소드는 **Property**가 된다.
  - **\<property\>** 태그는 **name**과 **ref**를 갖는다.
    - **name** 은 **property**의 이름
    - **ref** 는 **수정자 메소드(setter)** 를 통해 주입해줄 오브젝트의 bean 이름

| userDao.setConnectionMaker(connectionMaker()) | \<property name="connectionMaker" ref="connectionMaker"\> |
|---|---|

```xml
<bean id="userDao" class="springbook.dao.UserDao">
	<property name="connectionMaker" ref="connectionMaker">
</bean>
```
#### XML의 의존관계 주입 정보
- **\<beans\>** 로 두개의 bean을 감싸주면 DaoFactory로부터 XML로의 전환이 끝이 난다.
```xml
<beans>
	<bean id="connectionMaker" class="springbook.dao.DConnectionMaker"/>
	<bean id="userDao" class="springbook.dao.UserDao">
		<property name="connectionMaker" ref="connectionMaker">
	</bean>
</beans>
```
- **name** 은 DI에서 사용할 Setter의 프로퍼티 이름
- **ref** 는 주입할 오브젝트를 정의한 빈의 ID
- 같은 인터페이스를 구현한 의존 오브젝트를 여러 개 정의해두고, 원하는 걸 골라서 DI하는 경우도 있다.
  - 각 빈의 이름을 독립적으로 만들고, ref로 DI 받을 빈을 지정
```xml
<beans>
	<bean id="localDBConnectionMaker" class="springbook.dao.LocalDBConnectionMaker"/>
	<bean id="testDBConnectionMaker" class="springbook.dao.TestDBConnectionMaker"/>
	<bean id="productionDBConnectionMaker" class="springbook.dao.ProductionDBConnectionMaker"/>
	<bean id="userDao" class="springbook.dao.UserDao">
		<property name="connectionMaker" ref="localDBConnectionMaker">
	</bean>
</beans>
```
### XML을 이용하는 애플리케이션 컨텍스트
- 애플리케이션 컨텍스트가 DaoFactory 대신 XML 설정정보를 활용하도록 교체하기
- IoC/DI는 **GenericXmlApplicationContext** 사용
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="connectionMaker" class="com.example.demo.user.dao.DConnectionMaker"/>
    <bean id="userDao" class="com.example.demo.user.dao.UserDao">
        <property name="connectionMaker" ref="connectionMaker"/>
    </bean>
</beans>
```
- AnnotationConfigApplicationContext 대신 **GenericXmlApplicationContext** 사용
```java
ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
```
### DataSource 인터페이스로 전환
#### DataSource 인터페이스 적용
- ConnectionMaker는 DB 커넥션을 생성해주는 기능 하나만을 담당
- **DataSource** 라는 인터페이스로 대체가능
```java
public class UserDao {
	private DataSource dataSource;

	public void setDataSource(DataSource dataSource) {
		this.dataSource = dataSource;
	}

	public void add(User user) throws ClassNotFoundException, SQLException {
		Connection c = dataSource.getConnection();

		PreparedStatement ps = c.prepareStatement(
				"insert into users(id, name, password) value(?,?,?);");
		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());

		ps.executeUpdate();

		ps.close();
		c.close();
	}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Connection c = dataSource.getConnection();

		PreparedStatement ps = c.prepareStatement(
				"select * from users where id = ?");
		ps.setString(1, id);

		ResultSet rs = ps.executeQuery();
		rs.next();
		User user = new User();
		user.setId(rs.getString("id"));
		user.setName(rs.getString("name"));
		user.setPassword(rs.getString("password"));

		rs.close();
		ps.close();
		c.close();

		return user;
	}
}
```

```java
@Configuration
public class DaoFactory {
	@Bean
	public UserDao userDao() {
		UserDao userDao = new UserDao();
		userDao.setDataSource(dataSource());
		return userDao;
	}

	@Bean
	public DataSource dataSource(){
		SimpleDriverDataSource dataSource = new SimpleDriverDataSource();
		dataSource.setDriverClass(com.mysql.cj.jdbc.Driver.class);
		dataSource.setUrl("jdbc:mysql://localhost/springbook");
		dataSource.setUsername("spring");
		dataSource.setPassword("book");

		return dataSource;
	}
}
```

#### XML 설정 방식
```xml
 <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource"/>
```
- 아직, DB 접속정보를 넣어주지 못했다는 문제점이 있다.

### 프로퍼티 값의 주입
#### 값 주입
- 수정자 메소드에는 다른 빈이나 오브젝트뿐 아니라, 스트링 같은 단순 값을 넣어줄 수도 있다.
- 즉, 다른 빈 오브젝트의 레퍼런스가 아닌 단순 정보도 오브젝트를 초기화하는 과정에서 setter에 넣을 수 있다.
- 그러나 다른 빈 오브젝트의 레퍼런스(ref)가 아니라 값이므로, **value** 를 사용한다.

```xml
<property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
<property name="url" value="`jdbc:mysql://localhost/springbook"/>
<property name="username" value="spring"/>
<property name="password" value="book"/>
```
#### value 값의 자동 변환
- driverClass는 스트링 타입이 아니라, **java.lang.Class** 타입이다.
- 이런 형태가 가능한 이유는 스프링이 프로퍼티의 값을, setter의 파라미터 타입을 참고로 해서 적절한 형태로 변경하기 때문

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	<bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
		<property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
		<property name="url" value="`jdbc:mysql://localhost/springbook"/>
		<property name="username" value="spring"/>
		<property name="password" value="book"/>
	</bean>

	<bean id="userDao" class="com.example.demo.user.dao.UserDao">
		<property name="dataSource" ref="dataSource"/>
	</bean>
</beans>
```

## 정리
- 책임이 다른 코드를 분리하여 두 개의 클래스로 만듦
  - 관심사의 분리
  - 리팩토링
- 바뀔 수 있는 쪽의 클래스는 인터페이스를 구현, 다른 클래스에서 인터페이스를 접근
- 인터페이스를 정의한 쪽의 구현 방법이 달라져 클래스가 바뀌어도, 그 기능을 사용하는 클래스는 수정할 필요 없이 만듦
  - 전략 패턴
- 자신의 책임 자체가 변경되는 경우 외에는 불필요한 변화가 발생하지 않게 막고, 자신이 사용하는 외부 오브젝트의 기능은 자유롭게 확장/변경 가능하도록 만듦
  - 개방 폐쇄 법칙
- 한쪽의 기능 변화가 다른 쪽의 변경을 요구하지 않으며 자신의 책임과 관심사에만 순수히 집중하는 코드를 만듦
  - 낮은 결합도, 높은 응집도
- 오브젝트가 생성되고 여타 오브젝트와 관계를 맺는 작업의 제어권을 별도의 오브젝트 팩토리를 만들어 넘김.
- 오브젝트 팩토리의 기능을 일반화한 IoC 컨테이너로 넘겨, 오브젝트가 자신이 사용할 대상의 생성이나 선택에 관한 책임으로부터 자유롭게 만듦
  - 제어의 역전/IoC
- 전통적인 싱글톤 패턴 구현 방식의 단점을 보고, 서버에서 사용되는 서비스 오브젝트로서의 장점을 살릴 수 있는 싱글톤을 사용하면서도 싱글톤 패턴의 단점을 극복할 수 있게 설계된 컨테이너를 활용하는 방법을 앎
  -  싱글톤 레지스트리
- 설계 시점과 코드에는 클래스와 인터페이스 사이의 느슨한 의존관계만 만들어놓고, 런타임시에 실제 사용할 구체적인 의존 오브젝트를 제3자(DI 컨테이너)의 도움으로 주입받아, 다이내믹한 의존관게를 갖게 해주는 IoC의 특별한 케이스를 앎
  -  의존관계 주입/DI
- 의존 오브젝트 주입 시, 생성자를 이용하는 방법, Setter를 이용하는 방법을 앎
  -  생성자 주입과 수정자 주입
- XML을 통해 DI 설정정보를 만드는 법, 의존 오브젝트가 아닌 일반 값을 외부에서 설정해 런타임 시 주입하는 방법을 앎
  - XML 설정 
