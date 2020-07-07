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
		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

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
		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

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
		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

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
  ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

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
  ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

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
