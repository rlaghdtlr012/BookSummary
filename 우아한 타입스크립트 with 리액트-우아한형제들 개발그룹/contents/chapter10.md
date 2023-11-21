## 10.1 상태관리

### 상태(state)

- 상태란? 렌더링에 영항을 줄 수 있는 동적인 데이터 값
- 리액트 공식 문서에서의 상태: 렌더링 결과에 영향을 주는 정보를 담은 순수 자바스크립트 객체
- 리액트 앱 내의 상태: 지역상태, 전역상태, 서버상태로 분류

#### 지역상태(Local State)

- 컴포넌트 내부에서 사용되는 상태
- useState 훅을 가장 많이 사용. 때에 따라 useReducer와 같은 훅도 사용

#### 전역 상태(Global State)

- 앱 전체가 공유하는 상태
- 여러 개의 컴포넌트가 전역 상태를 사용할 수 있음
- 상태가 변경되면 컴포넌트들도 업데이트
- prop drilling 문제를 피하고자 지역 상태를 전역 상태로 공유할 수 있음
- c.f) props drilling: props를 통해 값을 전달하기 위해 값이 필요하지도 않은데도, 중간 과정의 컴포넌트에 props를 전달하는 과정. 컴포넌트가 많아지면 Prop drilling으로 인해 코드가 복잡해질수도 있음.

#### 서버 상태

- 외부 서버에 저장해야 하는 상태들
- 사용자 정보, 글 목록 등
- UI 상태와 결합하여 관리, 로딩 여부나 에러 상태등을 포함
- react-query, SWR 등으로 관리

### 상태를 잘 관리하기 위한 가이드

- 상태가 많아지면 복잡성 증가
- 또한 상태가 변경되면 컴포넌트에서 리렌더링이 발생
- 따라서 유지보수 및 성능 관점에서 상태의 개수를 최소화 하는 것이 바람직
- 가능하다면 상태가 없는 Stateless 컴포넌트를 활용하는 게 좋음
- 상태를 정의할 때 고려해야할 2가지
  - 시간이 지나도 변하지 않는다면 상태가 아님
  - 파생된 값은 상태가 아님

#### 시간이 지나도 변하지 않는다면 상태가 아니다

- 시간이 지나도 변하지 않는 값을 단순 상수 변수에 저장하면? -> 렌더링 될 때마다 새로운 객체 인스턴스가 생성되기에 불필요한 리렌더링이 자주 발생할 수 있음
- 따라서 컴포넌트 라이프사이클 내에서 마운트 될 대 인스턴스가 생성, 렌더링 될 때마다 동일한 객체 참조가 유지되도록 구현해야함
- useMemo: 객체의 참조 동일성을 유지하기 위한 하나의 메모이제이션 방법
  ```typescript
  const store = useMemo(() => new Store(), []);
  ```
- 하지만 useMemo가 반드시 객체의 참조 동일성을 보장하는 것은 아님
- 리액트 공식 문서에서는 useMemo의 메모이제이션은 의미상으로 보장된 것은 아님. 따라서 성능 향상의 용도로만 사용할 것을 권장
- 따라서 useMemo 없이 동작하도록 먼저 구현 후, 추후에 성능 개선을 위해 useMemo를 사용하는 것을 권장

  ```typescript
  // 렌더링마다 인스턴스가 생성. 따라서 초기값 설정에 큰 비용 소모
  useState(new Store());

  // 콜백을 통해 초기값을 계산하는 것이 더 나음
  useState(() => new Store());
  // or
  const [store] = useState(() => new Store());
  ```

- 하지만 useState도 추후에 상태가 변하여 리렌더링이 일어난다는 관점에서 객체 참조에 있어 적합한 훅은 아님
- **useRef가 동일한 객체 참조 유지의 목적으로 가장 적합**
- useRef(new Store()); 도 마찬가지로 초기 렌더링마다 인스턴스를 생성하기에 이 방법은 부적합
- 아래와 같은 방법이 같은 객체 참조에 가장 적합

  ```typescript
  const store = useRef<Store>(null);

  if (!store) {
    store.current = new Store();
  }
  ```

#### 파생된 값은 상태가 아니다

- 부모에게서 전달받을 수 있는 props이거나 기존 상태에서 계산될 수 있는 값은 상태가 아니다.
- Single Source Of Truth는 어더한 데이터도 단 하나의 출처에서 생성하고 수정해야 한다는 원칙을 의미하는 방법론
- 파생된 값을 상태로 관리하게 되면, 상태를 관리하는 출처가 달라지기 때문에 SSOT를 위반.
- 따라서 부모로부터 물려받은 props는 상태가 아님
  ```typescript
  const UserEmail = ({ initialEmail }) => {
      const [email, setEmail] = useState(initialEmail);
      const onChangeEmail = (event) => {
          setEmail(event.target.value);
      };
      return (
          <div>
              <input type="text" value={email} onChange=(onChangeEmail) />
          </div>
      )
  }
  ```
- 위 코드는 onChangeEmail에 따라 email 값이 변하지 않음
  - why? email은 useState를 통해 처음 한 번만 초기값이 설정이 되고, 그 이후에는 독자적으로 관리되기 때문
- 그리고 내부 데이터를 상태화 동기화하기 위해 useEffect 사용하면 안됨.
  - 왜냐하면 개발자가 상태를 추적하기 힘들기 때문
- 그렇다면 해결책은?
  - 두 철처 간의 데이터를 동기화하기 보다는 단일 출처에서 데이터를 사용하도록 변경해야함
  - 일반적으로 리액트에서는 상태를 상위 컴포넌트로 끌어올림
  ```typescript
  // 해결책: email이라는 상태의 관리 및 출처를 상위 컴포넌트로 끌어올려 출처를 하나로 통일
  // 하위 컴포넌트에서는 이를 props로 받아서 사용
  const UserEmail = ({ email, setEmail }) => {
      const onChangeEmail = (event) => {
          setEmail(event.target.value);
      };
      return (
          <div>
              <input type="text" value={email} onChange=(onChangeEmail) />
          </div>
      )
  }
  ```

#### useState vs useReducer 어떤 것을 사용해야 할까

- useState 대신 useReducer 사용을 권장하는 두 가지 경우
  - 다수의 하위 필드를 포함하고 있는 복잡한 상태 로직을 다룰 때
  - 다음 상태가 이전 상태에 의존적일 때
- useReducer는 '무엇을 변경할지'와 '어떻게 변경할지'를 분리하여 dispatch를 통해 어떤 작업을 할지를 액션으로 넘기고, reducer 함수 내에서 상태를 업데이트 하는 방식.

  ```typescript
  // boolean 상태를 토글하는 액션만 사용하는 경우, useState보다 useReducer를 사용
  // Before
  const [fold, setFold] = useState(true);
  const toggleFold = () => {
    setFold((prev) => !prev);
  };

  // after
  const [fold, toggleFold] = useReducer((v) => !v, true);
  ```

  ### 전역 상태 관리와 상ㅐ 관리 라이브러리

  - 상태는 사용하는 곳과 최대한 가까워야 하며 사용 범위르 제한해야만 한다.
  - 전연 상태를 사용하는 방법
    - 컨텍스트 API + useState or useReducer
    - 외부 상태 관리 라이브러리(Redux, MobX, Recoil)

- Context API

  - 다른 컴포넌트들과 데이터를 쉽게 공유하기 위한 목적으로 제공되는 API
  - prop drilling 같은 문제 해결 도구
  - 전역적으로 공유해야 하는 데이터를 컨텍스트로 제공
  - 해당 컨텍스트를 구독한 컴포넌트에서만 데이터를 읽을 수 있음
  - UI 테마 정보나 로케일 데이터 같은 전역 정보 사용에 용이
  - 상위 컴포넌트의 props를 하위 컴포넌트에 전달하기 위해 상위 컴포넌트의 구현부에 Context Provider를 넣어줌
  - 하위 컴포넌트는 해당 컨텍스트를 구독, 데이터를 읽어오는 방식
  - 여러 컴포넌트 사이에서 값을 공유하는 솔루션에 가까움

  ## 10.2 상태 관리 라이브러리

  - MobX
    - 객체지향 + 반응형 프로그래밍 패러다임의 영향을 받은 라이브러리
  - Redux
    - FP의 영향을 받은 라이브러리
    - 독립적으로 상태 관리 라이브러리를 사용할 수 있음
    - 상태 변경 추적에 최적화되어 있음
    - 단순한 설정에도 보일러플레이트 필요
  - Recoil
    - 상태를 저장할 수 있는 Atom, 상태를 변형할 수 있는 순수 함수 selector를 통해 상태 관리
    - Redux에 비해 보일러플레이트가 적고 난이도 쉬움
    - 아직 검증x
  - Zustand
    - Flux 패턴을 사용. 훅 기반의 API 모듈 제공
    - 클로저를 활용하여 스토어 내부 상태를 관리
    - 상태와 상태를 변경하는 액션을 정의
