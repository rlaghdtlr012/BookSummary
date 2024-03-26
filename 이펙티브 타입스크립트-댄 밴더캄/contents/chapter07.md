## 7장. 코드를 작성하고 실행하기

### 아이템 53: 타입스크립트 기능보다는 ECMAScript 기능을 사용하기

- 초기 자바스크립트는 기능이 미비하여 타입스크립트 측에서 직접 클래스, enum 등의 기능을 만들어서 사용했다. 그런데 시간이 지나면서 자바스크립트에서도 해당 기능들이 추가되자 기존에 타입스크립트에서 만들어놓았던 기능들과 호환성 문제가 생김.

- 따라서 자바스크립트의 기능을 쓰되, 타입스크립트는 타입만 관장하는 방법을 채택

- 하지만 위 방향성이 확립되기 전에, ts가 이미 만들어놓은 기능들 중에서 값(자바스크립트)과 타입(타입스크립트)의 경계가 모호해지는 기능들이 있는데 그것들이 아래 케이스들(사용을 지양해야 하는 것들.호환성 문제 발생 가능).

- 열거형(enum)
  - 값들의 모음을 나타내기 위해 열거형 사용
    ```typescript
    enum Flavor {
      VANILLA = 0,
      CHOCOLATE = 1,
      STRAWBERRY = 2,
    }
    let flavor = Flavor.CHOCOLATE; // 타입이 Flavor, 값은 1
    ```
  - 문제점

### 아이템 54: 객체를 순회하는 노하우

- 다음 예제는 정상적으로 실행되지만, 편집기에서 오류 발생
  ```typescript
  const obj = {
    one: "uno",
    two: "dos",
    three: "tres",
  };
  for (const k in obj) {
    const v = obj[k];
    // ~obj에 인덱스 시그니처가 없기 때문에 엘리먼트는 암시적으로 'any' 타입입니다.
  }
  ```
- k의 타입은 string인 반면, obj의 key의 타입은 'one', 'two', 'three'. 세 개의 키만 존재하기 때문
- 즉, k와 obj 객체의 키 타입이 서로 다르게 추론되어 오류 발생. 따라서 k의 타입을 더 구체적으로 명시해주면 됨

  ```typescript
  let k: keyof typeof obj; // 'one' | 'two' | 'three' 타입
  for (k in obj) {
    const v = obj[k]; // 정상
  }
  ```

- 골치 아픈 **타입 문제 없이, 단순히 객체의 키와 값을 순회하고 싶다면** -> **Object.entries** 사용
  ```typescript
  interface ABC {
    a: string;
    b: string;
    c: string;
  }
  function foo(abc: ABC) {
    for (const [k, v] of Object.entries(abc)) {
      k; // string 타입
      v; // any 타입
    }
  }
  ```
- 객체를 순회하며 키, 값을 얻으려면 (let k: keyof T)와 같은 keyif 선언이나 Object.entries를 사용한다.

### 아이템 55: DOM 계층 구조 이해하기

- 타입스크립트에서는 DOM 엘리먼트의 계층 구조를 파악하기 용이

  | 타입              | 예시                         |
  | ----------------- | ---------------------------- |
  | EventTarget       | window, XMLHttpRequest       |
  | Node              | document, Text, Comment      |
  | Element           | HTMLElement, SVGElement 포함 |
  | HTMLElement       | <i\>, <b\>                   |
  | HTMLButtonElement | <button>                     |

- EventTarget: DOM 타입 중 가장 추상화된 타입. 이벤트리스너를 추가하거나 제거하고, 보내는 것 밖에 못함
- Node: 엘리먼트 뿐만 아니라 텍스트 조각과 주석까지도 포함
- Elemet와 HTMLElement: <svg\> -> SVGElement, <html\> -> HTMLElement
- HTMLxxxElement: html의 여러가지 속성에 해당하는 element, ex: <input\> -> HTMLInputElement. 각 속성에 접근하려면 타입 저ㅇ보 역시 실제 엘리먼트 타입이어야 함. 즉, 타입을 매우 구체적으로 명시해줘야 함
- DOM, Event 관련 타입 오류가 날 경우, 코드를 작성할 때, 해당 엘리먼트의 타입을 보다 정확하고 자세하게 타입 지정을 해주는 것이 좋다.
- 또한 if 문을 통한 null 체크도 해주어야 한다.

### 아이템 56: 정보를 감추는 목적으로 private 사용하지 않기

- 타입스크립트에서의 public, private, protected 접근 제어자는 공개 규칙을 강제할 수 있는 것으로 **오해**할수도 있음.
- 하지만 public, private, protected 접근 제어자는 타입스크립트 키워드이기 때문에 컴파일 후에 제거됨.
- 타입스크립트의 접근 제어자들은 단지 컴파일 시점에만 오류를 표시해 줄 뿐, 런타임에는 아무런 효력이 없음
- 즉, 정보를 숨기기 위해 private을 사용하면 안됨
- 자바스크립트에서 정보를 숨기기 위한 가장 효과적인 방법: 클로저(closure)

  ```typescript
  declare function hash(text: string): number;

  // PasswordChecker 외부에서 passwordHash 변수에 접근할 수 없음.
  // 따라서 메서드 역시 생성자 안에 정의되어야 함
  // 메서드가 생성자 내부에 있을 시, 인스턴스 생성 때마다 메모리 낭비
  class PasswordChecker {
    checkPassword: (password: string) => boolean;
    constructor(passwordHash: number) {
      this.checkpassword = (password: string) => {
        return hash(password) === passwordHash;
      };
    }
  }
  ```

### 아이템 57: 소스맵을 사용하여 타입스크립트 디버깅 하기

- 타입스크립트 코드를 실행하는 것은 엄밀히 말하자면, 타입스크립트 컴파일러가 생성한 자바스크립트 코드를 실행한다는 것
- 하지만 디버거는 런타임에 동작하며, 타입스크립트는 컴파일 과정에서 사라짐
- 컴파일이 끝난 자바스크립트 코드를 디버깅 할 경우, 디버깅이 어려워짐
- 디버깅 때 남은 코드는 전처리, 컴파일 과정이 다 끝난 복잡한 코드 -> 디버깅 어려워짐
- 브라우저와 IDE에서의 소스맵을 지원해 줌
- tsconfig 파일에서 CompilerOptions에 SourceMap 속성을 true로 하면 브라우저 상에서 소스맵이 생성됨.
- 소스맵에 대해 알아야 할 사항
  - 타입스크립트에서 번들러, 압축기를 사용하고 있다면, 번들러나 압축기가 각자의 소스맵을 생성
  - 번들러가 타입스크립트의 소스맵을 잘 만들 수 있게 설정 필요
  - 디버거를 열지 않으면 소스맵이 로드되지 않지만, 상용 환경에서 소스맵이 공개(유출)되고 있는지를 확인해야 함
