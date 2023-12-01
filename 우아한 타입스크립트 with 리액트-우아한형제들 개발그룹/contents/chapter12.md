## 12.1 앰비언트 타입 활용하기

### 앰비언트 타입 선언

- .d.ts 확장자를 가진 파일에서의 타입 선언
- .d.ts 확장자를 가진 파일에서는 타입 선언만 가능. **값을 표현할 수 없음**
- 앰비언트 타입선언이란 값을 포함한 일반적인 선언과 구별하기 위해 .d.ts 확장자를 가진 파일에서 하는 타입 선언
- declare 키워드를 사용하여 컴파일러에게 어떤 것의 존재 여부를 명시해주는 역할. 단순히 존재 여부만 알려준다
- .ts, .js 파일 외의 파일(ex: .png 등)을 import 할 때, 에러가 발생한다. 타입스크립트는 기본적으로 .ts와 .js 파일만 이해하고, 그 외의 다른 파일 형식은 인식하지 못함
- 이런 상황에서 declare 키워드를 통해 모듈 선언하여 컴파일러에게 미리 정보를 제공함으로써 에러 수정 가능

  ```ts
  declare module "*.png" {
    const src: string;
    export default src;
  }
  ```

- 자바스크립트로 작성된 라이브러리

  - 만약 js로 작성된 npm 라이브러리가 있다면, 이는 타입 선언이 없기애 import한 모듈들의 타입이 모두 any로 추론될 것.
  - 만약 tsconfig.json 파일에서 any 사용을 막아놨다면 프로젝트가 빌드되지 않을 것.
  - 이 때, 앰비언트 타입 선언 사용하면 타입스크립트가 이를 확인하고 타입 검사를 진행.

- 타입스크립트로 작성된 라이브러리

  - 타입스크립트로 작성된 라이브러리라도 js 파일과 .d.ts 파일로 나워서 배포되는 것이 일반적.
  - tsconfig.json 파일의 declaration을 true로 설정하면 ts 컴파일러는 자동으로 .d.ts 파일을 생성

- global 선언을 통해 자바스크립트로 선언된 전역 변수를 ts 컴파일러에게 알려줄 때

  - 전역 객체인 Window에 변수나 함수를 추가하여, 전역적으로 변수가 함수를 지정하는 경우도 있음
  - 하지만 이 경우, ts에서는 사용자가 정의한 Window 객체의 속성을 모르기 때문에 Window 객체의 타입이 존재하지 않는다고 판단. -> 에러 발생
  - 이 때, global namespace에 있는 Window 객체에 해당 속성이 정의되어 있다는 것을 알리기 위해 앰비언트 타입 선언을 사용할 수 있음
    `ts
declare global {
  interface Window {
    deviceId: string | undefined;
    appVersion: string;
  }
}
`
    <br><br>

### 앰비언트 타입 선언 시 주의점

- 타입스크립트로 만드는 라이브러리에는 불필요

  - tsconfig.json의 declaration을 true로 설정하면 타입스크립트 컴파일러가 .d.ts 파일을 자동으로 생성해줌

- 전역으로 타입을 정의하여 사용할 때 주의점
  - 서로 다른 라이브러리가 같은 이름의 앰비언트 타입 선언을 할 경우, 충돌 발생
    <br><br>

### 앰비언트 타입 선언을 잘못 사용했을 때의 문제점

- 앰비언트 타입 선언을 할 때, 앰비언트 타입의 의존성 관계가 보이지 않기 때문에 변경에 의한 영향 범위를 개발자가 파악하기 어려움
- 왜냐하면 앰비언트 타입은 명시적인 import나 export 없이 코드 전역에서 사용할 수 있기 때문
- 앰비언트 변수 선언은 프로그램 전역으로 영향을 끼치기에 디버깅의 어려움을 줌
  <br><br>

### 앰비언트 타입 활용하기

- 타입을 정의하여 임포트 없이 전역으로 공유
- declare namespace를 활용하기

  - Node.js에서 .env 파일을 사용할 때, declare namespace를 활용하여 process.env로 설정값을 손쉬게 불러오고 환경변수의 자동완성 기능 사용 가능

    ```ts
    // 1. namespace를 활용하여 process.env 타입을 보강하지 않은 경우
    API_URL = "localhost:8080";
    log(process.env.API_URL as string);

    // 2. namespace를 활용하여 process.env 타입을 보강한 경우
    API_URL = "localhost:8080";

    declare namespace NodeJS {
      interface ProcessEnv {
        readonly API_URL: string;
        readonly API_INTERNAL_URL: string;
        // ...
      }
    }

    log(process.env.API_URL); // 알아서 타입 추론 가능
    ```

## 12.2 스크립트와 설정 파일 활용하기

타입스크립트 프로젝트에서 스크립트와 tsconfig를 잘 활용하면 개발 생산성을 높일 수 있음

### 스크립트 활용하기

- 실시간으로 타입검사하기

  - 프로젝트의 규모가 커질수록 에디터에서 타입을 감지하는 것이 느려짐
    ```bash
    yarn tsx -noEmit -incremental -w
    ```
  - noEmit: 자바스크립트로 된 출력 파일을 생성하지 않도록 설정
  - incremental 옵션: 증분 컴파일을 활성화하여 컴파일 시간 단축
  - w: 파일 변경을 실시간 모니터링

  - c.f) 증분 컴파일: 매번 모든 대상을 컴파일 하는 것이 아닌 변경 사항만 부분적으로 컴파일 하는 것. 컴파일 시간 줄이기 가능

- 타입 커버리지 확인하기

  - 타입스크립트에서 any를 남발하지 않고, 프로젝트가 얼마나 타입스크립트 통제하에 돌아가고 있는 지를 정량적으로 파악하는 명령어
    ```bash
    npx type-coverage -detail
    ```
  - 현재 프로젝트의 타입 커버리지 및 any를 사용하고 있는 변수의 위치를 알려줌
  - 전체 변수의 몇 %가 타입이 지정되었는지, 몇 개의 변수가 any로 선언된 것인지 알 수 있음.
  - ts로 마이그레이션 작업중인 프로젝트를 다룰 때, 커버리지를 체크함으로써 더 나은 퀄리티로 리팩토링 하기 위한 기반 마련이 되는 정량적 지표

### 설정 파일 활용하기

- 타입스크립트 컴파일 속도 높이기

  - tsconfig의 incremental 속성을 활용하여 ts 컴파일 속도를 높일 수 있음
  - incremental 속성 true -> 증분 컴파일이 활성화 되어 컴파일
    ```typescript
    {
      "compilerOptions": {
        ...
        incremental: trus
      }
    }
    ```
