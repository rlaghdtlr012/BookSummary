## 4.1 타입 확장하기

- ts에서는 interface와 type을 사용하여 타입을 정의
- extends, 교차 타입, 유니온 타입을 사용하여 타입을 확장

### 타입 확장의 장점

- 타입 확장의 가장 큰 장점은 코드 중복을 줄일 수 있다는 것. + 확장성

  ```typescript
  // 타입 확장의 예시
  interface BaseMenuItem {
    itemName: string | null;
    itemImageUrl: string | null;
    itemDiscountAmount: number;
    stock: number | null;
  }

  // 카트 아이템은 BaseMenuItem의 요소들을 다 갖고 있을 수 있고
  // extends 키워드를 통해 코드의 중복을 줄일 수 있음
  interface BaseCartItem extends BasMenuItem {
    quantity: number;
  }

  // type에서의 타입 확장
  type BaseMenuItem = {
    itemName: string | null;
    itemImageUrl: string | null;
    itemDiscountAmount: number;
    stock: number | null;
  };

  type BaseCartItem = {
    quantity: number;
  } & BaseMenuItem;
  ```

### 유니온 타입

- 2개 이상의 타입을 조합해서 사용하는 방법(합집합)
  ```typescript
  type MyUnion = A | B;
  // MyUnion은 타입 A와 B의 합집합. 집합 A의 모든 원소는 MyUnion의 원소이며 집합 B의 모든 원소도 MyUnion의 원소이다.
  ```
- 단, 유니온 타입으로 선언된 값은 **유니온 타입에 포함된 모든 타입이 공통으로 갖고 있는 속성**에만 접근할 수 있다.

  ```typescript
  interface CookingStep {
    orderId: string;
    price: number;
  }

  interface DeliveryStep {
    orderId: string;
    time: number;
    distance: string;
  }

  function getDeliveryDistance(step: CookingStep | DeliveryStep) {
    return step.distance;
    // Error!!! distance 속성은 "CookingStep | DeliveryStep" 타입에 존재하지 않는다.
  }
  ```

### 교차 타입

- 기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입으로 만드는 것

  ```typescript
  // 유니온 타입의 코드에서 추가로 이어지는 내용
  // BaedalProgress는 CookingStep과 DeliveryStep 타입을 합쳐 모든 속성을 가진 단일 타입이 된다.
  type BaedalProgress = CookingStep & DeliverStep;

  function logBaedalinfo(progress: BaedalProgress) {
    console.log(`주문 금액: $progress.price}`);
    console.log(`주문 금액: $progress.distance}`);
  }
  ```

- progress는 CookingStep이 가지고 있는 속성과 DeliveryStep이 가지고 있는 속성을 모두 포함.

### 배달의민족 메뉴 시스템에 타입 확장 적용하기

- "1인분", "족발,보쌈", "찜,탕,찌개". "돈까스,회,일식", "피자". 다음과 같은 서비스의 메뉴 목록을 바탕으로 Menu 인터페이스 표현

  ```typescript
  /**
   * 메뉴에 대한 타입
   * 메뉴 이름과 메뉴 이미지에 대한 정보를 담고 있음
   */
   interface Menu {
    name: string;
    image: string;
   }

   // Menu 인터페이스를 바탕으로 리액트 코드 작성
   function MainMenu() {
    // Menu 타입을 원소로 갖는 배열
    const menuList = Menu[] = [{name: "1인분," image: "1인분.png"}];
    return (
      <ul>
        {menuList.map((menu) => {
          <li>
            <img src ={menu.image} />
            <span>{menu.name}</span>
          </li>
        })}
      </ul>
    )
   }
  ```

- 이 상태에서 다음 두 가지 요구 사항이 추가되었다고 가정.

  1. 특정 메뉴를 길게 누르면 gif 파일이 재생되어야 한다.
  2. 특정 메뉴는 이미지 대신 별도의 텍스트만 노출되어야 한다.

  ```typescript
  /**
   * 요구사항에 맞게 변경된 코드
   * 방법 1: 인터페이스에 속성 추가
   */
  interface Menu {
    name: string;
    image: string;
    gif?: string;
    text?: string;
  }

  /**
   * 방법 2: 타입 확장 활용
   */
  interface Menu {
    name: string;
    image: string;
  }

  interface SpecialMenu extends Menu {
    gif: string;
  }
  interface PackageMenu extends Menu {
    text: string;
  }

  /**
   * 각 배열은 서버에서 받아온 응답 값이라고 가정
   */

  const menuList = [
    { name: "찜", image: "찜.png"},
    { name: "찌개", image: "찌개.png"},
    { name: "회", image: "회.png"},
  ];

  const specialMenuList = [
    { name: "돈까스", image: "돈까스.png", gif: "돈까스.gif"},
    { name: "피자", image: "피자.png", gif: "피자.gif"},
  ];

  const packageMenuList = [
    { name: "1인분", image: "1인분.png", text: "1인분.gif"},
    { name: "족발", image: "족발.png", text: "족발.gif"},
  ];

  // [방법 1]의 경우
  // 각 메뉴 목록을 Menu[] 하나로 표현할 수 있음
  menuList: Menu[]; // 가능
  specialMenuList: Menu[]; // 가능
  packageMenuList: Menu[]; // 가능

  // [방법 2]의 경우
  menuList: Menu[]; // 가능

  specialMenuList: Menu[]; // 불가능
  specialMenuList: SpecialMenu[]; // 가능

  packageMenuList: Menu[]; // 불가능
  packageMenuList: PackageMenu[]; // 가능
  ```

- 결과적으로 주어진 타입에 무분별하게 속성을 추가하여 사용하는 것보다 타입을 확장해서 사용하는 것이 좋음
- 적절한 네이밍을 사용해서 타입의 의도를 명확히 표현할 수 있고, 코드 작성 단계에서 예기치 못한 버그도 예방 가능
  <br><br>

## 4.2 타입 좁히기

- 변수 또는 표현식의 타입 범위를 더 작은 범위로 좁혀나가는 과정
- 이를 통해 더 정확하고 명시적인 타입 추론이 가능. 복잡한 타입을 작은 범위로 축소하여 타입 안정성을 높일 수 있음

### 타입 가드에 따라 분기 처리하기

- 스코프에서 타입에 따른 분기처리를 하고 싶을 때,
- if문을 사용하여 분기처리를 하면 될 거 같지만 (js로)컴파일 시, 타입 정보는 모두 제거되어 런타임에 존재하지 않게 됨.
- 즉, 컴파일해도 타입 정보가 사라지지 않는 방법을 사용해야 함.(= 런타임에서도 타입이 유효한 방법을 찾아야 함) -> **타입가드**
- 런타임에 유효하다 == 타입스크립트 뿐만 아니라 자바스크립트에서도 사용할 수 있는 문법이어야 한다.

### 원시 타입을 추론할 때: typeof 연산자 활용하기

- typeof A === B 조건으로 분기 처리시, 해당 분기 내에서 A의 타입이 B로 추론됨
- 이는 js 타입 시스템에만 대응될 수 있어서, null이나 배열, Object 타입등의 복잡한 타입을 검증하는 데 한계가 있음.
- 따라서 typeof는 주로 원시 타입을 좁히는 용도로만 사용
- typeof로 검사할 수 있는 타입 목록: string, number, boolean, undefined, object, function, bigint, symbol
  ```typescript
  const replaceHyphen: (date: string | Date) => string | Date = (date) => {
    if (typeof date === "string") {
      // 이 분기에서는 date의 타입이 string으로 추론됨
      return date.replace(/-/g, "/");
    }
    return date;
  };
  ```

### 인스턴스화 된 객체 타입을 판별할 때: instanceof 연산자 활용하기

- instanceof 연산자는 인스턴스화된 객체 타입을 판별하는 타입 가드로 사용 가능.
- A instanceof B 형태로 사용.
- A: 타입을 검사할 대상 변수, B: 특정 객체의 생성자

  ```typescript
  // instanceof를 사용한 객체의 타입 분기
  if (event.target instanceof HTMLInputElement) {
    // HTMLInputElement의 인스턴스인 경우에만 사용할 수 있는 blur 메서드
    event.target.blur();
  }
  ```

### 객체의 속성이 있는지 없는지에 따른 구분: in 연산자 활용하기

- in 연산자는 객체에 속성이 있는지 확인한 다음 T/F 반환
- 속성이 있는지 없는지에 따라 객체 타입을 구분 가능
- 객체에 속성이 있는지 없는지를 검사하는 것이기에 속성 값을 undefined로 할당한다고 해서 false가 반환되는 것은 아님
- delete 연산자를 사용하여 객체 내부에서 해당 속성을 제거해야만 false를 반환

  ```typescript
  interface BasicNoticeDialogProps {
    noticeTitle: string;
    noticeBody: string;
  }

  interface NoticeDialogWithCookieProps extends BasicNoticeDialogProps {
    cookieKey: string;
    noForADay?: boolean;
    neverAgain?: boolean;
  }

  export type NoticeDialogProps = BasicNoticeDialogProps | NoticeDialogWithCookieProps;

  // 객체 속성이 있는지 없는지에 따른 분기문 설정
  const NoticeDialog: React.FC(NoticeDialogProps) = (props) => {
    // 객체 속성값의 유무에 따른 분기
    if ("cookieKey" in props) {
      return ~~;
    }
  }
  ```

### is 연산자로 사용자 정의 타입 가드 만들어 활용하기

- 직접 타입가드 만들어서 사용
- A is B 형식으로 작성
- A는 매개변수 이름, B는 타입
  ```typescript
  // string 타입의 매개변수가 destinationCodeList 배열의 원소 중 하나인지를 검사하여 boolean을 반환하는 함수
  const isDestinationCode = (x: string): x is DestinationCode => {
    destinationCodelist.includes(x);
  };
  ```

## 4.3 타입 좁히기 - 식별할 수 있는 유니옴(Discriminated Unions)

### 에러 정의하기

- 배민에서는 사용자가 필요한 값을 올바르게 입력했는지를 확인하는 유효성 검사를 진행.
- 입력값을 검사하는 과정에서 입력의 에러를 크게 "텍스트 에러", "토스트 에러", "얼럿 에러"로 분류한다.
- 각 에러는 에러 코드와 메시지가 다르고 각각의 에러 처리를 어떻게 할 것인가를 다양하게 처리해야 한다.

  ```typescript
  type TextError = {
    errorCode: string;
    errorMessage: string;
  };
  type ToastError = {
    errorCode: string;
    errorMessage: string;
    toastShowDuration: number; // 토스트를 띄우줄 시간
  };
  type AlertError = {
    errorCode: string;
    errorMessage: string;
    onConfirm: () => void; // 얼럿 창의 확인 버튼을 누른 뒤 액션
  };

  type ErrorFeedbackType = TextError | ToastError | AlertError;
  const errorArr: ErrorFeedbackType[] = [
    { errorCode: "100", errorMessage: "텍스트 에러" },
    { errorCode: "200", errorMessage: "토스트 에러", toastShowDuration: 3000 },
    { errorCode: "300", errorMessage: "얼럿 에러", onConfirm: () => {} },
  ];
  ```

- errorArr를 정의함으로써 다양한 에러 객체를 관리할 수 있게 됨.
- toastShowDuration 필드와 onConfirm 필드를 모두 가지는 객체에 대해서는 타입 에러를 뱉게 하기 위해서는?
  ```typescript
  cosnt errorArr: ErrorFeedbackType[] = [
    {
      errorCode: "999",
      errorMessage: "잘못된 에러",
      toastShowDuration: 3000,
      onConfirm: () => P{,}
    },
  ];
  // 이런 경우 에러가 발생되어야 하지만
  // js의 덕타이핑으로 인해 별도의 타입 에러를 발생시키지 않음
  // 이는 추후 개발함에 있어서 큰 문제를 야기시킬 수 있음.
  ```

### 식별할 수 있는 유니온

- 따라서 에러 타입을 구분할 방법이 필여함
- 식별할 수 있는 유니온이란? 타입 간의 구조 호환을 막기 위해 타입마다 구분할 수 있는 **판별자(태그)를 달아주어 포함 관계를 제거**하는 것

  ```typescript
  // 에러 타입 정의를 통한 타입 구분
  type TextError = {
    errorType: "TEXT";
    errorCode: string;
    errorMessage: string;
  };
  type ToastError = {
    errorType: "TOAST";
    errorCode: string;
    errorMessage: string;
    toastShowDuration: number; // 토스트를 띄우줄 시간
  };
  type AlertError = {
    errorType: "ALERT";
    errorCode: string;
    errorMessage: string;
    onConfirm: () => void; // 얼럿 창의 확인 버튼을 누른 뒤 액션
  };

  type ErrorFeedbackType = TextError | ToastError | AlertError;
  const errorArr: ErrorFeedbackType[] = [
    { errorType: "TEXT", errorCode: "100", errorMessage: "텍스트 에러" },
    {
      errorType: "TOAST",
      errorCode: "200",
      errorMessage: "토스트 에러",
      toastShowDuration: 3000,
    },
    {
      errorType: "ALERT",
      errorCode: "300",
      errorMessage: "얼럿 에러",
      onConfirm: () => {},
    },
    {
      errorType: "TOAST",
      errorCode: "900",
      errorMessage: "얼럿 에러",
      toastShowDuration: 3000,
      onConfirm: () => {},
    }, // 에러 발생!!!!!!!
    // "TOAST" 에러에는 onConfirm에 관한 속성이 포함되어 있지 않다!!
  ];
  ```

- 우리가 처음에 기대했던 정확하지 않은 에러 객체에 대한 타입 에러가 발생하는 것을 확인 가능

### 식별할 수 있는 유니온 판별자 선정

- 위 코드에서 errorType이 식별할 수 있는 유니온 판별자라고 할 수 있다.
- 식별할 수 있는 유니온 판별자는 유닛 타입(다른 타입으로 쪼개지지 않는 오직 하나의 정확한 값을 가지는 타입. ex: null, undefined, true, 1 등)으로 선언되어야 정상적으로 동작함.
- 식별할 수 있는 유니온 판별자로 사용할 수 있는 타입

  1. 리터럴 타입이어야 함("TOAST", "ALERT", 3과 같이 정확히 특정한 값을 가지는 타입)
  2. 판별자로 선정한 값에 적어도 하나 이상의 유닛 타입이 포함되어야 하며, 인스턴스화 할 수 있는 타입은 포함되지 않아야 한다.

  ```typescript
  interface A {
    value: "A"; // 가능한 unit type
    answer: 1;
  }

  interface B {
    value: string; // 불가능한 unit type!!
    answer: 2;
  }

  interface C {
    value: Error; // 인스턴스화 할 수 있는 타입이므로 불가능한 unit type!!
    answer: 3;
  }
  ```

- 판별자가 value 일 때는 interface A의 value: "A" 만이 유일한 유닛 타입이 된다.
- 판별자가 answer 일 때는 모두 다 리터럴한 타입이므로 모두 유일한 유닛 타입이 된다.

## 4.4 Exhaustiveness Checking으로 정확한 타입 분기 유지하기

- Exhaustiveness: 사전적으로 철저함, 완전함을 의미
- 즉, Exhaustiveness Checking은 모든 케이스에 대해 철저하게 타입을 검사(강제)하는 것을 의미

### 상품권

- 상품권 가격에 따라 상품권의 이름을 반환하는 함수
  ```typescript
  type ProdctPrice = "10000" | "20000";
  const getProductName = (productPrice: ProdctPrice): string => {
    if (productPrice === "10000") return "배민상품권 1만 원";
    if (productPrice === "20000") return "배민상품권 2만 원";
    else {
      return "배민상품권";
    }
  };
  ```
- 위와 같은 코드에 만약 ProductPrice가 50000원인 케이스가 추가된다고 가정해보자.
- 그렇다면 type에 50000을 추가해야 하고, 이에 따라 50000원 케이스에 대한 분기문도 따로 작성해주어야 한다.
- 이 과정에서 코드를 누락하거나 실수하는 경우가 생길수도 있음. 그것을 방지하는 것이 exhaustive checking(**모든 타입에 대한 타입 검사 강제**)
- 모든 타입에 대한 타입 검사를 하지 않았을 경우 에러 발생

  ```typescript
  // exhaustive checking을 적용한 코드
  type ProdctPrice = "10000" | "20000" | "50000";
  const getProductName = (productPrice: ProdctPrice): string => {
    if (productPrice === "10000") return "배민상품권 1만 원";
    if (productPrice === "20000") return "배민상품권 2만 원";
    // 실수로 50000에 대한 분기처리를 누락했다고 가정
    // if (productPrice === "50000") return "배민상품권 5만 원";
    else {
      exhaustiveCheck(productPrice); // Error: String 타입의 인자는 never 타입에 할당될 수 없다.
      return "배민상품권";
    }
  };

  const exhausitiveCheck = (param: never) => {
    throw new Error("type Error");
  };
  ```

- 위 코드는 productPrice가 50000인 경우의 분기를 처리하지 않았기 때문에 exhaustiveCheck에서 에러를 발생시킨다.
- **모든 타입에 대한 분기처리를 하지 않아 줬을 때, 컴파일타임 에러가 발생하는 것을 exhaustiveCheck라고 한다.**
- exhaustiveCheck는 예상치못한 런타임 에러 방지 및 요구사항 변경시, 생길 수 있는 위험성을 줄일 수 있음.
