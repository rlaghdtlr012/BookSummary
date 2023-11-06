## 7. 스프링 삼각형과 설정 정보

    - 스프링을 이해하는 데는 POJO(Plain Old Java Object)를 기반으로 스프링 삼각형이라는 애칭을 가진 IOC/DI, AOP, PSA라고 하는 스프링의 3대 프로그래밍 모델에 대한 이해가 필수

#### 7-1. IOC/DI - 제어의 역전/의존성 주입

    - 자바에서의 의존성이란?
        - 의사코드 예시
            - 운전자가 자동차를 생산한다.
            - 자동차는 내부적으로 타이어를 생산한다.
        - 자바로 표현
            - new Car();
            - Car 객체 생성자에서 new Tire();
        - 의존성은 new에 해당한다.
        - new를 실행하는 Car와 Tire 사이에서 "Car가 Tire를 의존한다"
        라고 할 수 있다.

    - 전체(Car)가 부분(Tire) 의존한다는 것 == 의존성

    - 의존 관계는 new로 표현된다

##### \* 스프링 없이 의존성 주입하기 1 : 생성자를 통한 의존성 주입

```java
Tire tire = new KoreaTire();
Car car = new Car(tire); // 생성자를 사용한 의존성의 주입
```

- 자동차 내부에서 타이어를 생산하는 것이 아니라 외부에서 생산된 타이어를 자동차에 장착하는 작업 -> 주입
- 생성자를 통한 의존성 주입의 장점 - 자동차가 생산될 때 어떤 타이어를 생산해서 장착할까를 자동차가 스스로 고민하게 만드는 것(유연성이 떨어짐)이 아닌, 운전자가 차량을 생산할 때, "운전자"가 어떤 타이어를 장착할까를 고민하게 하는 것. - 즉, 자동차는 어떤 타이어를 장착할까를 고민하지 않아도 됨 - 의존성 주입을 적용할 경우, Car는 그저 Tire "인터페이스를 구현한 어떤 객체"가 들어오기만 하면 정상적으로 작동한다. -> 확장성이 좋아진다.
  <br><br>

##### \* 스프링 없이 의존성 주입하기 2 : 속성을 통한 의존성 주입

```java
Tire tire = new KoreaTire();
Car car = new Car();
car.setTire(tire) // 속성 접근자 메서드를 사용한 의존성 주입
```

- 현실 세계에 비유해 봤을 때, 앞서 생성자를 통해 의존성을 주입한다면 자동차를 생산(구입)할 때 한 번 타이어를 장착하면 더 이상 타이어를 교체 장착할 방법이 없다는 문제가 생김.
- 운전자가 원할 때, Car의 Tire를 교체하는 것 -> 속성을 통한 의존성 주입
- 최근에는 속성을 통한 의존성 주입보다는 생성자를 통한 의존성 주입을 더 선호. 실제 프로그램에서는 한 번 주입된 의존성을 계속 사용하는 경우가 일반적이기 때문에
  <br><br>

##### \* 스프링을 통한 의존성 주입 - 스프링 설정 파일(XML)에서 속성 주입

- 의사 코드 - 현실 세계와 비슷한 설걔

  - 운전자가 종합 쇼핑몰(XML)에서 자동차를 구매 요쳥한다.
  - 종합 쇼핑몰은 자동차를 생산한다.
  - 종합 쇼핑몰은 타이어를 생산한다.
  - 종합 쇼핑몰은 자동차에 타이어를 장착한다.
  - 종합 쇼핑몰은 운전자에게 자동차를 전달한다.

- 자바 코드

```java
ApplicationContext context = new ClassPathXmlApplicationContext("expery003/expert003.xml");
Car car = context.getBean("car", Car.class);
```

- XML로 표현

```bean
<bean id="koreaTitle" class="expert003.KoreaTire"></bean>
<bean id="americaTitle" class="expert003.AmericaTitle"></bean>
<bean id="car" class="expert003.Car">
    <property name="tire" ref="koreaTire"></property>
</bean>
```

```java
Car car = context.getBean("car", Car.class);
// bean의 property에 koreaTire를 넣어서 의존성을 주입해줬기 때문에 아래 코드는 생략이 가능하다.
// Tire tire = context.getBean("tire", Tire.class);
//  car.setTire(tire);
```

- property -> 자바에서 car.setTire(tire)와 같은 기능을 하는 코드
- car bean에다가 tire라는 property를 넣어줬기 때문애 koreaTire를 자동차의 타이어 속성에 결합이 가능하다.
  <br><br>

##### \* 스프링을 통한 의존성 주입 - @Autowired를 통한 속성 주입

- 의사 코드

  - 운전자가 종합 쇼칭몰에서 자동차를 구매 요청한다.
  - 종합 쇼핑몰은 자동차를 생산한다.
  - 종합 쇼핑몰은 타이어를 생산한다.
  - 종합 쇼핑몰은 자동차와 타이어를 장착한다.
  - 종합 쇼핑몰은 운전자에게 자동차를 전달한다.

```java
@Autowired
Tire tire;
```

- @Autowired 애노테이션을 이용하면 설정자 메서드를 이용하지 않고도, 스프링 프레임워크가 설정 파일을 통해 설정자 메서드 대신 속성을 주입해준다.

- @Autowired란 스프링 설정 파일을 보고 자동으로 속성의 설장자 메서드에 해당하는 역할을 해주겠다는 의미.

- 기존 XML 설정 파일

```
<bean id="car" class="expert003.Car">
    <property name="tire" ref="koreaTire"></property>
</bean>
```

- @Autowired를 이용한 새로운 XML 설정 파일

```
<bean id="car" class="expert004.Car"></bean>
// @Autowired를 통해 car의 property를 자동으로 엮어줄 수 있으므로(자동 의존성 주입) property의 생략이 가능해진다.
```

- Class.java 파일에서 @Autowired 지덩된 속성과, xml 파일에서 bean의 id 속성이 일치하는 것을 찾아 매치시킨다. 하지만 id 값이 없어도 bean이 한 개라면 스프링은 아래와 같은 알고리즘으로 속성을 매칭시켜준다.

![image](<../assets/Autowired(1).jpeg>) - @Autowired를 통한 속성 매칭 규칙

- 만약에 bean이 여러 개인데, id 값도 없다면, 에러가 발생

- 단, bean이 여러 개라도, id 값이 제대로 매핑되어 있다면, 의존성을 잘 주입시켜준다. -> 따라서 항상 bean 태그의 id 속성을 작성하는 습관을 들이자.

- @Autowired는 id 매칭보다 type 매칭이 우선이다. type과 id 중 type 구현에 우선순위가 있음.
  <br><br>

##### \* 스프링을 통한 의존성 주입 - @Resource를 통한 속성 주입

    - 의사 코드: 이전과 동일
    - @Autowired와 동일한데, 대신 Autowired라는 표현 대신 Resource라고 변경된다.

```java
public class Car {
    // 이전 코드
    // @Autowired
    // Tire tire;

    // 바뀐 코드
    @Resource
    Tire tire;

    public String getTireBrand() {
        return "장착된 타이어: " tire.getBrand();
    }
}
```

- 그렇다면 왜 굳이 @Autowired를 @Resource로 변경한걸까?

  - @Autowired는 스프링의 어노테이션이고, @Resource는 자바 표준 어노테이션.
  - 따라서 Spring을 사용하지 않는다면 오직 @Resource만 사용할 수 있음
  - @Resource는 type과 id 중, id의 매칭 우선순위가 더 높다.
  - @Resource는 id로 매칭할 빈을 찾지 못한 경우, type으로 매칭할 빈을 찾음
  - 허나 @Resource는 bean 태그의 자식 태그인 <property> 태그로 대체될 수 있다.
  - 그렇다면 <property> vs @Resource 중에서는 뭐가 더 나을까?
    - <property>: XML 파일만 봐도 DI 관계를 쉽게 확인할 수 있다. 유지보수성이 좋다. 프로젝트의 규모가 커지면 XML 파일의 규모가 커지기 마련인데, XML 파일도 용도별로 분리할 수 있으므로 <property> 태그가 더 유리하다.
    - @Resource: 개발 생산성이 더 낫다.
    - 설정 파일을 쓰는 이유: 재컴파일, 재배포 없이 프로그램의 실행 결과를 변경할 수 있기 때문.

- DI: 외부에 있는 의존 대상을 주입하는 것을 말한다.
  <br>

#### 7-2. AOP - Aspect? 관점? 핵심 관심사? 횡단 관심사?

    - AOP(Aspect-Oriented Programming): 관점 지향 프로그래밍

    - DI가 의존성(new)에 대한 주입이라면 *AOP는 로직(code) 주입*을 의미함

![image](<../assets/AOP(1).jpeg>) - 입금, 출금, 이체 모듈에서 반복적으로 나타나는 로깅, 보안, 트랜잭션 기능. 이렇게 여러 모듈에서 공통적으로 나타나는 부분을 *횡단 관심사*라고 한다.

```java
// DB 커넥션 코드
try {
    // DB 커넥션 연결
    // statement 객체 세팅
    insert / update / delete / select 구문
} catch (Exception e) {
    예외처리
} finally {
    DB 자원 반납
}
// 이 코드에서 insert / update / delete / select 부분을 제외한 부분은
// 어떤 DB 연산을 하든 공통적으로 나오는 코드 -> 횡단 관심사
// 그리고 insert / update / delete / select -> 핵심 관심사
```

    - 핵심 관심사: 모듈 별로 다름
    - 횡단 관심사: 모듈 별로 반복되어 중복해서 나타남

    - AOP -> 로직(코드)의 주입. 어디에? -> 메서드 안에. 그렇다면 메서드 안에 로직을 주입할 수 있는 곳은? 크게 5군데
    -> 1. Around: 메서드 전 구역
    -> 2. Before: 메서드 시작 전
    -> 3. After: 메서드 종료 후
    -> 4. After Returning: 메서드 정상 종료 후
    -> 5. After Throwing: 메서드에서 예외가 발생하면서 종료된 후

    - 횡단 관심사를 처리하기 위해 별도의 class(java 파일)를 만들어 횡단 관심사를 처리한다.

    - 빈이 설정되는 이유: 객체의 생성과 의존성 주임을 스프링 프레임워크에 위임하기 위해서

    - 스프링 AOP는 인터페이스 기반이다
    - 스프링 AOP는 프록시 기반이다
    - 스프링 AOP는 런타임 기반이다.

- AOP의 기본적인 용어
  1. Pointcut: 자르는 점
     - Aspect 적용 위치 지정자
     - 횡단 관심사를 적용할 타깃 메서드를 선택하는 지시자.(= 메서드 선택 필터)
     - 즉, 타깃 클래스의 타깃 메서드 지정자(= 메서드 선정 알고리즘)
  2. JoinPoint: 연결 가능한 지점
     - PointCut은 Joinpoint의 부분 집합
     - PointCut의 후보가 되는 모든 메서드들
     - Aspect 적용이 가능한 모든 지점
     - 스프링 AOP에서 JoinPoint란, 스프링 프레임워크가 관리하는 빈의 모든 메서드
     - 넓은 의미의 JoinPoint: Aspect 적용이 가능한 모든 지점
     - 좁은 의미의 Joinpoint: 호출된 객체의 메서드
  3. Advice: 조언? 언제, 무엇을!
     - pointcut에 적용할 로직, 즉 메서드를 의미 + 언제라는 개념까지 포함
     - 즉, pointcut에 언제, 무엇을 적용할지 정의한 메서드다. 타깃 메서드에 적용될 부가 기능
  4. Aspect: Advisor의 집합체
     - 여러 개의 Advice와 여러 개의 Pointcut의 결합체를 의미
     - Aspect - Advices(언제, 무엇을) + Pointcuts(어디에)
     - 즉 Aspect = 언제, 어디에, 무엇을?
  5. Advisor: 조언자. 어디서, 언제 무엇을!
     - Advisor = 한 개의 Advice + 한 개의 Pointcut
     - 여러 Advices + Pointcuts를 아우르는 Aspect라는 개념이 나왔기 때문에 Advisor는 사라지는 추세

```java
@Aspect
public class MyAspect {
    // runSomething() -> Pointcut
    // @Before -> 아래의 메서드(public void before)를 runSomething()이 실행되기 전에 실행하라는 의미 -> 이게 곧 Advice
    @Before("execution(runSomething())")
    public void before(JoinPoint joinPoint) {
        // 위 매개변수의 JoinPoint의 실체는? -> 그때 그때 다르지만 객체.메서드를 호출했을 때, JointPoint는 해당 객체의 메서드가 된다.
        // JointPoint 파라미터를 사용하여 실행 시점에 실제 호출된 메서드가 무엇인지, 실제 호출된 메서드를 소유학 객체가 무엇인지, 호출된 메서드의 파라미터는 무엇인지 등의 정보를 확인할 수 있다.
        System.out.println("얼굴 인식 확인: 문을 개방하라");
    }
}
```

#### 7-3. PSA - 일관성 있는 서비스 추상화

    - Portable Service Abstraction. 일관성 있는 서비스 추상화

    - 서비스 추상화의 예: JDBC

    - 다수의 기술을 공통의 인터페이스로 제어할 수 있게 한 것 = 서비스 추상화

    - 스프링은 OXM, ORM, 캐시, 트랜잭션 등 다양한 기술에 대한 PSA 지원.

<br><br>
