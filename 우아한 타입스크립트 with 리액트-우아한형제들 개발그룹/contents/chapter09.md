## 9.1 리액트 훅

- 훅의 탄생 배경

  - 기존 클래스 형 컴포넌트에서는 componentDidMount, componentDidUpdate에서 같이 하나의 생명주기 함수에서만 상태 업데이트를 할 수 있었음.
  - 프로젝트의 규모가 커질수록 관심사가 섞이게 되고, 상태관리가 힘들어지는 현상 발생
  - 리액트 훅이 도입되면서 컴포넌트의 생명주기에 맞춰, 로직을 분할하거나 재사용 가능해서 테스트에 용이해졌고, 사이드 이펙트와 상태를 관심사에 맞게 분리할 수 있게 됨

### useState

- useState의 타입 정의

  ```tsx
  function useState<S>(
    initialState: S | (() => S)
  ): [S, Dispatch<SetStateAction<S>>];

  type Dispatch<A> = (value: A) => void;
  type SetStateAction<S> = S | ((prevState: S) => S);
  ```

- useState가 반환하는 튜플의 첫번째 요소: 제네릭으로 지정한 S.[S,] <- 이거
- useState가 반환하는 튜플의 두 번째 요소: 상태를 업데이트 할 수 있는 Dispatch 타입의 함수. 이전 값을 받아서 새로운 상태를 반환하는 함수인 SetStateAction
- useState에 타입스크립트 적용. 컴파일 단계에서 개발자 레벨의 타입 에러를 미리 알 수 있다.

  ```tsx
  import { useState } from "react";

  interface Member {
    name: string;
    age: number;
  }

  const MemberList = () => {
    const [memberList, setMemberList] = useState<Member[]>([]);

    // member의 타입이 Member 타입으로 보정됨
    const sumAge = memberList.reduce((sum, member) => sum + member.age, 0);

    const addMember = () => {
      // 🚨 Error: Type 'Member' | { name: string; agee: number;}
      // is not assignable to type 'Member'. 타입 에러 발생
      setMemberList([
        ...memberList,
        {
          name: "DokgoBaedal",
          agee: 11,
        },
      ]);
      return (
        //
      );
    };
  };
  ```

### 의존성 배열을 사용하는 훅

- useEffect

  - 렌더링 이후, 컴포넌트에 어떤 일을 수행해야 하는지 알려주기 위한 훅

    ```typescript
    // useEffect 타입 정의
    function useEffect(effect: EffectCallback, deps?: DependencyList): void;

    type DependencyList = ReadonltArray<any>;
    type EffectCallback = () => void | Destructor;
    ```

  - useEffect의 첫번째 인자 effect는 아무것도 반환하지 않거나 Destructor를 반환
  - **Promise 타입을 반환하지 않으므로 useEffect의 콜백함수에는 비동기 함수가 들어갈 수 없다. -> 왠지 useEffect 안에서 비동기함수(async, await)를 사용하면 에러 띄워주더라.**
  - 그래서 차선책으로 useEffect 안에서 비동기 함수를 실행시키려면, useEffect 안에 비동기 로직을 사용하는 함수를 새로 정의하고, 함수를 호출(함수이름();) 하는 수 밖에 없었다.(내 경험담)
  - useEffect에서 비동기 함수 호출시, 경쟁상태를 불러일으킬 수 있음
  - c.f) 경쟁상태(Race Condition): 여러 프로세스나 스레드가 동시에 공유된 자원에 접근하려고 할 때 발생하는 문제. 실행 순서나 타이밍을 예측할 수 없어서 프로그램 동작이 원하는대로 되지 않을 수 있음
  - 두 번째 인자: deps => effect가 수행되기 위한 조건. 보통 기본 자료형을 넣고, 객체나 배열을 넣을 때는 주으이해야 한다.
  - useEffect는 deps가 변경되엇는지를 얕은 비교로만 판단하기 때문에 실제 객체 값이 바귀지 않고, 객체의 참조 값만 변경되었어도, 콜백 함수를 실행함.
  - 이를 방지하기 위해 "실제 사용하는 값"을 의존성에 넣어야 한다.

    ```tsx
    type SomeObject = {
      name: string;
      id: string;
    };

    interface LabelProps {
      value: SomeObject;
    }

    /**
     * useEffect는 의존성을 얕은 비교를 통해서 판단하기 때문에 value 같은 객체 자체를 의존성에 넣어주는 것은 주의가 필요
     * BAD!!!
     */
    const Label = ({ value: LabelProps }) => {
      useEffect(() => {
        // value.name, value.id ~~
      }, [value]);
    };

    /**
     * 이렇게 직접 사용되는 값을 이용하여 의존성을 관리하는 것이 더 좋다.
     * BETTER!!!
     */
    const Label = ({ value: LabelProps }) => {
      const { id, name } = value;
      useEffect(() => {
        //id, name~~~
      }, [id, name]);
    };
    ```

  - **useEffect는 레이아웃 배치화 화면 렌더링이 모두 완료된 후에 실행됨**

- useLayoutEffect

  - 컴포넌트가 다 렌더링이 된 후에 실행되는 useEffect와는 달리 화면에 해당 컴포넌트가 그려지기 전에 콜백함수를 실행.

    ```tsx
    // useEffect는 렌더링이 다 된 후에,
    const [name, setName] = useState("");

    // 1번 useEffect
    useEffect(() => {
      // 렌더링이 다 된 후에, setName 실행된다. 따라서 처음에 "안녕하세요   님"으로 뜨는 현상이 발생함
      setName("배달이");
    }, []);

    // 2번 useLayoutEffect
    useLayoutEffect(() => {
      // useLayoutEffect는 컴포넌트가 그려지기 전에 setName을 실행하기 때문에 "안녕하세요    님"과 같이 렌더링 초기에 빈 값이 띄워지는 현상을 방지할 수 있음
      setName("배달이");
    }, [])

    return (
      <div>
        {`안녕하세요 ${name}`님}
      </div>
    );
    ```

### useMemo, useCallback

- useMemo와 useCallback은 모두 이전에 생성된 값 또는 함수를 기억하며, 동일한 값과 함수를 반복해서 생성하지 않도록 해주는 훅.
- 어떤 값을 계산하는데 오랜 시간이 걸릴 때나 렌더링이 자주 발생하는 form에서 유용하게 사용 가능.
- useEffect와 마찬가지로 의존성을 비교하는 데에는 얕은 비교를 통해 비교. 따라서 객체나 배열을 쓸 때 주의가 필요. 혹은 그냥 "값" 그 자체로만 의존성에 넣어주는 것이 낫다
- **모든 값과 함수를 useMemo와 useCallback을 사용하여 과도하게 메모이제이션하면 컴포넌트의 성능 향상 보장 안 됨**

### useRef

- DOM을 직접 선택해야 하는 경우

  ```tsx
  import { useRef } from "react";

  const MyComponent = () => {
    const ref = useRef<HTMLInputElement>(null);

    const onClick = () => {
      ref.current?.focus();
    }

    return (
      <>
        <button onClick={onClick}>ref에 포커스!</ref>
        ㄴ<input ref={ref} />
      </>
    )
  }

  export default MyComponent;
  ```

- useRef는 세 종류의 타입 정의를 가지고 있음.
- useRef에 넣어주는 인자 타입에 따라 반환하는 타입이 달라짐

  ```tsx
  function useRef<T>(initialValue: T): MutableRefObject<T>;
  function useRef<T>(initialValue: T | null): RefObject<T>;
  function useRef<T = undefined>(): MutableRefObject<T | undefined>;

  // useRef<HTMLInputElement | null>()을 사용하면 current 값이 바뀌는 sideEffect가 나올 수 있음
  interface MutableRefObject<T> {
    current: T;
  }

  // useRef<HTMLInputElement>(null)을 사용하면 current 값이 readonly가 되어 값을 바꿀 수 없음
  interface RefObject<T> {
    readonly current: T | null;
  }
  ```

- ref는 리액트에서 DOM 요소 접근이라는 목적으로 사용되기에 props로 넘겨주는 방식으로 전달하면 안됨. 리액트에서 ref를 prop으로 전달하기 위해서는 -> **forwardRef 사용**

  ```tsx
  interface Props {
    name: string;
  }

  // forwardRef의 두 번째 인자에 ref를 넣어서 자식 컴포넌트로 ref를 전달
  const MyInput = forwardRef<HTMLInputElement, Props>((props, ref) => {
    return (
      <>
        <label>{props.name}</label>
        <input ref={ref} />
      </>
    );
  });
  ```

- useRef의 여러 가지 특성

  - useRef는 자식 컴포넌트를 저장하는 변수 뿐만 아니라 다른 방식으로도 사용 가을
  - useRef는 변수 값이 바뀌어도 컴포넌트의 리렌더링 발생 x
  - 이를 통해 상태가 변경되어도 불필요한 리렌더링 피할 수 있다
  - 다른 상태들은 상태 변경 함수 호출 후, 값을 확인할 수 있지만 useRef는 값을 설정한 후, 즉시 값을 확인할 수 있음

    ```tsx
    // 예시 코드
    // state로 된 값을 확인하기 위해서는 아래와 같이 로직을 구성하여 값 확인을 해야함
    const [ex, setEx] = useState(null);

    useEffect(() => {
      console.log(ex);
    }, [ex]);

    // 하지만 useRef의 경우 아래와 같이 바로 값 확인 가능
    const ex = useRef(null);

    console.log(ex.current);
    ```

- 훅의 규칙

  1. 훅은 항상 최상위 레벨에서 호출되어야 한다.
  2. 훅은 항상 함수 컴포넌트나 커스텀 훅 등의 리액트 컴포넌트 내에서만 호출되어야 한다.
     <br><br>

## 9.2 커스텀 훅

### 나만의 훅 만들기

- 리액트에서 제공하는 useState, useEffect, useRef 등의 훅에 더해 사용자 훅을 생성하여 컴포넌트 로직을 함수로 뽑아내 재사용 가능하게 한 기능
- 작명은 반드시 use로 시작해야함

  ```jsx
  // useInput 컴포넌트
  import { useState } from 'react';

  const useInput = (initialValue) => {
    const [value, setValue] = useState(initialValue);

    const onChange = (e) => {
      setValue(e.target.value);
    }

    return { value, onChange };
  }

  // useInput 컴포넌트 사용
  const MyComponent = () => {
    const { value, onChange } = useInput("");

    return (
      <div>
        <h1><{value}/h1>
        <input onChange={onChange} value={text} />
      </div>
    )
  }
  ```
