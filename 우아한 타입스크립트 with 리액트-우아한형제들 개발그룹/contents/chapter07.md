## 7.1 API 요청

### fetch로 API 요청하기

```typescript
// fetch API 요청을 보내서 cartItem의 목록을 가져온 뒤, cartCount를 업데이트 해주는 코드
const CartBadge: React.FC = () => {
  const [cartCount, setCartCount] = useState(0);

  useEffect(() => {
    fetch("https://api.baemin.com/cart").then(({ cartItem }) => {
      setCartCount(cartItem.length);
    });
  }, []);
};
```

- 하지만 API URL을 수정해야 하는 경우, 위와 같은 컴포넌트 내부의 비동기 호출 코드는 변경에 취약함.
- 또한 여러 API 사양 변경 사항에 대한 유연성이 떨어진다.

### 서비스 레이어로 분리하기

- 비동기 호출 코드는 컴포넌트 영역에서 분리되어 다른 영역(서비스 레이어)에서 처리 되어야 함.
- fetch 함수를 호출하는 부분이 서비스 레이어로 이동
- 컴포넌트는 서비스 레이어의 비동기 함수를 호출해서 렌더링 하는 흐름이 되어야 함

  ```typescript
  // 수정된 코드
  // 비동기 API 호출하는 부분을 다른 레이어로 아예 빼놓았다. 이러면 추후에 유지보수에 용의함
  async function fetchCart() {
      const controller = new AbortController();

      const timeoutId = setTimeout(() => controller.abort(), 5000);

      return await fetch("https://api.baemin.com/cart". {
          signal: controller.signal,
      })
  }
  ```

### Axios 활용하기

- API Entry가 2개 이상일 때, 각각의 API 요청처리하는 인스턴스를 따로 구성해야 한다.

  ```typescript
  const apiRequester: AxiosInstace = axios.create(defaultConfig);
  const orderApiRequester: AxiosInstance = axios.create({
    baseUrl: "https://api.baemin.or/",
    ...defaultConfig,
  });

  const orderCartApiRequester: AxiosInstance = axios.create({
    baseUrl: "https://cart.baemin.order/",
    ...defaultConfig,
  });
  ```

### Axios 인터셉터 사용하기

- 각각의 requester는 서로 다른 역할을 담당하는 서버이기 때문에 requester 별로 별도의 헤더를 설정해줘야 할 수도 있음.
- 인터셉터의 기능: requester에 따라 비동기 호출 내용을 추가해서 처리할 수 있게 함.

  ```typescript
  // 각각의 다른 API 요청 설정에 다른 Header 정보를 부여
  config.headers = {
    ...config.headers,
    "Content-Type": "application/json;charset=utf-8",
    user: getUserToken(),
    agent: getAgent(),
  };

  config.headers = {
    ...config.headers,
    "Content-Type": "application/json;charset=utf-8",
    "order-client": getOrderClientToken(),
  };
  ```

### API 응답 타입 지정하기

- API의 응답값을 하나의 Response 타입으로 묶을 수 있음

  ```typescript
  interface Response<T> {
    data: T;
    status: string;
    serverDateTime: string;
    errorCode?: string;
    errorMessage?: string;
  }

  const fetchCart = (): AxiosPromise<Response<FetchCartResponse>> => {
    apiRequester.get < Response < FetchCartResponse >> "cart";
  };
  ```

### SUPERSTRUCT를 사용하여 런타임에서 응답 타입 검증하기

- 런타임 응답 타입을 검증하기 위해 Superstruct라는 라이브러리가 사용됨

  - Superstruct를 사용하여 인터페이스 정의와 자바스크립트 데이터의 유효성 검사를 쉽게 할 수 있다.
  - Superstruct는 런타임에서의 데이터 유효성 검사를 통해 개발자와 사용자에게 자세한 런타임 에러를 보여주기 위해 고안됨

  ```typescript
  import {
    assert,
    is,
    validate,
    object,
    number,
    string,
    array,
  } from "superstruct";

  const Article = object({
    id: number(),
    title: stirng(),
    tags: array(string()),
    author: object({
      id: number(),
    }),
  });

  const data = {
    id: 34,
    title: "Hello World",
    tags: ["news", "features"],
    author: {
      id: 3,
    },
  };

  // superstruct를 이용한 유효성 검사
  assert(data, Article);
  is(data, Article);
  validate(data, Article);
  ```

- assert, is validate라는 모듈은 무엇인가?

  - 세 가지 다 데이터의 유효성 검사를 도와주는 모듈
  - 세 가지 다 데이터 정보를 담은 data 변수와 데이터 명세를 가진 스키마인 Article을 인자로 받아 데이터가 스키마에 부합하는지를 검사한다.
  - 차이점: 모듈마다 데이터의 유효성을 다르게 접근하고 반환값 형태가 다르다

- assert: 유효하지 않을 경우, 에러를 던짐
- is: 유효성 검사 결과에 따라 true 또는 false를 반환
- validate: [error, data] 형식의 튜플을 반환. 유효하지 않을 때는 에러 값이 반환, 유효한 경우, [undefined, dataValue]가 반환됨

```typescript
// 유효성 검사 사용 예시
import { assert } from "superstruct";

// listItems 배열을 순회하면서, 각각의 요소가 ListItem이라는 형식에 맞는 지 런타임 유효성을 검증하는 함수
function isListItem(listItems: ListItem[]) {
  listItems.forEach((item) => assert(item, ListItem));
}
```
