## 3장. 타입 추론

### 아이템 19: 추론 가능한 타입을 사용해 장황한 코드 방지하기

- ts에서의 많은 타입 구문은 사실 불필요하다(왜냐하면 ts에서 자체적으로 이미 타입 추론을 해주고 있기 때문)
- 따라서 모든 변수에 타입을 선언하는 것은 비생산적인 일.
  ```typescript
  let x: number = 12; // 이것보다
  let x = 12; // 이 방식으로만 해도 충분하다.
  // 왜냐하면 이미 x의 타입을 number라고 추론하고 있기 때문
  ```
- 타입 추론이 된다면 명시적 타입 구문이 굳이 필요하지 않음(변수, 객체, 배열 모두 이에 해당). 오히려 방해가 됨

- 불필요한 타입 명시가 오히려 방해가 되는 케이스

  ```typescript
  // 초기에 id를 string으로 지정해뒀다가 나중에 id가 number가 될 수도 있는 경우, 코드를 아래와 같이 짤 수 있다.
  // 하지만 이미 string으로 명시한 id의 타입을 id: number라고 지정하면 타입 에러가 발생한다.
  interface Product {
    id: string;
    name: string;
    price: number;
  }

  function logProduct(product: Product) {
    const id: number = product.id; // 'string' 형식은 'number' 형식에 할당할 수 없습니다.
    const name: string = product.name;
    const price: number = product.price;
  }
  ```

  - 위와 같은 경우, 차라리 타입 명시를 하지 않았더라면 타입 체커를 통과하여 오류가 발생하지 않았을 것.

  - 위 케이스의 경우 logProduct는 비구조화 할당문을 사용하여 구현하는 것이 더 나음
    ```typescript
    // 비구조화 할당을 사용하여 타입 추론
    function logProduct(product: Product) {
      const {id, name. price} = product;
      console.log(id, name, price);
    }
    ```

- 타입 명시는 ts 타입 체커가 스스로 타입을 추론하기에 정보가 부족할 경우에 하는 것이 바람직함.

- 이상적인 ts 코드는 함수/메서드 시그니처에 타입 구문을 포함하되, **함수 내에서 생성된 지역 변수에는 타입 구문을 생략하여, 코드를 읽는 사람으로 하여금 구현 로직에 집중할 수 있게 하는 것이 좋음**

- 타입 추론이 가능함에도 불구하고, 타입을 명시하는 것이 바람직한 경우

  - 객체 리터럴을 정의할 때

    ```typescript
    const furby = {
      name: "furby",
      id: 231424134,
      price: 35,
    };
    logProduct(furby); // 타입에러 발생. id 속성의 형식이 호환되지 않습니다.

    const furby: Product = {
      nname: "furby",
      id: 231424134, // 'number' 형식은 'string' 형식에 할당할 수 없습니다.
      price: 35,
    };
    ```

  - 함수의 반환 타입을 추론할 수 있더라도 반환 타입에 타입을 명시하여 오류를 방지할 수 있음.
    - 함수의 반환 타입을 명시해야 하는 이유
      - 함수에 대해 더욱 명확하게 알 수 있기 때문에. 함수의 반환값으로 어떤 타입이 올 지, 알고 로직을 작성하기 때문에 주먹구구식으로 로직을 구현하는 것을 방지할 수 있음
      - 명명된 타입을 사용하기 위해서.

### 아이템 20: 다른 타입에는 다른 변수 사용하기

- js에서는 한 변수에 여러 타입을 가지는 값을 재지정해도 되지만 ts에서는 안됨
  ```typescript
  let id = "qeqweasd";
  someFunction(id);
  id = 12313; // '12313' 형식은 'string' 형식에 할당할 수 없습니다.
  ```
- 위와 같은 경우에, ts에서 id를 이미 string으로 추론하고 있는 상황에서 number의 값으로 재할당을 할 경우, 타입 에러가 발생한다.
- 즉, **변수의 값은 바뀔 수 있지만, 변수의 타입은 보통 바뀌지 않는다!**
- 유니온 타입을 사용하여 string | number의 방법으로 해결할 수 있지만, 이 방법은 id를 사용할 때마다 id가 어떤 타입인지를 확인해야 하기 때문에 더 번거로울 수 있다.

- 따라서 변수를 여러 타입에 따라 재사용하는 것이 아닌, **다른 타입에는 별도의 변수를 사용하는 것이 가장 바람직하다.**

- 타입에 따라 별도의 변수를 지정하는 것의 장점
  - 서로 관련이 없는 두 개의 값을 분리한다.
  - 변수명을 더 구체적으로 지을 수 있음.
  - 타입 추론을 향상시키며, 타입 구문이 불필요해진다.
  - 타입이 더 간결해진다.(string | number 대신 string과 number 별도로 사용)
  - let 대신 const로 변수를 선언할 수 있게 됨. const로 변수를 선언하면 더 간결하고, 타입 추론이 쉬운 코드가 됨

### 아이템 21: 타입 넓히기

- 타입 체커는 변수에 지정된 단일 값을 가지고 할당 가느안 값들의 집합을 유추해야 함.
- 이러한 과정을 '타입 넓히기'라고 부름
- 아래 코드는 오류가 발생함

  ```typescript
  interface Vector3 {
    x: number;
    y: number;
    z: number;
  }
  function getComponent(vector: Vector3, axis: "x" | "y" | "z") {
    return vector[axis];
  }

  let x = "x";
  let vec = { x: 10, y: 20, z: 30 };
  getComponent(vec, x);
  // 'string' 형식의 인수는 '"x" | "y" | "z"' 형식의 매개변수에 할당될 수 없습니다.
  ```

- 오류가 나는 이유는 let x를 'x'로 할당해줬음에도 불구하고 '타입 넓히기'로 인해 x의 타입이 '"x" | "y" | "z"'가 아닌 'string'으로 추론됐기 때무
- ts에서는 합리적으로 타입을 추론하려고 하지만, 작성자의 타입 지정 의도까지 완벽하게 파악하지는 못한다.
- 해결책 (const를 사용하여 타입 좁히기)
  ```typescript
  const x = "x";
  let vec = { x: 10, y: 20, z: 30 };
  getComponent(vec, x); // 정상
  ```
- 왜냐하면 const는 재할당 될 수 없으므로 ts는 의심의 여지 없이 더 좁은 타입의 'x'를 추론할 수 있음.

- 타입 추론의 강도를 직접 제어하는 방법
  - _명시적 타입 구문 제공하기_
    ```typescript
    const v: { x: 1 | 3 | 5 } = {
      x: 1,
    } x의 타입이 1 | 3 | 5
    ```
  - _타입 체커에 추가적인 문맥 제공하기(아이템 26)_
  - _const 단언문 사용하기_
    ```typescript
    const v1 = {
      x: 1,
      y: 2,
    }; // 타입은 { x: number; y: number; };
    const v2 = {
      x: 1 as const,
      y: 2,
    }; // 타입은 { x: 1; y: number; };
    const v3 = {
      x: 1,
      y: 1,
    } as const; // 타입은 { readonly x: 1; readonly y: 2; }
    ```

### 아이템 22: 타입 좁히기

- ts가 넓은 타입으로부터 좁은 타입으로 진행되는 과정을 의미. 대표적인 예시가 null 체크
- 분기문에서 예외를 던지거나 instanceof를 사용하여 타입을 좁히는 등의 방법이 있다.

  ```typescript
  const el = document.getElementById("foo");
  // 1, 예외를 던지는 방법을 통한 타입 좁히기
  if (!el) throw new Error("Unable to find #foo");

  // 2. instanceof 방법을 통한 타입 좁히기
  function contains(text: string, search: string | RegExp) {
    if (search instanceof RegExp) {
    }
  }

  // 3. 명시적 '태그'를 붙임으로써 타입 좁히기
  interface UploadEvent {
    type: "upload";
    filename: string;
    contents: string;
  }
  interface DownloadEvent {
    type: "download";
    filename: string;
  }
  type AppEvent = UploadEvent | DownloadEvent;
  function handleEvent(e: AppEvent) {
    switch (e.type) {
      case "download":
        e; // 타입이 DownloadEvent
        break;
    }
  }
  ```

### 아이템 23: 한꺼번에 객체 생성하기

- 일반적으로 변수의 값은 바뀔 수 있지만, ts의 타입은 변경되지 않음. 즉, 객체를 생성할 때, 속성을 하나씩 추가하기 보다는 여러 속성을 포함해서 한꺼번에 타입 생성을 하는 것이 유리함

  ```typescript
  // ts에서는 아래와 같은 상황에서 오류가 뜸
  const pt = {};
  pt.x = 3; // '{}' 형식에 'x' 속성이 없습니다.
  pt.y = 4; // '{}' 형식에 'y' 속성이 없습니다.

  // 다음과 같은 방식을 사용하면 오류 내용이 바뀜
  interface Point {
    x: number;
    y: number;
  }

  const pt: Point = {};
  // '{}' 형식에 'Point' 형식의 x, y 속성이 없습니다.
  pt.x = 3;
  pt.y = 4;

  // 해결책 1: 객체를 한 번에 정의함으로 해결
  const pt = {
    x: 3,
    y: 4,
  }; // 정상

  // 해결책 2: 타입 단언문(as)를 사용하여 해결
  const pt = {} as Point;
  pt.x = 3;
  pt.y = 4;

  // 해결책 1이 더 나음
  ```

- 안전한 타입으로 속성을 추가하려면 destructuring({...a, ...b})의 방법을 사용하는 것도 좋음

### 아이템 24: 일관성 있는 별칭 사용하기

```typescript
const borough = { name: "Brooklyn", location: [40.688, -73.979] };

// borough.location 배열에 loc이라는 별칭 부여
const loc = borough.location;

loc[0] = 0;
console.log(borough.location); // [0, -73.979]
```

- 별칭의 값을 변경하면 원래 속성값도 변경됨.(제어의 흐름 분석이 어렵게 됨)

- 별칭을 사용했을 때, 타입이 의도한대로 추론되지 않는 경우의 예시

  ```typescript
  interface Coordinate {
    x: number;
    y: number;
  }

  interface BoundingBox {
    x: [number, number];
    y: [number, number];
  }

  interface Polygon {
    exterior: Coordinate[];
    bbox?: BoundingBox;
  }
  function isPointInPolygon(polygon: Polygon, pt: Coordinate) {
    polygon.bbox; // 타입이 BoundingBox | undefined
    const box = polygon.bbox;
    box; // 타입이 BoundingBox | undefined
    if (polygon.bbox) {
      polygon.bbox; // 타입이 BoundingBox
      box; // 타입이 BoundingBox | undefined -> 여기서 문제 발생!!!
    }
  }
  ```

- 위 예시에서 polygon.bbox는 타입이 정제되었지만, 별칭인 box타입은 정제되지 않는 모습.
- 위 현상을 '별칭을 일관성 있게 사용한다.'는 기본 원칙을 지키면 방지할 수 있음.

  ```typescript
  if (box) {
    // ~~~ 정상. 이런 식으로 별칭의 타입 좁히기를 사용하여 오류를 방지할 수 있음
    // 하지만 이는 코드를 읽는 이로 하여금, polygon.bbox와 box는 같은 값이지만
    // 다른 변수를 사용하고 있기에 혼란을 줄 수 있는 문제가 남아있음
  }

  // 객체 비구조화를 통해 해결 가능
  const { bbox } = polygon;
  if (bbox) {
    const { x, y } = bbox;
    // ~~ 정상. 객체 비구조화를 통해 같은 타입(값)에 대해 같은 변수명을 사용가능하게 함
  }
  ```

- 타입 별칭은 ts가 타입을 좁히는 것을 방해하기에 일관되게 사용해야 한다.
- 비구조화 문법을 통해 일관된 이름을 사용해야 한다.

### 아이템 25: 비동기 코드에는 콜백 대신 async 함수 사용하기

- await 키워드는 각각의 프로미스가 처리(resolve) 될 때까지 fetchPages 함수의 실행을 멈춤
- 프로미스가 거절(reject)되면 예외를 던짐. 따라서 통상적으로 try ~ catch 문과 함께 사용
  ```typescript
  async function fetchPages() {
    try {
      const response1 = await fetch(url1);
      const response2 = await fetch(url2);
      const response3 = await fetch(url3);
    } catch (e) {
      // ...
    }
  }
  ```
- async/await의 장점
  - 콜백보다 프로미스가 코드를 작성하기 쉬움
  - 콜백보다 프로미스가 타입을 추론하기 쉬움
- 가급적 프로미스를 생성하기보다는 async, await을 사용하는 것이 좋음
  ```typescript
  // async 함수 예시
  const getNumber = async () => 42; // 타입이 () => Promise<number>
  // 프로미스 함수 예시
  const getNumber = () => Promise.resolve(42); // 타입이 () => Promise<number>
  // 둘은 같은 역할을 하는 함수들
  ```
- async 함수에서 프로미스를 반환하면 또 다른 프로미스로 래칭되지 않음.
- 반환 타입은 `Promise<Promise<T>>` 가 아닌 `Promise<T>`가 됨.(반환 타입이 명확해짐)

### 아이템 26. 타입 추론에 문맥이 어떻게 사용되는지 이해하기

- ts는 타입을 추론할 때, 단순히 값만 고려하는 것이 아닌, 값이 존재하는 곳의 문맥까지도 살핌

  ```typescript
  type Language - 'JavaScript' | 'TypeScript' | 'Python';
  function setLanguage(language: Language) { /** */ }

  setLanguage('JavaScript'); // 인라인 형태. 정상

  let language = 'JavaScript'; // 참조 형태. 타입 오류남
  setLanguage(language); // 'string' 형식의 인수는 'Language'명식의 매개변수에 할당될 수 없습니다.
  ```

- 인라인 형태에서 ts는 함수 선언을 통해 매개변수가 Language 타입이어야 한다는 것을 알고 있음.
- 허나, 이 값을 변수로 분리해내면, ts는 let language = 'JavaScript'에서의 할당 시점에 language의 타입을 제대로 추론하지 못한다.(language의 타입이 string인지, Language인지 판단 불가)
- 해결 방법 2가지
  - 타입 선언 때, language의 타입을 제한 하는것
    ```typescript
    // 변수 할당 시점에 타입 제한
    let language: Language = "JavaScript";
    setLanguage(language); // 정상
    ```
  - language를 상수로 만드는 방법
    ```typescript
    // language를 상수로 만들어서 타입 체커에게 language는 변하지 않는 값이라고 알려준다.
    // 따라서 더 정확한 'JavaScript'라는 문자열 리터럴 추론 가능
    const language = "JavaScript";
    setLaguage(language); // 정상
    ```
- 튜플에서의 타입추론 불가 문제

  ```typescript
  function panTo(where: [number, number]) {
    /** */
  }
  panTo([10, 20]); // 정상

  const loc = [10, 20];
  panTo(loc); // 'number[]' 형식의 인수는 '[number, nymber]' 형식의 매개변수에 할당될 수 없습니다.
  ```

  - 위 예시처럼 panTo는 loc을 number[](길이를 알 수 없는 배열)로 추론함.
  - 해결책

    ```typescript
    const loc: [number, number] = [10, 20]; // ts가 의도를 정확히 파악할 수 있도록 타입 선언을 제공하는 방법
    panTo(loc);

    const loc = [10, 20] as const;
    panTo(loc); // 'readonly [10, 20]' 형식은 'readonly'이며, 변경 가능한 형식 '[number, number]'에 할당할 수 없습니다.
    ```

    - const는 값이 가리키는 참조가 변하지 않는다는 얕은(shallow) 상수
    - as const는 그 값의 내부(deeply)까지 상수라는 뚯
      - 즉, as const는 readonly가 되어, 추론을 과하게 정확하게 하는 현상이 발생
      - 따라서 panTo 함수의 파라미터에 readonly 구문을 추가해서 해결
        ```typescript
        function panTo(where: readonly [number, number]) {
          /** */
        }
        const loc = [10, 20] as const;
        panTo([10, 20]); // 정상
        ```

- 객체에서의 타입추론 불가 문제

  ```typescript
  type Language = "JavaScript" | "TypeScript" | "Python";
  interface GovernedLanguage {
    language: Language;
    organization: string;
  }

  function complain(language: GovernedLanguage) {
    /**  */
  }
  complain({ language: "TypeScript", organization: "MS" }); // 정상

  const ts = {
    language: "TypeScript",
    organization: "MS",
  };
  complain(ts); // '{ language: 'string, organization: 'string'}' 형식의 인수는 'GovernedLanguage' 형식의 매개변수에 할당될 수 없습니다.
  // language 속성의 형식이 호환되지 않습니다.
  // string 형식은 Language 형식에 할당할 수 없습니다.
  ```

  - ts 객체에서 language의 타입은 string으로 추론됨
  - 해결책
    - ts 객체에 타입 선언 추가(const ts: GovernedLanguage = ...)
    - 상수 단언(const ts = {~~~} as const) 사용

- 즉, 변수를 따로 뽑아서 별도로 선언했을 때, 오류가 발생한다면, 타입 선언을 추가해줘야 한다.
- 변수가 정말로 상수라면 상수 단언(as const)를 사용해야 한다.

### 아이템 27: 함수형 기법과 라이브러리로 타입 흐름 유지하기

- 자바스크립트에서는 서드파티 라이브러리(lodash, 람다 등) 종속성을 추가할 때, 서드파티 라이브러리를 사용하여 코드를 짧게 만드는데 시간이 오래든다면, 사용하지 않는 편이 나음.
- 타입스크립트로 작성하면 서드파티 라이브러리를 사용하는 것이 무조건 유리.
- 왜냐하면 타입 정보를 참고하며 작업할 수 있기 때문에
- 내장된 함수형 기법과 lodash 같은 유틸ㄹ리티 라이브러리를 사용하는 것이 타입 흐름을 개선하고, 가독성을 높이고, 명시적인 타입 구문의 필요성을 줄여준다.
