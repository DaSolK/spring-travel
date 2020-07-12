# Chapter 02 : 테스트
---
스프링에서 가장 중요한 가치는 **객체지향** 과 **테스트** 이다.
계속 변화하고 복잡한 애플리케이션에 필요한 대응
1. 확장과 변화를 고려한 객체지향적 설계와 이를 효과적으로 담아낼 수 있는 IoC/DI 기술
2. 만들어진 코드에 확신과 변화에 유연하게 대처할 수 있는 자신감을 주는 테스트

## UserDaoTest 다시 보기
### 테스트의 유용성
- 코드의 개선 과정에 꺼리낌과 불안 없이 적용할 수 있었다.
- 처음과 동일한 기능을 수행함을 보장
- **테스트** 란, ***내가 예상하고 의도했던 대로 코드가 정확하게 동작하는지 확인해서, 만든 코드를 확신할 수 있게 해주는 작업***
- 테스트가 원하는 결과로 나오지 않으면, 코드나 설계에 결함이 있음을 알 수 있음.
  - 결함 제거 (디버깅), 최종 테스트로 성공 보장

### UserDaoTest의 특징
```java
public static void main(String[] args) throws ClassNotFoundException, SQLException {

  ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
  
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
- 위 테스트 코드의 특징
  - 자바에서 가장 손쉽게 실행 가능한 main() 메소드를 이용
  - 테스트할 대상인 UserDao의 오브젝트를 가져와 메소드를 호출
  - 테스트에 사용할 입력 값(User 오브젝트)을 직접 코드에서 만들어 넣음
  - 테스트의 결과를 콘솔에 출력
  - 각 단계의 작업이 에러 없이 끝나면 콘솔에 성공 메시지로 출력
- 테스트 방법에서 돋보이는 점
  - main() 메소드를 이용해 쉽게 테스트 수행
  - 테스트 대상인 UserDao를 직접 호출

#### 웹을 통한 DAO 테스트 방법의 문제점
- 보통 웹 프로그램에서 사용하는 DAO를 테스트 하는 방법
  - DAO를 만든 뒤 바로 테스트 하지 않음
  - 서비스 계층, MVC 프레젠테이션 계층까지 포함한 모든 입출력 기능을 대충이라도 코드로 다 만듦
  - 테스트용 웹 애플리케이션을 서버에 배치한 후, 웹 화면을 띄워 폼을 열고, 값을 입력한 뒤 버튼을 눌러 등록
    - 폼 값을 받아서 파싱한 후, User오브젝트를 만들고 UserDao를 호출해주는 기능이 만들어져 있어야 함.
  - 에러가 없으면 검색 폼이나 파라미터를 지정할 수 있는 URL을 사용해 방금 입력한 데이터를 다시 가져올 수 있는지 테스트
    - UserDao가 돌려주는 결과를 화면에 출력해주는 기능이 만들어져 있어야 확인이 가능
- 단점
  - DAO 뿐만 아니라 서비스 클래스, 컨트롤러, JSP 뷰 등 모든 레이어의 기능을 다 만들고 나서야 테스트가 가능
  - 테스트 하는 중 에러가 나거나 테스트가 실패했으면, 어디서 문제가 발생하였는지 찾아야 함.
    - 하나의 테스트를 수행하는 데 참여하는 클래스와 코드가 너무 많음
  - 번거로우며 오류가 있을 때 빠르고 정확하게 대응하기가 힘들다.

#### 작은 단위의 테스트
- 테스트는 가능하면 작은 단위로 쪼개서 집중해서 할 수 있어야 함
  - 관심사의 분리라는 원리가 여기서도 적용됨
  - 테스트의 관심이 다르면 테스트할 대상을 분리하고 집중해서 접근해야 함
- UserDaoTest는 한 가지 관심에 집중할 수 있게 작은 단위로 만들어진 테스트
  - 웹 인터페이스 / MVC 클래스 / 서비스 오브젝트 등이 필요 없음
  - 서버 배포도 필요 없음
  - IDE나 도스창에서 테스트 가능
  - 에러가 나거나, 원치 않은 결과라면, UserDao 코드나, DB 연결 방법 정도의 문제로 간추려 진다.
- ***작은 단위의 코드에 대해 테스트를 수행한 것*** 을 **Unit Test**라고 한다.
  - 단위는 무엇이라 정의되지 않으며, 그 크기와 범위도 정해지지 않는다.
  - **충분히 하나의 관심에 집중해서 효율적으로 테스트할 만한 범위의 단위**
- 단위는 작을 수록 좋으며, 단위를 넘어서는 다른 코드들은 신경 쓰지 않고, 참여하지도 않고 테스트가 동작할 수 있으면 좋다.
- **단위 테스트의 이유**
  - ***개발자가 설계하고 만든 코드가 원래 의도한 대로 동작하는지를 개발자 스스로 빨리 확인받기 위해서***다.

#### 자동수행 테스트 코드
- UserDaoTest의 특징
  - 테스트할 데이터가 코드를 통해 제공
  - 테스트 작업 역시 코드를 통해 자동으로 실행
- 테스트는 자동으로 수행되도록 코드로 만들어지는 것이 중요
- 애플리케이션을 구성하는 클래스 안에 테스트 코드를 포함시키기 보다는 별도로 테스트용 클래스를 만들어, 테스트 코드를 넣는 것이 낫다.
- 자동 수행 테스트의 장점은 자주 반복할 수 있는 것

#### 지속적인 개선과 점진적인 개발을 위한 테스트
- 처음에는 단순 무식하게 정상동작하는 코드였지만, 테스트가 있어, 매우 작은 단계를 거쳐가며 계속 개선할 수 있었다.
- 기능 추가에 있어서도 점진적인 개발이 가능하다.

### UserDaoTest의 문제점
- 수동 확인 작업의 번거러움
  - 여전히 사람의 눈으로 확인하는 과정이 필요하다.
  - 콘솔의 나온 값을 보고 등록과 조회가 성공적으로 되고 있는지 확인해야 한다.
- 실행 작업의 번거러움
  - 매번 main()을 실행하는 것은 번거롭다.
  - DAO가 수백 개가 되고, 그에 대한 main()메소드도 그만큼 만들어지면, 전체 기능을 테스트해보기 위해 main() 을 수백번 실행해야 한다.
## UserDaoTest 개선
### 테스트 검증의 자동화
- add()에 전달한 User 오브젝트에 담긴 사용자 정보와 get()을 통해 다시 DB에서 가져온 정보가 서로 정확히 일치하는가?
- 모든 테스트는 성공과 실패로 나누어지고, 실패는 에러로 인한 실패와 결과값이 다른 실패로 나누어져, **테스트 에러** / **테스트 실패**로 구분된다.

```java
public static void main(String[] args) throws ClassNotFoundException, SQLException {

  ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

  UserDao dao = context.getBean("userDao", UserDao.class);

  User user = new User();
  user.setId("whiteship");
  user.setName("백기선");
  user.setPassword("married");

  dao.add(user);

  System.out.println(user.getId() + " 등록 성공");

  User user2 = dao.get(user.getId());

  if(!user.getName().equals(user2.getName())) {
    System.out.println("테스트 실패 (name)");
  } else if (!user.getPassword().equals(user2.getPassword())) {
    System.out.println("테스트 실패 (password)");
  } else {
    System.out.println("조회 테스트 성공");
  }
}
```

### 테스트의 효율적인 수행과 결과 관리
#### JUnit 테스트로 전환
- 프레임워크는 개발자가 만든 클래스에 대한 제어 권한을 넘겨받아서 주도적으로 애플리케이션의 흐름을 제어한다.
- 개발자가 만든 클래스의 오브젝트를 생성하고 실행하는 일은 프레임워크에 의해 진행

#### 테스트 메소드 전환
- 기존에 만들었던 main() 메소드 테스트는 프레임워크에 적용하기엔 적합하지 않다.
  - 테스트가 main() 메소드로 만들어졌다는 건 제어권을 직접 갖는다는 의미
- 우선적으로 **main() 메소드에 있던 테스트 코드를 일반 메소드로 옮기는 것** 이 필요
- JUnit 프레임워크가 요구하는 2가지 조건
  - ***메소드가 public으로 선언돼야 한다***
  - ***메소드에 @Test라는 애노테이션을 붙여주는 것***

```java
public class UserDaoTest {

	@Test
	public void addAndGet() throws SQLException {
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

		UserDao dao = context.getBean("userDao", UserDao.class);
		...
	}
}
```
- 테스트 의도가 명확하게 보이는 메소드 명을 설정하는 게 좋음.

#### 검증 코드 전환
- ***<u>소스 최신으로 변경함</u>***
- 테스트의 결과를 검증하는 if/else 문장을 JUnit이 제공하는 방법으로 전환
- **assertThat** 이라는 스태틱 메소드를 이용
  - 첫번째 파라미터의 값을 뒤에 나오는 **matcher** 라고 불리는 조건으로 비교해서 일치하면 다음으로 넘어가고, 아니면 테스트가 실패하도록 만들어 준다. **is()** 는 matcher의 일종으로 equals()로 비교해주는 기능을 가짐.

`assertThat(user2.getPassword(), is(user.getPassword()));`

```java
public class UserDaoTest {

	@Test
	public void addAndGet() throws SQLException, ClassNotFoundException {
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

		UserDao dao = context.getBean("userDao", UserDao.class);
		User user = new User();
		user.setId("gyumee");
		user.setName("박성철");
		user.setPassword("springno1");

		dao.add(user);

		User user2 = dao.get(user.getId());

		assertEquals(user2.getName(), user.getName());
		assertEquals(user2.getPassword(), user.getPassword());
	}
}
```

#### JUnit 테스트 실행
- 따로 환경설정으로 실행이 가능하지만, main()에 메소드를 추가해, JUnit 프레임워크를 시작해줄 수 있다.

## 개발자를 위한 테스팅 프레임워크 JUnit
### JUnit 테스트 실행 방법
#### IDE
- JUnit 테스트를 직관적이고, 소스와 긴밀하게 보여줌
#### 빌드 툴
- ANT나 메이븐 등의 빌드 툴과 스크립트를 사용하여 테스트가 가능

### 테스트 결과의 일관성
- 매번 DB 삭제를 하는 행위도 제어해, 테스트 수행 이전의 상태로 만들기
#### deleteAll()의 getCount() 추가
- deleteAll
  - USER 테이블의 모든 레코드를 삭제

```java
public void deleteAll() throws SQLException {
  Connection c = dataSource.getConnection();

  PreparedStatement ps = c.prepareStatement("delete from users");
  ps.executeUpdate();

  ps.close();
  c.close();
}
```
- getCount()
  - USER 테이블의 레코드 갯수를 돌려준다.

```java
public int getCount() throws SQLException {
  Connection c = dataSource.getConnection();

  PreparedStatement ps = c.prepareStatement("select count(*) from users");

  ResultSet rs = ps.executeQuery();
  rs.next();
  int count = rs.getInt(1);

  rs.close();
  ps.close();
  c.close();

  return count;
}
```
#### deleteAll()과 getCount()의 테스트
- 위 두가지 내용은 자동 실행이 되는 독립된 기능으로 만들기 애매하다.
  - 수동으로 USER 테이블에 넣은 뒤, 테스트를 해야하지만, 사람이 참여하게 되어 반복 실행에 어울리지는 않는다.
  - addAndGet()에 합쳐서 테스트 진행

```java
public class UserDaoTest {

	@Test
	public void addAndGet() throws SQLException, ClassNotFoundException {
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

		UserDao dao = context.getBean("userDao", UserDao.class);

		dao.deleteAll();
		assertEquals(dao.getCount(), 0);

		User user = new User();
		user.setId("gyumee");
		user.setName("박성철");
		user.setPassword("springno1");

		dao.add(user);
		assertEquals(dao.getCount(), 1);
		User user2 = dao.get(user.getId());

		assertEquals(user2.getName(), user.getName());
		assertEquals(user2.getPassword(), user.getPassword());
	}
}
```
#### 동일한 결과를 보장하는 테스트
- addAndGet() 테스트 실행 이전에 다른 이유로 USER 테이블에 데이터가 들어가 있다면, 이때는 테스트가 실패할 수 있다.
- addAndGet() 테스트만 DB를 사용하는 것이 아니면, 이전에 어떤 작업을 하다가 테스트를 실행할지 알 수 없다.
- 따라서, 지워주는 것도 좋지만, **테스트 이전에 테스트에 문제없는 상태를 만들어주는 것이 좋다**

### 포괄적인 테스트
- 테스트를 안만드는 것도 위험하지만, 성의없이 만드는 것도 위험하다.
- 특히, 한가지 결과만 검증하는 것은 상당히 위험하다.

#### getCount() 테스트

```java
public User(String id, String name, String password) {
  this.id = id;
  this.name = name;
  this.password = password;
}

public User() {}
```

- 한번에 여러개 생성가능한 생성자 먼저 구현

```java
@Test
public void count() throws SQLException, ClassNotFoundException {
  ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

  UserDao dao = context.getBean("userDao", UserDao.class);
  User user1 = new User("gyumee", "박성철", "springno1");
  User user2 = new User("leegw700", "이길원", "springno2");
  User user3 = new User("bumjin", "박범진", "springno3");

  dao.deleteAll();
  assertEquals(dao.getCount(), 0);

  dao.add(user1);
  assertEquals(dao.getCount(), 1);

  dao.add(user2);
  assertEquals(dao.getCount(), 2);

  dao.add(user3);
  assertEquals(dao.getCount(), 3);
}
```

#### addAndGet() 테스트 보완

```java
@Test
public void addAndGet() throws SQLException, ClassNotFoundException {
  ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

  UserDao dao = context.getBean("userDao", UserDao.class);
  User user1 = new User("gyumee", "박성철", "springno1");
  User user2 = new User("leegw700", "이길원", "springno2");

  dao.deleteAll();
  assertEquals(dao.getCount(), 0);

  dao.add(user1);
  dao.add(user2);
  assertEquals(dao.getCount(), 2);

  User userget1 = dao.get(user1.getId());
  assertEquals(userget1.getName(), user1.getName());
  assertEquals(userget1.getPassword(), user1.getPassword());

  User userget2 = dao.get(user2.getId());
  assertEquals(userget2.getName(), user2.getName());
  assertEquals(userget2.getPassword(), user2.getPassword());
}
```

#### get() 예외조건에 대한 테스트
- get 메소드에 전달된 id가 없다면 어떻게 할까
  - 하나는 null과 같은 특별한 값을 리턴하는 경우
  - id에 해당하는 정보를 찾을 수 없다고 예외를 던지는 것
- 어떻게 테스트 코드를 만들면 좋은가
  - 예외가 던져지면 테스트 메소드의 실행은 중단되고 테스트는 실패가 되나, 이번에는 반대로 예외가 던져저야 성공한 케이스
    - assertThat() 메소드로는 확인이 어렵다

```java
@Test(expected=EmptyResultDataAccessException.class)
public void getUserFailure() throws SQLException, ClassNotFoundException {
  ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

  UserDao dao = context.getBean("userDao", UserDao.class);
  dao.deleteAll();
  assertEquals(dao.getCount(), 0);

  dao.get("unknown_id");
}
```

***junit5 소스 수정***
```java
@Test
public void getUserFailure() throws SQLException, ClassNotFoundException {
  ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

  UserDao dao = context.getBean("userDao", UserDao.class);
  dao.deleteAll();
  assertEquals(dao.getCount(), 0);
  assertThrows(EmptyResultDataAccessException.class, () -> {
    dao.get("unknown_id");
  });
}
```
- 위 예시에서는 SQLException이 발생하여, 테스트가 실패하지만, 예외케이스에 대한 처리 방법을 알 수 있다.

#### 테스트를 성공시키기 위한 코드의 수정
- get 메소드의 수정
- 테스트의 성공을 볼 수 있다.

```java
public User get(String id) throws ClassNotFoundException, SQLException {
  Connection c = dataSource.getConnection();

  PreparedStatement ps = c.prepareStatement(
      "select * from users where id = ?");
  ps.setString(1, id);

  ResultSet rs = ps.executeQuery();

  User user = null;
  if(rs.next()) {
    user = new User();
    user.setId(rs.getString("id"));
    user.setName(rs.getString("name"));
    user.setPassword(rs.getString("password"));
  }

  rs.close();
  ps.close();
  c.close();

  if (user == null) throw new EmptyResultDataAccessException(1);

  return user;
}
```

#### 포괄적인 테스트
- 성공하는 케이스만 만드는 경우가 있으나, 네거티브 테스트를 하는 것이 중요하다.
- 따라서, 부정적인 케이스를 먼저 만드는 습관이 좋다.

### 테스트가 이끄는 개발
- get 메소드의 예외 테스트 생성 과정
  - 새로운 기능을 넣기 위해, UserDao 코드를 수정
  - 테스트를 먼저 만들어 테스트가 실패하는 것을 보고 나서 UserDao의 코드를 수정

#### 기능설계를 위한 테스트
- 존재하지 않는 id로 get 메소드를 실행하면 특정한 예외가 던져져야 한다는 식으로 만들어야 할 기능을 결정
- UserDao 코드를 수정하는 대신 getuserFailure() 테스트를 먼저 만듦
  - 만들 수 있었던 이유는 **추가하고 싶은 기능을 코드로 표현하려 했기 때문**

#### 테스트 주도 개발
- ***만들고자 하는 기능의 내용을 담고 있으면서 만들어진 코드를 검증해줄 수 있도록 테스트 코드를 먼저 만들고, 테스트를 성공하게 해주는 코드를 작성하는 방식의 개발 방법*** 을 **테스트 주도 개발(TDD, Test Driven Development)** 라고 한다.
- **실패한 테스트를 성공시키기 위한 목적이 아닌 코드는 만들지 않는다** 는 기본 원칙을 가진다.
  - 테스트를 먼저 만들고 진행하면, 해당 테스트가 성공하도록 하는 코드만 만드는 식이므로, 테스트를 빼먹지 않고 꼼꼼하게 진행할 수 있다.
  - 이미 테스트를 만들어 놓아서, 코드에 대한 피드백이 수월하다.
  - 코드에 확신이 생긴다.
- **테스트를 작성하고 이를 성공시키는 코드를 만드는 작업 주기를 가능한 한 짧게 가져가도록 권장**

### 테스트 코드 개선
- 기계적으로 반복되는 부분이 있음

```java
ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");

UserDao dao = context.getBean("userDao", UserDao.class);
```
- JUnit을 통해, **테스트를 실행할 때마다 반복되는 준비 작업을 별도의 메소드에 넣게 해주고, 이를 매번 실행히켜주는 기능을 적용**

#### @Before
- 중복됐던 코드를 넣을 setUp()이라는 이름의 메소드를 만들고 테스트 메소드에서 제거한 코드를 넣는다.
- dao 변수가 setUp() 메소드의 로컬 변수이니, 인스턴스 변수로 변경한다.
- **@Before** 에너테이션을 적용한다.

```java
@Before
public void setUp(){
  ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
  this.dao = context.getBean("userDao", UserDao.class);
}
```
***JUnit5 소스 변경***
```java
@BeforeEach
public void setUp(){
  ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
  this.dao = context.getBean("userDao", UserDao.class);
}
```
```java

@SpringBootTest(classes=com.example.demo.user.dao.UserDao.class)
public class UserDaoTest {
	private UserDao dao;

	@BeforeEach
	public void setUp(){
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
		this.dao = context.getBean("userDao", UserDao.class);
	}

	@Test
	public void addAndGet() throws SQLException, ClassNotFoundException {
		User user1 = new User("gyumee", "박성철", "springno1");
		User user2 = new User("leegw700", "이길원", "springno2");

		dao.deleteAll();
		assertEquals(dao.getCount(), 0);

		dao.add(user1);
		dao.add(user2);
		assertEquals(dao.getCount(), 2);

		User userget1 = dao.get(user1.getId());
		assertEquals(userget1.getName(), user1.getName());
		assertEquals(userget1.getPassword(), user1.getPassword());

		User userget2 = dao.get(user2.getId());
		assertEquals(userget2.getName(), user2.getName());
		assertEquals(userget2.getPassword(), user2.getPassword());
	}

	@Test
	public void count() throws SQLException, ClassNotFoundException {
		User user1 = new User("gyumee", "박성철", "springno1");
		User user2 = new User("leegw700", "이길원", "springno2");
		User user3 = new User("bumjin", "박범진", "springno3");

		dao.deleteAll();
		assertEquals(dao.getCount(), 0);

		dao.add(user1);
		assertEquals(dao.getCount(), 1);

		dao.add(user2);
		assertEquals(dao.getCount(), 2);

		dao.add(user3);
		assertEquals(dao.getCount(), 3);
	}

	@Test
	public void getUserFailure() throws SQLException, ClassNotFoundException {
		dao.deleteAll();
		assertEquals(dao.getCount(), 0);
		assertThrows(EmptyResultDataAccessException.class, () -> {
			dao.get("unknown_id");
		});
	}
}
```

- **JUnit이 하나의 테스트 클래스를 가져와 테스트를 수행하는 방식**
  - 테스트 클래스에서 @Test가 붙은 pubilc이고 void형이며 파라미터가 없는 테스트 메소드를 모두 찾는다
  - 테스트 클래스의 오브젝트를 하나 만든다
  - @Before가 붙은 메소드가 있으면 실행한다
  - @Test가 붙은 메소드를 하나 호출하고 테스트 결과를 저장해둔다
  - @After가 붙은 메소드가 있으면 실행한다.
  - 나머지 테스트 메소드에 대해 2~5번을 반복한다
  - 모든 테스트의 결과를 종합해서 돌려준다

#### 픽스처
- ***테스트를 수행하는 데 필요한 정보나 오브젝트*** 를 **픽스처(fixture)** 라고 한다.
  - UserDaoTest에서 dao가 대표적인 픽스처로, 여러 테스트에서 반복적으로 사용하여 @Before 메소드를 통해 생성해두면 좋다
  - User 오브젝트들은 UserDao의 기능이 계속 만들어지고 그에 따라 테스트 메소드로 계속 추가될 텐데, UserDao에 대한 테스트라면 대부분 User 오브젝트를 사용할 것이기 때문에 픽스처로 설정하는 것이 좋다.

```java
@SpringBootTest(classes=com.example.demo.user.dao.UserDao.class)
public class UserDaoTest {
	private UserDao dao;
	private User user1;
	private User user2;
	private User user3;

	@BeforeEach
	public void setUp(){
		ApplicationContext context = new GenericXmlApplicationContext("applicationContext.xml");
		this.dao = context.getBean("userDao", UserDao.class);
	}

	@Test
	public void addAndGet() throws SQLException, ClassNotFoundException {
		this.user1 = new User("gyumee", "박성철", "springno1");
		this.user2 = new User("leegw700", "이길원", "springno2");

		dao.deleteAll();
		assertEquals(dao.getCount(), 0);

		dao.add(user1);
		dao.add(user2);
		assertEquals(dao.getCount(), 2);

		User userget1 = dao.get(user1.getId());
		assertEquals(userget1.getName(), user1.getName());
		assertEquals(userget1.getPassword(), user1.getPassword());

		User userget2 = dao.get(user2.getId());
		assertEquals(userget2.getName(), user2.getName());
		assertEquals(userget2.getPassword(), user2.getPassword());
	}

	@Test
	public void count() throws SQLException, ClassNotFoundException {
		this.user1 = new User("gyumee", "박성철", "springno1");
		this.user2 = new User("leegw700", "이길원", "springno2");
		this.user3 = new User("bumjin", "박범진", "springno3");

		dao.deleteAll();
		assertEquals(dao.getCount(), 0);

		dao.add(user1);
		assertEquals(dao.getCount(), 1);

		dao.add(user2);
		assertEquals(dao.getCount(), 2);

		dao.add(user3);
		assertEquals(dao.getCount(), 3);
	}

	@Test
	public void getUserFailure() throws SQLException, ClassNotFoundException {
		dao.deleteAll();
		assertEquals(dao.getCount(), 0);
		assertThrows(EmptyResultDataAccessException.class, () -> {
			dao.get("unknown_id");
		});
	}
}
```

## 스프링 테스트 적용
- 아직 찜찜한 곳은 애플리케이션 컨텍스트 생성 방식이다.
  - @Before 메소드가 테스트 메소드 개수만큼 반복되므로, 애플리케이션 컨텍스트도 3번 만들어진다.
- 현재는 이상 없으나, 테스트가 많아질 수록 시간이 걸릴 것이다.
  - **애플리케이션 컨텍스트가 만들어질 때는 모든 싱글톤 빈 오브젝트가 초기화된다**
    - 어떤 빈은 오브젝트가 생성될 때 자체적인 초기화 작업을 진행해서 많은 시간을 필요로 할 때가 있기 때문
### 테스트를 위한 애플리케이션 컨텍스트 관리
#### 스프링 테스트 컨텍스트 프레임워크 적용
```java
@RunWith(SpringJUnit4ClassRunner)
@ContextConfiguration(locations="/applicationContext.xml")
public class UserDaoTest {
	@Autowired
	private ApplicationContext context;

	private UserDao dao;
	private User user1;
	private User user2;
	private User user3;

	@BeforeEach
	public void setUp(){
		this.dao = this.context.getBean("userDao", UserDao.class);
	}
  ...
```
***JUnit5 소스 변경***

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(locations="/applicationContext.xml")
public class UserDaoTest {
	@Autowired
	private ApplicationContext context;

	private UserDao dao;
	private User user1;
	private User user2;
	private User user3;

	@BeforeEach
	public void setUp(){
		this.dao = this.context.getBean("userDao", UserDao.class);
	}
  ...
```
- **@RunWith** 는 JUnit 프레임워크의 테스트 실행 방법을 확장할 때 사용
- **@ContextConfiguration** 은 자동으로 만들어줄 애플리케이션 컨텍스트의 설정파일 위치를 지정

#### 테스트 메소드의 컨텍스트 공유
- 어러 개의 테스트 클래스가 있는데, 모두 같은 설정파일을 가진 애플리케이션 컨텍스트를 사용한다면, 스프링은 테스트 클래스 사이에도 애플리케이션 컨텍스트를 공유하게 해준다.

#### @Autowired
- **@Autowired** 는 ***스프링의 DI에 사용되는 특별한 애노테이션***
- @Autowired가 붙은 인스턴스 변수가 있으면, 테스트 컨텍스트 프레임워크는 변수 타입과 일치하는 컨텍스트 내의 빈을 찾는다.
- 타입이 일치하는 빈이 있으면 인스턴스 변수에 주입해준다.
- 일반적으로는 주입을 위해서는 생성자나 수정자 메소드 같은 메소드가 필요하지만, 이 경우에는 메소드가 없어도 주입이 가능하다.
- 별도의 DI 설정 없이 필드의 타입정보를 이용해 빈을 자동으로 가져올 수 있는데, 이런 방법을 타입에 의한 **자동와이어링** 이라고 한다.
- @Autowired를 이용해 애플리케이션 컨텍스트가 갖고 있는 빈을 DI 받을 수 있다면 굳이 컨텍스트를 가져와 getBean()을 사용하는 것이 아니라, 아예 UserDao 빈을 직접 DI 받을 수 있다.
```java
public class UserDaoTest {
  ...
  @Autowired
  private UserDao dao;
  ...
```
- @Autowired는 같은 타입의 빈이 두 개 이상 있는 경우에는 타입만으로는 어떤 빈을 가져올지 결정할 수 없다.

### DI와 테스트
- 절대 사용하지 않는다는 DI 주입방식이더라도 인터페이스를 사용해야 하는 이유
  - 소프트웨어 개발에서 절대란 없다.
  - 클래스의 구현 방식은 바뀌지 않는다고 하더라도, 인터페이스를 두고 DI를 적용해 두면, 다른 차원의 서비스 기능을 도입할 수 있다.
    - DB 커넥션 카운트 등
  - 테스트
    - 효율적인 테스트를 손쉽게 만들기 위해서라도 DI를 적용해야 한다.
    - DI는 테스트가 작은 단위의 대상에 대해 독립적으로 만들어지고 실행되게 하는 중요한 역할
  
  #### 테스트 코드에 의한 DI
  - 테스트 코드 내에서 수정자 메소드를 이용해서 직접 DI해도 된다.
    - UserDao가 사용할 DataSource 오브젝트를 테스트 코드에서 변경할 수 있다는 뜻
  - DB 커넥션이 운영버전이라면, 테스트 할 때는 사용해서는 안된다.
    - applicationContext.xml 설정을 바꾸어 설정할 수 있지만, 위험하니 테스트 중에 DAO가 사용할 DataSource 오브젝트를 바꿔주는 게 좋다.
  
```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(locations="/applicationContext.xml")
@DirtiesContext
public class UserDaoTest {
  @Autowired
  private UserDao dao;
  private User user1;
  private User user2;
  private User user3;

  @BeforeEach
  public void setUp(){
    DataSource dataSource = new SingleConnectionDataSource("jdbc:mysql://localhost/testdb", "spring", "book", true);
    this.dao.setDataSource(dataSource);
  }
```
- 위 방법은 의존관계가 변경될 수 있어, **@DirtiesContext** 애노테이션으로, 테스트 컨텍스트 프레임워크에게 해당 클래스의 테스트에서 애플리케이션 컨텍스트의 상태를 변경한다는 것을 알려준다.
- 테스트 컨텍스트는 위 애노테이션이 붙은 테스트 클래스에는 애플리케이션 컨텍스트 공유를 허용하지 않는다.
- 다만 매번 애플리케이션 컨텍스트를 매번 만드는 것은 좋지 못하다.

#### 테스트를 위한 별도의 DI 설정
- 매번 수동으로 DI하는 건 단점이 많다.
- 두 가지 종류의 설정파일을 만들어서 서버에서 운영용/테스트용으로 사용할 DataSource를 빈으로 각각 등록해두는 것
- test-applicationContext.xml을 만들어 설정을 변경
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="dataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
        <property name="driverClass" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost/testDB"/>
        <property name="username" value="spring"/>
        <property name="password" value="book"/>
    </bean>

    <bean id="userDao" class="com.example.demo.user.dao.UserDao">
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```
```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(locations="/test-applicationContext.xml")
@DirtiesContext
public class UserDaoTest {
...
```

#### 컨테이너 없는 DI 테스트
- 아예 스프링 컨테이너를 사용하지 않는 방법
- UserDao나 DataSource 구현 클래스 어디에도 스프링 API를 직접 사용하거나, 애플리케이션 컨텍스트를 이용하는 코드가 없다.
  - 스프링 DI 컨테이너에 의존하지 않는다.

```java
public class UserDaoTest {
  private UserDao dao;
  private User user1;
  private User user2;
  private User user3;

  @BeforeEach
  public void setUp(){
    dao = new UserDao();
    DataSource dataSource = new SingleConnectionDataSource("jdbc:mysql://localhost/testdb", "spring", "book", true);
    this.dao.setDataSource(dataSource);
  }
  ...
```

- 매번 새로운 UserDao 오브젝트가 만들어지는 단점은 있으나, UserDao는 가벼운 오브젝트라 부담이 없다.

#### DI를 이용한 테스트 방법 선택
- 항상 스프링 컨테이너 없이 테스트 할 수 있는 방법을 먼저 생각한다.
- 오브젝트와 복잡한 의존관계를 갖고 있는 오브젝트를 테스트해야할 경우, 스프링의 설정을 이용한 DI 방식의 테스트를 이용하면 편리하다.
- 예외적인 케이스에서 의존관계를 강제로 설정해준다.

## 학습 테스트로 배우는 스프링
- 테스트는 본인의 코드 뿐 아니라, 자신이 만들지 않은 프레임워크나 제공한 라이브러리에 대해서도 진행해야 한다. (**학습테스트(learning test)**)

### 학습 테스트의 장점
- 다양한 조건에 따른 기능을 손쉽게 확인해볼 수 있다.
- 학습 테스트코드를 개발 중에 참고할 수 있다
- 프레임워크나 제품을 업그레이드할 때 호환성 검증을 도와준다.
- 테스트 작성에 대한 좋은 훈련이 된다.
- 새로운 기술을 공부하는 과정이 즐거워진다.

### 버그 테스트
- **버그 테스트** 란 ***코드에 오류가 있을 때 그 오류를 가장 잘 드러내줄 수 있는 테스트***
  - 테스트의 완성도를 높여준다.
  - 버그의 내용을 명확하게 분석하게 해준다.
  - 기술적인 문제를 해결하는 데 도움이 된다

## 정리
- 테스트는 자동화돼야 하고, 빠르게 실행할 수 있어야 한다.
- main 테스트 대신 JUnit 프레임워크를 이용한 테스트 작성이 편리하다
- 테스트 결과는 일관성 있어야 한다. 코드의 변경 없이 환경이나 테스트 실행 순서에 따라서 결과가 달라지면 안된다.
- 테스트는 포괄적으로 작성해야 한다. 충분한 검증을 하지 않은 테스트는 없는 것보다 나쁠 수 있다.
- 코드 작성과 테스트 수행의 간격이 짧을수록 효과적이다.
- 테스트하기 쉬운 코드가 좋은 코드다
- 테스트를 먼저 만들고 테스트를 성공시키는 코드를 만들어가는 테스트 주도 개발 방법도 유용하다
- 테스트 코드도 애플리케이션 코드와 마찬가지로 적절한 리팩토링이 필요하다.
- @Before, @After를 사용해서 테스트 메소드들의 공통 준비 작업과 정리 작업을 처리할 수 있다.
- 스프링 테스트 컨텍스트 프레임워크를 이용하면 테스트 성능을 향상시킬 수 있다
- 동일한 설정파일을 사용하는 테스트는 하나의 애플리케이션 컨텍스트를 공유한다
- @Autowired를 사용하면 컨텍스트의 빈을 테스트 오브젝트에 DI할 수 있다.
- 기술의 사용 방법을 익히고 이해를 돕기 위해 학습 테스트를 작성하자
- 오류가 발견될 경우 그에 대한 버그 테스트를 만들어두면 유용하다.

