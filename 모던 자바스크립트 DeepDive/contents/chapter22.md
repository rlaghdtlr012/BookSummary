## 22.1 this 키워드

- 객체란 *상태*를 나타내는 프로퍼티와 *동작*을 나타내는 메서드를 하나의 논리적인 단위로 묶은 복합적인 자료구조
- 동작을 나타내는 메서드는 자신이 속한 객체의 상태를 참조하고 변경할 수 있어야 하는데, 이때 메서드가 자신이 속한 객체의 프로퍼티를 참조하려면, **자신이 속한 객체를 가리키는 식별자를 참조할 수 있어야 함**
- 즉, 자신이 속한 객체를 가리키는 식별자 === this
- this: 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 자기 참조 변수. this를 통해 자신이 속한 객체 또는 자신이 생성할 인스턴스의 프로퍼티나 메서드를 참조할 수 있음.
- this는 자바스크립트 엔진에 의해 암묵적으로 생성. 코드 어디서든 참조 가능
- this가 가리키는 값, 즉 this 바인딩은 함수 호출 방식에 의해 **동적으로 결정**됨
- this 바인딩: this가 가리킬 객체와 this를 바인딩 하는 것

```javascript
const circle = {
  radius: 5,
  getDiameter() {
    return 2 * this.radius;
  },
};
console.log(circle.getDiameter());
// 이 경우에서 객체 안의 this == 메서드를 호출한 객체, 즉 circle을 가리킴
```

```javascript
function Circle(radius) {
  // this는 생성자 함수가 생성할 인스턴스를 가리킴
  this.radius = radius;
}
Circle.prototype.getDiameter = function () {
  // this는 생성자 함수가 생성할 인스턴스를 가리킴
  return 2 * this.radius;
};

// 인스턴스 생성
const circle = new Circle(5);
console.log(circle.getDiameter()); // 10
```

- 자바나 C++에서의 this는 언제나 클래스가 생성하는 인스턴스를 가리키지만, 자바스크립트에서의 this는 함수가 호출되는 방식에 따라 this에 바인딩 될 값, 즉 this 바인딩이 동적으로 결정됨
- 일반 함수 내에서의 this == 전역 객체인 window
- 메서드 내부에서 this는 메서드를 호출한 객체
- 생성자 함수 내부에서 this는 생성자 함수가 생성할 인스턴스

## 22.2 함수 호출 방식과 this 바인딩

- this 바인딩(this에 바인딩 될 값)은 함수가 어떻게 호출되었는지에 따라 동적으로 결정됨
- this 바인딩은 함수 호출 시점에 결정됨

### 22.2.1 일반 함수 호출

- 기본적으로 this에는 전역 객체가 바인딩 됨
- 전역 함수 및 중첩 함수를 일반 함수로 호출하면 함수 내부의 this에는 전역 객체(window)가 바인딩 됨

```javascript
var value = 1;

const obj = {
  value: 100,
  foo() {
    console.log(this); // {value: 100, foo: f}
    console.log(this.value); // 100

    // 메서드 내에서 정의한 중첩 함수
    function bar() {
      console.log(this); // window;
      consol.log(this.value); // 1
    }
  }
  bar();
};

obj.foo();
```

- **어떠한 함수라도 일반 함수로 호출되면 this에 전역 객체가 바인딩 된다.**
- 일반 함수로 호출된 모든 함수(중첩 함수, 콜백 함수 포함) 내부의 this에는 전역 객체가 바인딩 된다.

#### 메서드 내부의 중첩 함수나 콜백 함수의 this 바인딩을 메서드의 this 바인딩과 일치시키는 방법

- Function.prototype.apply, Function.prototype.call, Function.prototype.bind 메서드를 통해 this의 명시적 바인딩 가능

  ```javascript
  var value = 1;

  const obj = {
    value: 100,
    foo() {
      console.log(this); // {value: 100, foo: f}
      console.log(this.value); // 100

      // 메서드 내에서 정의한 중첩 함수
      function bar() {
        consol.log(this.value); // 100
      }.bind(this);
    }
    bar();
  };

  obj.foo();
  ```

  또는 **화살표 함수를 사용해서 this 바인딩**을 일치시킬 수도 있음

  ```javascript
  var value = 1;
  const obj = {
    value: 100,
    foo() {
      setTimeout(() => console.log(this.value), 100); // 100
    },
  };
  obj.foo();
  ```

### 22.2.2 메서드 호출

- 메서드 내부의 this는 메서드를 호출한 객체에 바인딩 된다.
- 메서드 내부의 this는 프로퍼티로, 메서드를 가리키고 있는 객체와는 관계가 없고 메서드를 **호출한 객체에 바인딩이 됨**

### 22.2.3 생성자 함수 호출

- 생성자 함수 내부의 this에는 생성자 함수가 (미래에) 생성할 인스턴스가 바인딩 된다.

  ```javascript
  function Circle(radius) {
    // 생성자 함수 내부의 this는 생성자 함수가 생성할 인스턴스를 가리킨다.
    this.radius = radius;
    this.getDiameter = function () {
      return 2 * this.radius;
    };
  }

  // 반지름이 5인 Circle 객체를 생성
  const circle1 = new Circle(5);

  // 반지름이 10인 Circle 객체를 생성
  const circle2 = new Circle(10);

  console.log(circle1.getDiameter()); // 10
  console.log(circle2.getDiameter()); // 20
  ```

### 22.2.4 Function.prototype.apply/call/bind 메서드에 의한 간접 호출

- apply, call, bind 메서드는 Function.prototype의 메서드. 즉, 이들 메서드는 모든 함수가 상속받아 사용할 수 있음
- apply, call 메서드의 본질적인 기능: 함수 호출
- bind 메서드는 메서드의 this와 메서드 내부의 중첩 함수 또는 콜백 함수의 this가 불일치하는 문제를 해결하기 위해 유용하게 사용됨

  ```javascript
  const person = {
    name: "Lee",
    foo(callback) {
      // bind 메서드로 callback 함수 내부의 this 바인딩을 전달
      setTimeout(callback.bind(this), 100);
    },
  };

  person.foo(function () {
    console.log(`Hi! my name is ${this.name}`); // Hi! my name is Lee
  });
  ```

- 함수 호출 방식에 따라 this 바인딩이 동적으로 결정됨
  | 함수 호출 방식 | this 바인딩 |
  | ----------------- | ---------------------------- |
  | 일반 함수 호출 | 전역 객체(window or Node.js) |
  | 메서드 호출 | 메서드를 호출한 객체 |
  | 생성자 함수 호출 | 생성자 함수가 (미래에) 생성할 인스턴스 |
  | Function.prototype.apply/call/bind 메서드에 의한 간접 호출 | Function.prototype.apply/call/bind 메서드에 첫번째 인수로 전달한 객체 |
