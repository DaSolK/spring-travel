# 1. 오브젝트와 의존관계

객체지향 프로그래밍이 제공하는 폭넓은 혜택을 누리도록 기본으로 돌아가자 → 스프링

- 오브젝트의 생성, 사용, 소멸까지의 과정
- 오브젝트를 어떻게 설계해야 하는 지

오브젝트의 관심 → 결국 오브젝트의 설계로 발전

- 객체지향 설계의 기초와 원칙
- 다양한 목적을 위해 재활용 가능한 설계 방법인 디자인 패턴
- 좀 더 깔끔한 구조가 되도록 지속적으로 개선해나가는 리팩토링
- 오브젝트가 기대한 대로 동작하고 있는 지 검증하는 데 쓰이는 단위 테스트

1장에서는 스프링 자체보다는 스프링의 관심 대상인 오브젝트의 설계와 구현, 동작원리에 집중하자

## 1.1. 초난감 DAO

사용자 정보를 JDBC API를 통해 DB에 저장하고 조회할 수 있는 간단한 DAO

> DAO(Data Access Object)는 DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트

### 1.1.1. User

자바빈 규약을 따르는 오브젝트를 이용한 사용자 정보 저장 User 클래스

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

> 다음 두 가지 관례를 따라 만들어진 오브젝트
- 디폴트 생성자: 파라미터가 없는 디폴트 생성자 필요, 툴이나 프레임워크에서 리플렉션을 이용해 오브젝트를 생성하기 때문
- 프로퍼티: 자바빈이 노출하는 이름을 가진 속성을 프로퍼티라고 함, 프로퍼티는 set으로 시작하는 setter와 get으로 시작하는 getter를 이용해 수정 또는 조회

### 1.1.2. UserDao

사용자 정보를 DB에 넣고 관리할 수 있는 DAO 클래스인 UserDao를 만들자. 등록, 수정, 삭제와 각종 조회 기능이 있어야겠지만, 우선은 새로운 사용자 생성(add)와 아이디를 가지고 읽어오는 get 두 개로 시작

```java
public class UserDao {
  public void add(User user) throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager
     .getConnection("jdbc:mysql://localhost/springbook", "spring", "book");
    
    PreparedStatement ps = 
      c.prepareStatement("insert into users(id, name, password) values (?,?,?)");
    ps.setString(1, user.getId());
    ps.setString(2, user.getName());
    ps.setString(3, user.getPassword());
    
    ps.executeUpdate();
    
    ps.close();
    c.close();
  }

  public User get(String id) throws ClassNotFoundException, SQLException {
    Class.forName("com.mysql.jdbc.Driver");
    Connection c = DriverManager
     .getConnection("jdbc:mysql://localhost/springbook", "spring", "book");
    
    PreparedStatement ps = c.prepareStatement("select * from users where id = ?");
    
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

그래서 이 클래스가 제대로 동작하는 지 어떻게 확인?

### 1.1.3. main()을 이용한 DAO 테스트 코드

main()에서 User를 DB에 add하고 다시 get으로 가져와서 검증하는 방식 → 초난감 DAO!

그래서 지금 UserDao 클래스의 문제가 무엇일까? 기능적으로는 동작하는 것을 우리가 확인했음, 잘 동작하는 코드를 굳이 수정하고 개선해야 하는 이유는? 개선했을 때의 장점은?

이런 문제 제기와 의문에 대한 답을 찾아나가는 과정이 스프링을 공부하는 것

## 1.2. DAO의 분리

### 1.2.1. 관심사의 분리

소프트웨어 개발에서 끝은 없다 → 사용자의 비지니스 프로세스와 요구사항은 끊임없이 바뀜

따라서 객체를 설계할 때 가장 염두에 둘 것은 미래의 변화를 어떻게 대비할 것인가임

객체지향 기술은 가상의 추상세계를 효과적으로 구성하므로 편리하게 변경, 발전, 확장시킬 수 있다

기능 변경 시 몇 줄의 코드만 수정하고 그 변경이 나머지 기능에 문제를 일으키지 않는다는 것까지 5분 걸린 개발자와 5시간 걸리면서 다른 기능에 오류를 일으킬지도 모른다는 불안감까지 만드는 개발자 중 누가 더 미래 변화를 잘 준비한 것일까? 당연히 전자임 → **분리**와 **확장**을 고려한 설계

변화는 대체로 한 가지 관심에 대해 일어나므로, 우리는 한 가지 관심이 한 군데에 집중하게 하면 됨. 즉 관심이 같은 것끼리 모으고, 관심이 다른 것은 따로 떨어져 있게

다시 말해서 DB 접속용 암호를 변경하는데 DAO 수백 개를 모두 수정한다던지, 다른 개발자가 개발한 코드 변경이 일어날때 마다 내가 만든 클래스도 수정하고 싶지 않은 것이 자명하니까

### 1.2.2. 커넥션 만들기의 추출

UserDao의 `add()` 메소드 하나에서 세 가지 관심 사항 발견됨

1. DB와 연결을 위한 커넥션을 어떻게 가져올까에 대한 관심
2. 사용자 등록을 위해 DB에 보낼 SQL 문장을 담을 Statement를 만들고 실행하는 관심
3. 작업이 끝난 후 사용한 리소스인 Statement와 Connection 오브젝트를 닫는 관심

당장 1번 DB 커넥션 부분이 `get()` 메소드에도 중복으로 있는 것이 보임

따라서 DB 연결 코드를 `getConnection()` 과 같이 분리하면 차후에 해당 메소드의 코드만 수정하면 됨

그래서 변경사항에 대한 검증은 어떻게? → 리팩토링과 테스트

앞에서 만든 main() 메소드 활용해서 테스트해보면 됨 → 이 작업은 UserDao 기능에는 영향 주지 않으면서 코드의 구조만 변경한다, 이런 작업을 리팩토링이라고 함

> 리팩토링은 기존의 코드를 외부의 동작방식에는 변화 없이 내부 구조를 변경해서 재구성하는 작업 또는 기술을 말함

### 1.2.3. DB 커넥션 만들기의 독립

서로 다른 유즈케이스에서 서로 다른 DB를 사용하고 싶다면? getConnection() 메소드를 수정하지 않고 UserDao를 사용하게 할 수 있을까?

**상속을 통한 확장**

UserDao에서 메소드의 구현 코드를 제거하고 `getConnection()` 을 추상 메소드로 만들어놓는다

이러면 실제 유즈케이스에서 해당 클래스 상속 후 원하는 대로 구현하면 됨

이것을 통해 어떻게 데이터를 등록하고 가져올 것인가에 대한 관심을 담당하는 UserDao와 DB 연결 방법은 어떻게 할 것인가라는 관심을 담고있는 ChildUserDao가 분리되었고, 손쉽게 확장된다라고 표현 가능해진다

이런 길고 긴 설명이 결국 디자인 패턴 중 일부인 템플릿 메소드, 팩토리 메소드 패턴 등 간결하게 표현가능

> 템플릿 메소드 패턴
상속을 통해 슈퍼클래스의 기능을 확장할 때 사용하는 대표적인 방법
변하지 않는 기능은 슈퍼클래스에 만들어두고, 자주 변경되며 확장할 기능은 서브클래스에서 만들도록 한다
추상 메소드와 훅 메소드 (선택적으로 오버라이드)를 활용해 템플릿 메소드만듬

> 팩토리 메소드 패턴
템플릿 메소드와 비슷하게 상속으로 기능 확장, 따라서 구조도 비슷
슈퍼클래스 코드에서는 서브클래스에서 구현할 메소드를 호출해서 필요한 타입의 오브젝트를 가져와서 사용
주로 인터페이스 타입으로 오브젝트를 리턴하므로 서브클래스에서 정확히 어떤 클래스의 오브젝트를 만들어 리턴할 지는 슈퍼클래스에서 모름

상속을 통한 확장의 한계점

- 만약 이미 UserDao가 다른 목적으로 상속한다면? → 자바는 다중상속 허용 X
- 상속을 통한 상하위 클래스의 관계는 생각보다 밀점, 서브클래스는 슈퍼클래스의 기능을 직접사용가능하므로 슈퍼클래스 내부 변경이 생기면 서브클래스도 함께 수정해야할 수 있음
- 확장된 기능인 DB 커넥션 생성코드를 다른 DAO에 적용할 수 없는 단점 → 즉 DAO가 계속 만들어지면 getConnection 구현 코드가 중복되서 나타나게 됨

## 1.3. DAO의 확장

관심사에 따라 분리한 오브젝트들은 제각기 독특한 변화의 특징이 있음

지금까지는 데이터 액세스 로직을 어떻게 만들 것인가와 DB 연결을 어떤 방법으로 할 것인가라는 두 개의 관심을 상하위 클래스로 분리했음

변화의 성격이 다르다 → 변화의 이유, 시기, 주기 등이 다르다는 뜻

### 1.3.1. 클래스의 분리

지금까지 관심사의 분리는 독립된 메소드를 만들어서 분리 → 상하위 클래스로 분리

이런 점진적 변화 말고, 아예 별도의 클래스에 담고 UserDao가 사용하게 하자

다 좋은데 이전처럼 상속만해서 UserDao 코드 수정 없이 DB 커넥션 생성 기능 변경 방법이 없어짐

따라서 클래스를 분리한 경우에도 상속을 이용했을 때와 마찬가지로 자유로운 확장이 가능하게 하려면 다음 두 가지 문제를 해결해야 함

1. SimpleConnectionMaker의 메소드 문제, 특정 유즈케이스에서 makeNewConnection()이 아니라 openConnection이라는 메소드 이름을 사용하면 add(), get() 내의 코드를 모두 변경해야함
2. DB 커넥션을 제공하는 클래스가 어떤 것인지 UserDao가 구체적으로 알고 있어야 함, 즉 SimpleConnectionMaker라는 인스턴스 변수까지 정의하고 있으므로 다른 클래스를 구현하면 UserDao 자체를 수정해야 함

이런 문제의 근본적인 원인은 UserDao가 바뀔 수 있는 정보, 즉 DB 커넥션을 가져오는 클래스에 대해 너무 많이 알고 있음 → 어떤 클래스가 쓰일지, 그 클래스에서 커넥션을 가져오는 메소드는 이름이 무엇인지 등등

다시 말해서 UserDao는 DB 커넥션을 가져오는 구체적인 방법에 종속되어 버림

### 1.3.2. 인터페이스의 도입

가장 좋은 해결책은 두 개의 클래스가 서로 긴밀하게 연결되지 않도록 중간에 추상적인 연결고리를 만들어 주는 것 → 추상화를 위해 제공하는 가장 유용한 도구인 인터페이스 활용, 인터페이스를 통해 접근하게 되면 구현클래스를 바꿔도 신경 쓸 일이 없다

인터페이스는 어떤 일을 하겠다는 기능만 정의해놓은 것

ConnectionMaker 인터페이스를 만들면 makeConnection 메소드를 쓴다는 것이 자명하고 각자의 유즈케이스에 맞게 해당 인터페이스 구현을 하면 됨

→ 이렇게 해도 UserDao에서 `connectionMaker = new DConnectionMaker();` 부분이 있음

이 부분 대문에 여전히 UserDao 수정이 생김.. 다시 원점

### 1.3.3. 관계설정 책임의 분리

이 문제가 발생하는 것은 UserDao가 여전히 어떤 ConnectionMaker 구현 클래스를 사용할지에 대한 관심이 남아있기 때문임, 즉 UserDao와 UserDao가 사용할 ConnectionMaker의 특정 구현 클래스 사이의 관계를 설정해주는 것에 관한 관심 → 이걸 분리하지 않으면 독립적으로 확장 불가능

한 오브젝트가 다른 오브젝트의 기능을 이용한다면 사용되는 쪽이 사용하는 쪽에 서비스를 제공 → 사용되는 오브젝트를 서비스, 사용하는 오브젝트를 클라이언트

UserDao의 클라이언트에서 UserDao를 사용하기 전에 어떤 ConnectionMaker 구현 클래스를 사용할지를 결정하게 하면 됨 → 외부에서 생성자 파라미터나 오브젝트 파라미터 등으로 넘겨도 된다

인터페이스를 도입하고 클라이언트의 도움을 받는 방법은 상속보다는 훨씬 유연함

ConnectionMaker를 사용하기만 하면 다른 DAO 클래스에서도 그대로 적용할 수 있고, DB 접속 방법에 대한 관심은 한군데에 집중되며 변경할 때도 한 군데만 변경하면 됨

### 1.3.4. 원칙과 패턴

당장 객체지향 기술 이론 용어가 친숙하지 않아도 당황하지 말자

**개방 폐쇄 원칙**

클래스나 모듈은 확장에는 열려있어야 하고 변경에는 닫혀있어야 한다

UserDao는 DB 연결 방법이라는 기능 확장에 열려있음, UserDao에 영향을 주지 않아도 기능 확장 가능, 그리고 그 자신은 변화에 영향 받지 않으므로 변경에는 닫혀있음

**높은 응집도와 낮은 결합도**

하나의 모듈, 클래스가 하나의 책임 또는 관심사에 집중되어있다 → 높은 응집도

책임과 관심사가 다른 오브젝트 또는 모듈과는 느슨하게 연결된 형태를 유지한다 → 낮은 결합도

**전략 패턴**

자신의 Context에서 필요에 따라 변경이 필요한 알고리즘은 인터페이스로 통째로 외부 분리, 이를 구현한 구체적인 알고리즘 클래스를 필요에 따라 바꿔서 사용하는 디자인 패턴