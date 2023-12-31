## 3장 타입 추론

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
