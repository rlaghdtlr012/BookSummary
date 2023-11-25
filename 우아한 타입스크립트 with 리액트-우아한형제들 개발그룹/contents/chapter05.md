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
