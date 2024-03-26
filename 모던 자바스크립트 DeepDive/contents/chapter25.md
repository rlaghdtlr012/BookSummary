# 25. 클래스

### 25.1 클래스는 프로토타입의 문법적 설탕인가?

- 자바스크립트는 프로토타입 기반 객체지향 언어.
- 기존 ES5까지는 클래스 없이 프로토타입으로 OOP 구현.
- ES6에 클래스 도입. 하지만 기존 프로토타입 모델을 폐지한 것은 아님. 사실 클래스는 함수이며, 프로토타입 기반 패턴을 클래스 기반 패턴처럼 사용할 수 있도록 하는 문법적 설탕(syntatic sugr)이다.
- 자바스크립트 클래스의 특징
  - new 연산자 없이 호출하면 에러 발생
  - 상속을 지원하는 extends와 super 키워드 제공
  - 호이스팅이 발생하지 않는 것처럼 동작한다.
  - 클래스 내의 모든 코드에는 암묵적으로 strict mode 적용됨
- 즉, _클래스는 자바스크립트에서의 새로운 객체 생성 매커니즘_

### 25.2 클래스의 정의

- 클래스는 값처럼 사용할 수 있는 일급 객체
- 클래스 몸체에 정의할 수 있는 메서드는 constructor, 프로토타입, 정적 메서드 총 3가지

  ```javascript
  class Person {
    // 생성자
    constructor(name) {
      // 인스턴스 생성 및 초기화
      this.name = name; // name 프로퍼티는 public
    }

    // 프로토타입 메서드
    sayHi() {
      console.log(`Hi! My name is ${this.name}`);
    }

    // 정적 메서드
    static sayHello() {
      console.log("Hello");
    }
  }

  const me = new Person("Lee");

  // 인스턴스의 프로퍼티 참조
  console.log(me.name); // Lee

  // 프로토타입 메서드 호출
  me.sayHi(); // Hi! My name is Lee

  // 정적 메서드 호출
  Person.sayHello(); // Hello
  ```

### 25.3 클래스 호이스팅

- 클래스는 함수로 평가됨
  ```javascript
  class Person {}
  console.log(typeof Person); // function
  ```
- 그렇가면 선언된 클래스 역시 런타임 이전에 먼저 평가되어 함수 객체를 생성!!!
- 생성자 함수는 함수 정의가 평가되어 함수 객체를 생성하는 시점에 프로토타입도 더불어 생성됨.
- 클래스는 클래스 정의 이전에 참조할 수 없음 -> 그럼 호이스팅 안되는건가? 그건 아님
- var, let, const, function, function\*, class 키워드 등의 모든 식별자의 선언문은 런타임 이전에 먼저 실행 및 평가되기 때문에 호이스팅이 일어남.

### 25.4 인스턴스 생성

- 클래스는 생성자 함수이며 new를 사용해 인스턴스 생성됨
- 기명 클래스 표현식의 클래스 이름을 사용해 인스턴스를 생성하려고 하면 참조 에러가 발생한다.
  ```javascript
  const Person = class MyClass {};
  const you = new MyClass(); // reference Error occur!
  const you1 = new Person(); // 이렇게 해야함
  ```
