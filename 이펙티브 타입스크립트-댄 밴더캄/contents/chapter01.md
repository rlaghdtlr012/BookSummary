## 1장. 타입스크립트 알아보기

### 아이템 1: 타입스크립트와 자바스크립트의 관계 이해하기

- 타입스크립트는 자바스크립트의 상위 집합(superset)
- ts는 js의 상위집합이기에 .js 파일 안에 코드는 ts 코드라고봐도 무방
- 타입 시스템의 목표 중 하나는 런타임 오류를 발생시킬 수 있는 코드를 미리 찾아내는 것
- 그러나 ts는 모든 오류를 찾아내는 것은 아님. 타입 체커를 통과하면서 런타임 오류를 발생시키는 코드도 존재 가능

### 아이템 2: 타입스크립트 설정 이해하기

- 타입스크립트 설정을 제대로 사용하려면 tsconfig에서 noImplicitAny, strictNullChecks를 이해해야 함
  ```typescript
  function add(a, b) {
    // ~'a' 매개변수는 암시적으로 'any' 형식이 포함됩니다.
    // ~'b' 매개변수는 암시적으로 'any' 형식이 포함됩니다.
    return a + b;
  }
  ```
- 타입을 넣지 않아도 암묵적으로 any로 추론함. 오류 안남
- 하지만 noImplicitAny 옵션을 true로 한다면 암묵적인 any 타입 추론을 하지 않기에 오류가 남
- strictNullChecks: null과 undefined가 모든 타입에서 허용되는지 확인하는 설정
  ```typescript
  // strictNullChecks false 일 때는 오류 안뜸
  // strictNullChecks true 일때는 오류뜸(null 형식은 number 형식에 할당할 수 없습니다.)
  const x: number = null;
  ```
- strict 설정을 하면 대부분의 오류를 잡아줌

### 아이템 3: 코드 생성과 타입이 관계없음을 이해하기

- 타입스크립트의 2가지 역할
  - 최신 ts, js를 브라우저에서 동작할 수 있도록 **구버전 js로 트랜스파일(translate + compile)**한다.
  - 코드의 **타입 오류를 체크**한다.
- 위 두 가지 사안은 완전히 독립적. 즉, 타입 오류가 있는 코드도 컴파일이 가능
- 타입 오류가 있을 때, 컴파일되지 않게 하려면 noEmitOnError 설정
- 타입 정보를 유지하기 위한 방법 -> 타입 정보를 명시적으로 저장하는 '태그 기법'

  ```typescript
  interface Square {
    // 타입 정보 명시적으로 저장
    kind: "square";
    width: number;
  }
  interface Rectangle {
    // 타입 정보 명시적으로 저장
    kind: "rectangle";
    height: number;
    width: number;
  }
  type Shape = Square | Rectangle;

  function calculateArea(shape: Shape) {
    // 타입에 따른 분기 처리
    if (shape.kind === "rectangle") {
      return shape.width * shape.height;
    } else {
      return shape.width * shape.width;
    }
  }
  ```

- 그리고, Square과 Rectangle 둘 다, 클래스로 만들면 타입과 값 모두 사용할 수 있음!(대신 instanceof로 분기)
- as 등의 타입 단언문은 런타임에 아무런 영향을 주지 않음. 따라서 typeof 같은 방법으로 타입을 걸러야 함
- ts는 타입과 런타임이 무관하기에 함수 오버로딩이 불가능하다.
  ```typescript
  function add(a: number, b: number) {
    return a + b;
  }
  // ~ 중복된 함수 구현입니다.
  function add(a: string, b: string) {
    return a + b;
  }
  // ~ 중복된 함수 구현입니다.
  ```
- 위 코드는 C++, JAVA에서는 오버로딩이 가능하겠지만, ts에서는 런타임과 타입 체킹이 독립적이기 때문에 위와 같은 함수의 오버로딩이 불가능하다.
- 대신 온전한 타입수준에서는 동작 가능하다.

  ```typescript
  // 이렇게 온전한 타입 수준에서의 선언문을 작성 가능
  // 하지만 컴파일 되어 나오는 구현체는 하나다!!!!
  function add(a: number, b: number): number;
  function add(a: string, b: string): string;

  // 컴파일된 후 남게되는 하나의 구현체
  function add(a, b) {
    return a + b;
  }
  ```

### 아이템 4: 구조적 타이핑에 익숙해지기

- 자바스크립트는 본질적으로 덕 타이핑 기반
- ts는 이를 모델링하기 위해 구조적 타이핑을 사용함
- 어떤 인터페이스에 할당 가능한 값이라면 타입 선선에 명시적으로 나열된 속성들을 가지고 있을 것이며, 타입은 봉인 되어 있지 않음

### 아이템 5: any 타입 지양하기

- any 타입을 지양해야되는 이유
  - any 타입에는 타입 안정성이 없음: number 타입으로 선언되어 있는 값을 string으로 취급하는 상황이 나올 수도 있음
  - any는 함수 시그니처(함수를 호출하는 쪽은 약속된 입력을 입력하고, 함수는 약속된 타입을 반환한다는 뜻)를 무시해버림. 즉, 약속된 input과 output이 안나올수도 있음
  - any는 자동완성과 같은 언어 서비스가 지원되지 않음.(속성 추론 불가능)
  - any는 타입 설계를 감춰버린다: 객체 안의 수많은 속성의 타입을 일일이 작성해야 하는데, 이 때에도 any 타입을 사용하면 안됨. -> 왜냐하면 객체의 설계를 알 수 없게되기 때문
  - any는 타입시스템의 신뢰도를 떨어뜨림: any를 사용한다면 타입 체커의 성능이 안좋아지게 되고, 그렇게되면 타입체커에 대한 신뢰가 떨어지기 때문에 ts 도입 자체가 어려워질 수 있다.
