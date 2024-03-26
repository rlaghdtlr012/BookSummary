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

### 아이템 39: any를 구체적으로 변형해서 사용하기

- any는 js 상의 모든 타입을 아우를 수 있기에 any보다 더 구체적으로 표현할 수 있는 타입을 사용하여 타입 안정성을 높이는 것이 좋다.
- 함수의 매개변수가 객체이긴 하지만 값을 알 수 없다면 {[key: string]: any}와 같이 선언하면 됨
- 만약에 매개변수가 배열이라면 타입을 any 보다는 any[]로 선언하는 것이 낫다.

### 아이템 40: 함수 안으로 타입 단언문 감추기

- 함수 내부에는 타입 단언을 사용하고 함수 외부로 드러나는 타입 정의를 정확히 명시하는 것이 바람직하다.
  ```typescript
  function cacheLast<T extends Function>(fn: T): T {
    let lastArgs: any[] | null = null;
    let lastResult: any;
    return function (...args: any[]) {
      if (!lastArgs || !shallowEqual(lastArgs, args)) {
        lastResult = fn(...args);
        lastArgs = args;
      }
      return lastResult;
    } as unknown as T;
  }
  ```
- 위 코드에 any가 많지만, 전부 함수 내부에서만 사용되고 있고, 타입 정의하는 곳에는 any가 없어서 cacheLast 함수를 호출하는 쪽에서는 any가 사용됐는지 알 지 못한다.
- any 단언을 사용해야 하는 상황이라면, 정확한 정의를 가지는 함수 안으로 숨기자.

### 아이템 41: any의 진화를 이해하기

- 변수의 타입은 변수를 선언할 때, 결정이 되고, 그 이에 새로운 값이 추가되도록 확장할수는 없다. 하지만 any는 예외

- 아래 코드에서 out은 선언 단계에서 any[]로 선언되었었지만 return된 out을 보면 number[] 타입으로 바뀐 것을 볼 수 있다. 왜 그럴까?
  ```typescript
  function range(start: number, limit: number) {
    const out = []; // 타입이 any[]
    for (let i = start; i < limit; i++) {
      out.push(i); // 타입이 any[]
    }
    return out; // 타입이 number[]
  }
  ```
- out은 any[]로 선언되었지만, number 타입의 값을 넣는 순간부터 number[]의 타입으로 "진화(evolve)"한다.(초기값을 null로 선언해도 마찬가지)
  ```typescript
  const result = []; // any[]
  result.push("a");
  result; // string[]
  result.push(1);
  result; // (number | string)[]
  ```
- 타입의 진화는 값을 할당하거나 배열에 요소를 넣은 후에만 일어난다.
- 타입을 안전하게 지키기 위해서는 **암시적 any를 진화시키는 방식보다, 명시적 타입 구문을 사용하는 것이 더 좋은 설계**

### 아이템 42: 모르는 타입의 값에는 any 대신 unknown을 사용하기

- unknown에는 함수의 반환값과 관련된 형태, 변수 선언과 관련된 형태, 단언문과 관련된 형태가 있음

- 함수의 반환값과 관련된 unknown

  - 함수의 반환값을 any로 하고(이러면 안됨), 함수를 호출한 곳에서 타입을 지정을 생략했다면, 그 값은 암시적으로 any가 되고, 사용되는 곳에서 타입 오류가 발생하게 된다.
  - 반환값으로 any를 사용하면 위험한 이유: any는 ts 타입의 집합관계에서 모든 타입의 부분 집합이면서 모든 타입의 상위 집합이기 때문에 문제가 됨.
  - 그에 비해, unknown은 어떠한 타입이든 unknown에 할당 가능하지만, unknown은 오직 unknown과 any에만 할당 가능
  - unknown 타입인 채로 값을 사용하면 오류 발생. 따라서 적절한 타입으로의 타입 변환이 강제됨.
  - 함수에서 제네릭보다는 unknown을 반환하고 사용자가 직접 단언문을 사용하거나 원하는 대로 타입을 좁히도록 강제하는 것이 좋음

- 단언문에서의 unknown

  - 이중 단언문에서 any 대신 unknown을 사용하는 것이 바람직하다.
  - 왜냐하면 any의 경우 오류가 사전에 발견되지 않아서 프로그램에 안좋은 영향을 미칠수도
  - 하지만 unknown은 사전에 즉시 오류를 발생시키므로 더 안전.

- unknown은 any보다 안전한 타입. 어떠한 값은 있지만, 그 타입을 알지 못하는 경우, unknown을 사용하자.
- 사용자가 타입 단언문이나 타입 체크를 사용하도록 강제하려면 unknown을 사용하면 된다.

### 아이템 43: 몽키 패치보다는 안전한 타입을 사용하기

- js의 장점: 객체나 클래스에 임의의 속성을 추가할 수 있을만큼 유연하다.
- window나 document등의 전역 객체에 속성을 추가할 때, ts의 타입 체커는 본 Document나 HTMLElement의 내장 속성에 대해서는 알고 있지만, 임의로 추가한 속성에 대해서는 알지 못한다.
  ```typescript
  document.monkey = "Tamarin";
  // 'Document' 유형에 'monkey' 속성이 없습니다.
  ```
- 위 문제를 해결하기 위해서는 any 타입 단언을 사용할 수 있지만, 이는 타입의 안정성을 상실하고, 언어 서비스를 사용할 수 없게 된다는 단점이 있음.
- 최선의 해결책은 document 혹은 DOM으롭부터 데이터를 분리하는 것. 만약 그럴수 없다면, 두 가지 차선책이 존재.
  - 첫 번째, interface의 특수 기능 중 하나인 보강(augmentation) 사용.
    ```typescript
    interface Document {
      monket: string;
    }
    document.monkey = "Tamarin"; // 정상
    // 보강이 any보다 나은 점: 타입이 더 안전, 속성에 자동완성 및 주석 붙이기 가능
    // 모듈적 관전에서는 global 선언을 추가해야 함
    declare global {
      interface Document {
        monkey: string;
      }
    }
    document.monkey = "Tamarin"; // 정상
    ```
  - 두 번째, 더 구체적인 타입 단언문 사용
    ```typescript
    interface MonketDocument extends Document {
      monkey: string;
    }
    // 더 구체적인 타입 단언
    (document as MonkeyDocument).money = "Macaque";
    // Document를 손상시키지 않고 새로운 타입 생성한다는 장점
    ```

### 아이템 44: 타입 커버리지를 추적하여 타입 안정성 유지하기

- noImplicitAny 설정 및 암시적 any 사용 대신 명시적 타입 구문을 추가해도 any 관련된 문제들을 다 해결한 것은 아님

  - any[]라든가 {[key: string]: any} 등의 명시적 any 타입 또한 코드 전반에 영향을 끼침
  - 서드파티 타입 선언(@types 선언 파일)들로부터 any 타입이 전파되기 때문에 조심해야 함.

- npx type-coverage 명령어를 통해 any를 추적할 수 있음
- ex: 9985 / 10117 98.69% => 전체 10117개 심벌 중, 9985개(98.69%)가 any가 아니거나 any의 별칭이 아닌 타입을 가지고 있다는 뜻
- npx type-coverage --detail => any 타입이 있는 곳을 모두 출력
