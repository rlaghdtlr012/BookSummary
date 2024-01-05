## 5장. any 다루기

- 타입스크립트의 타입 시스템은 정적이면서, 동적인 특징 동시에 가진다.
- 타입스크립트는 점진적 마이그레이션이 가능하다는 장점.
- 또한 코드의 일부분에 타입 체크를 비활성화 시켜주는 any 타입이 중요한 역할을 함.
- any의 장점을 살리고, 단점을 줄이는 법이 이 장의 주된 목표

### 아이템 38: any 타입은 가능한 한 좁은 범위에서만 사용하기

- 함수에서의 any 사용법

  ```typescript
  function processBar(b: Bar) {
    /** */
  }
  function f() {
    const x = returnValueFoo();
    processBar(x);
    // Bar 형식의 매개변수에 Foo 타입의 매개변수를 넣으면 타입 에러가 남
    // 'Foo' 형식의 인수는 'Bar' 형식의 매개변수에 할당될 수 없습니다.
  }
  ```

  - 해결책

    ```typescript
    // Bad
    function f1() {
      const x: any = returnValueFoo();
      processBar(x);
      return x; // Really Bad!!! 반환된 값을 다른 곳에서 사용할 경우, 타입 체크가 되지 않음
    }

    // Better
    function f2() {
      const x = returnValueFoo();
      processBar(x as any); // 차라리 이게 나음
    }
    ```

    - 변수에 any 타입을 지정할 바엔 차라리 as any의 형태가 더 권장됨
    - 왜나하면 x가 processBar 함수에만 사용됐을 뿐, 다른 코드에는 영향을 주지 않았기 때문
    - 만약 f1 함수가 return x; 의 형태로 x를 반환한다면, f1의 반환 타입은 any가 되어, 그 영향력은 프로젝트 전반에 악영향을 끼침!!

  - @ts-ignore를 사용하면 오류를 일시적으로는 해결할 수 있으나 근본적인 해결책은 아님

- 객체에서의 any 사용법

  ```typescript
  // 만약 큰 객체 config의 속성 하나(c)에서 타입 오류가 난 경우
  const config: Config = {
    a: 1,
    b: 2,
    c: {
      key: value,
    },
  } as any; // Bad!!!

  const config: Config = {
    a: 1,
    b: 2,
    c: {
      key: value as any, // Better
    },
  };
  ```

  - 객체 전체를 any로 단언하면, 다른 속성들(a, b) 또한 타입 체크가 되지 않는 부작용 발생.
  - 따라서 최소한의 범위에만 any를 사용하는 것이 좋음

- **즉, 의도치 않은 타입 안전성의 손실을 피하기 위해 any의 사용 범위를 최소한으로 좁혀야 한다!!**
- **함수에서 any 타입의 값을 반환하면 타입 안정성이 나빠지기에 절대로 any 타입의 값을 반환하면 안된다.**
