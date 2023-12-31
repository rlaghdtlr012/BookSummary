## 5.1 조건부 타입

### extends와 제네릭을 활용한 조건부 타입

- T extends U ? X : Y
- **타입 T를 U에 할당할 수 있으면 X 타입, 아니면 Y 타입으로 결정됨을 의미**

  ```typescript
  interface Bank {
    financialCode: string;
    companyName: string;
    name: string;
    fullName: string;
  }

  interface Card {
    financialCode: string;
    companyName: string;
    name: string;
    appCardType?: string;
  }

  type PayMethod<T> = T extends "crad" ? Card : Bank;
  type CardPayMethodType = PayMethod<"card">;
  type BankPayMethodType = PayMethod<"bank">;
  ```

- Bank는 계좌를 이용한 결제 수단 정보, Card 타입과 다른 점은 fullName으로 은행의 전체 이름 속성을 가지고 있음
- Card는 카드를 이용한 결제 수단 정보, Bank 타입과 다른 점은 appCardType으로 카드사 앱을 사용해서 카드 정보를 등록할 수 있는지를 구별해주는 속성이 있음
- PayMethod 타입은 제네릭 타입으로 extends를 사용한 조건부 타입. 제네락 매개변수에 card가 들어오면 Card 타입, 그 외의 값이 들어오면 Bank 타입으로 결정됨.
- PayMethod를 사용해서 CardPayMethod, BankPayMethod를 도출할 수 있음.

### 조건부 타입을 사용하지 않았을 때의 문제점

```typescript
interface PayMethodBaseFromRes {
  financialCode: string;
  name: string;
}

interface Bank extends PayMethodBaseFromRes {
  fullName: string;
}

interface Card extends PayMethodBaseFromRes {
  appCardTyle?: string;
}

type PayMethodInfo<T extends Bank | Card> = T & PayMethodInterface;

type PayMethodInterface {
  companyName string;
  ...
}
```

- payMethodsBaseFromRes: 서버에서 받아오는 결제 수단 기본 타입으로 은행과 카드에 모두 들어가 있음
- Bank, Card: 은행과 카드 각각에 맞는 결제 수단 타입. 결제 수단 기본 타입인 PayMethodBaseFromRes를 상속받아 구현
- PayMethodInterface: 프론트에서 관리하는 결제 수단 관련 데이터. UI를 구현하는데 사용하는 타입
- PayMethodInfo<T extends Bank | Card>: 최종적인 은행, 카드 결제 수단 타입. 프론트에서 추가되는 UI 데이터 타입과 제네릭으로 받아오는 Bank 또는 Card를 합성. extends를 제네릭에서 한정자로 사용하여 **Bank 또는 Card를 포함하지 않는 타입은 제네릭으로 으로 넘겨주지 못하게 방어**. BankPayMethodInfo = PayMethodInterface & Bank처럼 카드와 은행의 타입을 만들어줄 수 있지만, 제네릭을 확용해서 중복된 코드를 제거

```typescript
/**
 * desc: useGetRegisteredList 함수는 react-query의 useQuery를 사용하여 구현한 커스텀 훅
 * useGetRegisteredList 함수는 useQuery의 반환값을 돌려줌.
 * useCommonQuery<T>는 useQuery를 한 번 래핑해서 사용하고 있는 함수. useQuery의 반환 data를 T 타입으로 반환
 * fetcherFactory는 axios를 래핑해주는 함수. 서버에서 데이터를 받아온 후, onSuccess 콜백 함수를 거친 결과값을 반환해줌
 */
type PayMethodType = PayMethodInfo<Card> | PayMethodInfo<Bank>;

export const useGetRegisteredList = (
  type: "card" | "appcard" | "bank"
): UseQueryResult<PayMethodType[]> => {
  const url = `baeminpay/codes/${type === "appcard" ? "card" : type}`;

  const fetcher = fetcherFactory<PayMethodType[]>({
    onSuccess: (res) => {
      const usablePocketList =
        res?.filter(
          (pocket: PocketInfo<Card> | PocketInfo<Bank>) =>
            pocket?.useType === "USE"
        ) ?? [];
      return usablePocketList;
    },
  });

  const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher);

  return result;
};
```

- useGetRegisteredList 함수는 타입으로 "card", "appcard", "bank"를 받아서 **해당 결제 수단의 결제 수단 정보를 반환하는 함수**
- useGetRegisteredList 함수가 반환하는 Data 타입은 PayMethodInfo<Card> | PayMethodInfo<Bank>.
- 사용자가 타입으로 card를 넣었을 때, useGetRegisteredList 함수가 반환하는 Data 타입은 PayMethodType 유추 가능
- 하지만 useGetRegisteredList가 실제로 반환하는 Data는 PayMethodType
- 이처럼 사용자의 의도와는 다르게 정확한 타입을 반환하지 못하는 함수가 되어버림
- 인자에 따라 반환되는 타입을 다르게 설정하고 싶다면 extends를 사용한 조건부 타입 사용

### extends 조건부 타입을 활용하여 개선하기

- useGetRegisteredList 함수의 반환 Data는 인자의 타입에 따라 정해짐
- 타입으로 "appcard" 또는 "card"를 받으면 카드 결제 수단 정보 타입인 PocketInfo<Card>를 반환. "bank"를 받는다면 PocketInfo<Bank>를 반환
- 조건부 타입을 활용하면 하나의 API 함수에서 타입에 따른 정확한 반환 타입을 추론할 수 있음
- 조건부 타입을 사용하여 PayMethodType을 개선해보자

  ```tsx
  type PayMethodType<T extends "card" | "appcard" | "bank"> = T extends
    | "card"
    | "appcard"
    ? Card
    : Bank;
  ```

- PqyMethodType이 "card"나 "appcard"일 경우, PaymethodInfo<Card> 타입을, 아닐 때는 PaMethodInfo<Bank> 타입을 반환
- 결제 수단 타입은 "card", "appcard", "bank"만 들어올 수 있게 extends를 한정자로 사용하여 제네릭에 넘겨오는 값을 제한함
- extends 활용 예시
  - 제네릭과 extends를 함께 사용하여 제네릭을 받는 타입을 제한. 개발자가 잘못된 값을 넘길 수 없기 때문에 휴먼 에러를 방지할 수 있음
  - extends를 활용해 조건부 타입을 설정. 반환 값을 사용자가 원하는 값으로 구체화 가능

### infer를 활용해서 타입추론하기

- extends로 조건을 서술하고, infer로 타입을 추론하는 방식

  ```tsx
  type UnpackPromise<T> = T extends Promise<infer K>[] ? K : any;

  const promises = [Promise.resolve("Mark", Promise.resolve(38))];
  type Expected = UnpackPromise<typeof promises>; // string | number
  ```

- UnpackPromise 타입은 제네릭을 T를 받아 T가 Promise로 래핑된 경우, K를 반환. 그렇지 않은 경우 any를 반환
- Promise<infer K> => Promise의 반환값을 추론해 해당값의 타입을 K로 한다는 의미
  <br><br>

## 5.2 템플릿 리터럴 타입 활용하기

- 자바스크립트의 템플릿 리터럴 문법을 사용해 특정 문자열에 대한 타입을 선언할 수 있는 기능.

```typescript
type HeadingNumber = 1 | 2 | 3 | 4 | 5;
type HeaderTag = `h${HeadingNumber}`;

type Direction =
  | "top"
  | "topLeft"
  | "topRight"
  | "bottom"
  | "bottomleft"
  | "bottomRight";

// 위 Direction 타입을 템플릿 리터럴을 사용하여 아래와 같이 가독성 좋고 재사용과 수정에 용이한 타입 선언 가능

type Vertical = "top" | "bottom";
type Horizon = "left" | "rigjt";

type Direction = Vertical | `${Vertical}${Capitalize<Horizon>}`;
```

<br><br>

## 5.3 커스텀 유틸리티 타입 활용하기

- 커스텀 유틸리티 타입을 활용하여 표현을 다양하게 하기

### 유틸리티 함수를 활용해 styled-components의 중복 타입 선언 피하기

- 컴포넌트에서 전달받은 props를 styled-components에 전달해 줄 때, props의 타입과 동일하게 전달해야 하는 경우가 대부분.

- 이 때, ts에서 제공하는 Pick, Omit 같은 유틸리티 타입을 잘 활용하여 코드를 간결하게 작성 가능

  ```typescript
  type StyledProps = Pick<Props, "height" | "color" | "isFull">;
  ```

- Pick 유틸리티 타입을 활용하여 props에서 필요한 부분만 선택하여 styled-components 컴포넌트의 타입을 정의하여 코드의 중복을 피할 수 있고, 유지보수를 편하게 할 수 있음.
- 이외에도 상속받는(하는) props 등에도 Pick, Omit 같은 유틸리티 타입 활용 가능

### PickOne 유틸리티 함수

- **ts에서 서로 다른 2개 이상의 객체를 유니온 타입으로 받을 때, 타입 검사가 제대로 진행되지 않는 이슈**가 있음.

- 이 문제를 해결하기 위한 것이 PickOne이라는 이름의 유틸리티 함수

  ```typescript
  type Card = {
    card: string;
  };

  type Account = {
    account: string;
  };

  function withdraw(type: Card | Account) {
    // ...
  }

  withdraw({ card: "samsung", account: "hana" });
  ```

- 위와 같은 경우, withdraw 함수의 매개변수로 Card 혹은 Account를 유니온 타입으로 받고 싶은데, 실제로 withdraw 인자에 card와 account 속성을 모두 받아도 에러가 발생하지 않는다.

- 왜 에러가 발생하지 않는 것인가?

  - 합집합의 개념이라 card, account 속성이 하나씩만 할당되거나, 둘 다 할당되는 경우에도 합집합의 범주에 들어가기 때문에 타입 에러가 발생하지 않음
  - 이런 문제를 해결하기 위해 ts에서는 식별할 수 있는 유니온 기법을 자주 활용

- 식별할 수 있는 유니온으로 객체 타입을 유니온으로 받기

  - "식별할 수 있는 유니온"은 각 타입에 type이라는 공통된 속성을 추가하여 구분 짓는 방법

    ```typescript
    type Card = {
      type: "card";
      card: string;
    };

    type Account = {
      type: "account";
      account: string;
    };

    function withdraw(type: Card | Account) {
      // ...
    }
    withdraw({ type: "card", card: "samsung" });
    withdraw({ type: "account", account: "hana" });
    ```

  - **위 방법은 식별할 수 있는 유니온인 "type"을 추가해야 함으로 번거로움**이 따라온다.
  - 이러한 번거로움을 방지하기 위해 **PickOne이라는 유틸리티 타입 적용**

- PickOne 커스텀 유틸리티 타입 구현하기

  - 구현하고자 하는 타입: "card" 또는 "account" 속성 하나만 존재하는 객체를 받는 타입
  - 방법 1: 하나의 속성이 들어왔을 때, 다른 타입을 옵셔널 + undefined 값으로 지정

    - 사용자가 의도적으로 undefined 값을 넣지 않는 이상, 원치 않는 속성에 값을 넣었을 때 타입 에러 발생
      ```typescript
      { account: string; card?: undefined; } | { account?: undefined; card: string; }
      ```
    - 타입이 추가된 경우
      ```typescript
      type PayMethod =
        | {
            account: string;
            card?: undefined;
            payMoney?: undefined;
          }
        | { account?: undefined; card: string; payMoney?: undefined }
        | { account?: undefined; card?: undefined; payMoney: string };
      ```
    - 즉 요점은, 개발자가 의도한 속성을 제외한 나머지 값은 옵셔널 타입 + undefined로 설정하여 원하고자 하는 속성만 받도록 구현할 수 있다는 것이다.
    - 이를 유틸리티 타입으로 구현하면 아래와 같다.
      ```typescript
      type PickOne<T> = {
        [p in keyof T]: Record<P, T[P]> &
          Partial<Record<Exclude<keyof T, P>, undefined>>;
      }[keyof T];
      ```
    - PickOne 살펴보기(2가지 타입으로 분리해서 생각)

      - One<"T"> -> 단, T에는 객체가 들어온다고 가정.

        ```typescript
        type One<T> = { [P in keyof T]: Record<P, T[P]> }[keyof T];
        ```

        - [P in keyof T]에서 T는 객체를 가정하기에 P는 T 객체의 키값을 의미
        - Record<P, T[P]>는 P 타입을 키로 가지고, value는 P를 키로 둔 T 객체의 값의 레코드 타입을 의미
        - 따라서 키는 객체의 키 모음이고, value는 해당 키의 원본 객체 T를 의미
        - 위에서 다시 [keyof T]의 키값으로 접근하기 때문에 최종 결과는 전달받은 T와 같다.

        ```typescript
        type Card = { card: string };
        const one: One<Card> = { card: "samsung" };
        ```

      - ExcludeOne<"T">

        ```typescript
        type ExcludeOne<T> = {
          [P in keyof T]: Partial<
            Record<Exclude<keyof T, P>, undefined>
          >[keyof T];
        };
        ```

        - [P in keyof T]에서 T는 객체를 가정하기에 P는 T 객체의 키값을 의미
        - Exclude<keyof T, P>는 T 객체가 가진 키 값에서 P 타입과 일치하는 키 값을 제외한다. 이 타입을 A라고 하자.
        - Record<A, undefined>는 키로 A 타입을, 값으로 undefined 타입을 갖는 레코드 타입. 즉, 전달 받은 객체 타입은 모드 { [key]: undefined } 형태. 이 타입을 B라고 하자
        - Partial<B.>는 B 타입을 옵셔널로 만든다. 따라서 { [key]?: undefined }와 같은 의미
        - 최종적으로 [P in keyof T]로 매핑된 타입에서 동일한 객체의 키 값인 [keyof T]로 접근하기 때문에 { [key]?: undefined } 타입이 반환됨

      - 결론적으로 얻고자 하는 타입은 속성 하나와 나머지는 옵셔널 + undefind 타입이기 때문에 PickOne 타입으로 표현할 수 있음.
        ```typescript
        type PickOne<T> = One<T> & ExcludeOne<T>;
        ```
      - 최종적으로, 전달된 T 타입의 1개의 키는 값을 가지고 있으며, 나머지 키는 옵셔널한 undefined 값을 가진 객체를 의미

      - PickOne 예시
        ```typescript
        type Card = { card: string };
        type Account = { account: string };
        const pickOne1: PickOne<Card & Account> = { card: "samsung" }; // O
        const pickOne2: PickOne<Card & Account> = { account: "hana" }; // O
        const pickOne3: PickOne<Card & Account> = {
          card: "samsung",
          account: undefined,
        }; // O
        const pickOne4: PickOne<Card & Account> = {
          card: undefined,
          account: "hana",
        }; // O
        const pickOne5: PickOne<Card & Account> = {
          card: "samsung",
          account: "hana",
        }; // X. 안 됨. 에러 발생
        ```

- PickOne 타입 적용하기

  - 위 예시에서 커스텀 유틸리티 타입으로 원하는 타입을 지정할 수 있게 됨.
  - 하지마 바로 커스텀 유틸리티 타입 함수를 작성하는 것은 쉽지 않음.
  - 따라서 커스텀 유틸리티 타입을 사용하기 위해서는 구현하고 싶은 타입을 작은 단위로 정확하게 쪼개어 단계적으로 구현하는 접근이 필요.

### NonNullable 타입 검사 함수를 사용하여 간편하게 타입 가드하기

- null을 가질 수 있는 값에서 null에 대한 처리를 해주는 것 -> 타입 가드의 패턴 중 하나
- if 조건문을 통한 타입 가드 방법이 아닌 is 키워드와 NonNullable 타입으로 타입 검사

- NonNullable 타입이란

  - 제네릭 타입으로 받는 T가 null 또는 undefined일 때, never 또는 T를 반환하는 타입
  - null이나 undefined가 아닌 경우를 제외할 수 있음

    ```typescript
    // NonNullable 타입
    type NonNullable<T> = T extends null | undefined ? never : T;

    // NonNullable 함수. 매개변수가 null이거나 undefined인 경우를 검증
    function NonNullable<T>(value: T): value is NonNullable<T> {
      // 매개변수 value가 null, undefined가 아닐 때, True를 반환
      return value !== null && value !== undefined;
    }
    ```

- Promise.all에 NonNullable 적용하기

  - AdCampaignAPI는 null을 반환할 수 있기에, shopAdCapaignList의 타입은 Array<AdCampaign[] | null>로 추론됨

  ```typescript
  const shopList = [
    { shopNo: 100, category: "chicken" },
    { shopNo: 101, category: "pizza" },
    { shopNo: 102, category: "noodle" },
  ];

  const shopAdCampaignList = await Promise.all(
    shopList.map((shop) => AdCampaignAPI.operation(shop.shopNo))
  );

  // 여기서 NonNullable을 사용하여 shopAdCapaignList의 타입을 Array<AdCampaign[]>로 추론할 수 있음
  const shopAds = shopAdCampaignList.filter(NonNullable);
  ```

  <br><br>

## 5.4 불변 객체 타입으로 활용하기

- 상수값을 객체에 담을 때, 열린 타입으로 설정하는 경우가 있다(왜냐하면 어떤 타입이 들어오거나 반환될 지 모르기 때문)
- 이 때, as const와, keyof를 사용하여 타입에 맞지 않는 타입이 들어올 경우, 컴파일 에러를 발생시켜, 컴파일 단계의 실수를 사전에 예방 가능

- Atom 단위 컴포넌트에서 theme style 객체 활용하기

  - Atom 단위의 작은 컴포넌트(Button, Header, Input 등)에서 폰트 크기, 폰트 색상, 배경 색상 등 다양한 환경에서 유연하게 사용될 수 있게 구현해야 하는데, 이것들은 대개 props로 전달되며, 변경에 취약
  - 따라서 theme 객체를 두고 관리

- keyof 연산자로 객체의 키값을 타입으로 추출하기

  ```typescript
  interface ColorType {
    red: string;
    green: string;
    blue: string;
  }

  type ColorKeyType = keyof ColorType; // 'red' | 'green' | 'blue'
  ```

- typeof 연산자로 값을 타입으로 다루기

  - 우선 keyof 연산자는 객체 타입을 받는다.
  - 따라서 객체의 키값을 타입으로 다루려면 값 객체를 타입으로 변환해야 한다.
  - js에서는 typeof가 타입을 추출하기 위한 연산자.
  - ts에서는 typeof가 변수 혹은 속성의 타입을 추론하는 역할로 사용
  - typeof를 사용한 colors 객체의 타입을 추론한 예시

    ```typescript
    const color = {
      red: "#F45452",
      green: "#0C952A",
      blue: "#1A7CFF",
    };

    type ColorsType = typeof colors;
    /**
     {
        red: string;
        green: string;
        blue: string
     }
     */
    ```

- 객체의 타입을 활용해서 컴포넌트 구현하기

  - color, backgroundColor, fontSize의 타입을 theme 객체에서 추출하는 코드

    ```typescript
    import { FC } from 'react';
    import styled from 'styled-components';
    const colors = {
      black: "#000000",
      gray: "#222222",
      white: "#FFFFFF",
      mint: "#2AC1BC",
    };

    const theme = {
      colors: {
        default: colors.gray,
        ...colors
      },
      backgroundColor: {
        default: colors.white,
        gray: colors.gray,
        mint: colors.mint,
        black: colors.black,
      },
      fontSize: {
        default: "16px",
        small: "14px",
        large: "18px",
      }
    };

    type ColorType = typeof keyof theme.colors;
    type BackgroundColorType = typeof keyof theme.backgroundColor;
    type FontSizeType = typeof keyof theme.fontSize;

    interface Props {
      color?: ColorType;
      backgroundColor?: BackgroundColorType;
      fontSize?: FontSizeType;
      onClick: (event: React.MouseEvent<HTMLButtomElement>) => void | Promise<void>;
    }
    ```

  - 이 Props 인터페이스를 사용하는 Atom 컴포넌트에서 theme 객체를 통해 따로 관리되는 ColorType, BackgroundColorType, FontSizeType 값 이외의 것들이 들어오면 타입 오류가 발생되기에, 오류 발생을 사전에 막아준다.
  - **따라서 객체의 키값을 추출한 타입을 활용**하면 객체에 접근할 때 ts의 도움을 받아 실수를 방지할 수 있음.
    <br><br>

## 5.5 Record 원시 타입 키 개선하기

### 무한한 키를 집합으로 가지는 Record

- 객체 키 선언시, 키가 어떤 값인지 명확하지 않다면, Record의 키를 string이나 number 같은 원시 타입으로 타입을 명시하기도 하는데 이 때, 예상치못한 런타임 에러 야기할 수도 있음.

  ```typescript
  type Category = string;
  interface Food {
    name: string;
    // ...
  }
  // Record의 키(음식분류)는 string(한식, 일식), value는 Food[]인 상태
  const foodByCategory: Record<Category, Food[]> = {
    한식: [{ name: "제육덮밥" }, { name: "뚝불" }],
    일식: [{ name: "초밥" }, { name: "텐동" }],
  };

  foodByCategory["양식"]; // ts 상에서 Food[]로 추론
  foodByCategory["양식"].map((food) => console.log(food.name)); // ts 상에서 오류 발생X. 하지만 런타임에서 foodByCategory["한식"]가 undefined가 되어 런타임 에러 발생
  ```

- 위와 같은 경우 foodByCategory["양식"]?.map과 같은 옵셔널 체이닝으로 런타임 에러를 방지할 수 있지만 매번 옵셔널 체이닝을 사용하여 검사하는 것은 비효율적.

### 유닛 타입으로 변경하기

- 키가 만약 유한한 집합이라면 유닛 타입(오직 하나의 정확한 값을 가지는 타입)을 사용할 수 있음

  ```typescript
  type Category = "한식" | "일식";

  interface Food {
    name: string;
    // ...
  }

  const foodByCategory: Record<Category, Food[]> = {
    한식: [{ name: "제육덮밥" }, { name: "뚝불" }],
    일식: [{ name: "초밥" }, { name: "텐동" }],
  };
  ```

- 이처럼 키가 유한하다면 유닛타입으로 키의 타입을 설정. 만약 키가 무한하다면?

### Partial을 활용하여 정확한 타입 표현하기

- Key가 무한한 상황에서 Partial을 사용하면 해당 값이 undefined일 수 있는 상태임을 표현할 수 있음!!
- Partial을 사용하여 PartialRecord 타입ㅇㄹ 선언하고, 객체 선언 때, 이를 사용

  ```typescript
  type PartialRecord<K extends string, T> = Partial<Record<K, T>>;
  type Category = string;

  interface Food {
    name: string;
    // ...
  }

  const foodByCategory: PartialRecord<Category, Food[]> = {
    한식: [{ name: "제육덮밥" }, { name: "뚝불" }],
    일식: [{ name: "초밥" }, { name: "텐동" }],
  };

  foodByCategory["양식"].map((food) => console.log(food.name)); // Object is possibly 'undefined'
  ```

- 위와 같이 타입이 정해지지 않은 키가 무한일 경우, Partial은 추후에 타입 지정되지 않은 키값이 들어왔을 때, 그에 대한 value 값이 "undefined"일 수도 있다는 메시지를 남겨주고, 개발자는 이 메시지를 보고, undefined에 대한 분기처리를 할 수 있다.
