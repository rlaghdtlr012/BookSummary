## 4장. 타입 설계

- 타입 시스템의 가장 큰 장점: 데이터 타입을 명확히 알 수 있어 코드를 이해하기 쉬움

### 아이템 28: 유효한 상태만 표현하는 타입을 지향하기

- 효과적으로 타입을 설계하려면, 유효한 상태만 표현할 수 있는 타입을 만들어 내는 것이 가장 중요

- 잘못된 설계
  ```typescript
  interface State {
    pageText: string;
    isLoading: boolean;
    error?: string;
  }
  async function changePage(state: State, newPage: string) {
    state.isLoading = true;
    try {
      const response = await fetch(getUrlForPage(newPage));
      if (!response.ok) {
        throw new Error(`unable to load ${newPage}: ${response.statusText}`);
      }
      const text = await response.text();
      state.isLoading = false;
      state.pageText = text;
    } catch (e) {
      state.error = "" + e;
    }
  }
  ```
- 위 코드의 문제점

  - state.error를 초기화 하지 않아서 페이지 전환 중에 로딩 메시지가 과거의 메시지를 보여줌
  - 페이지 로딩 중에 사용자가 changePage를 할 경우, 동작을 예측하기 힘듦.(새 페이지에서 오류가 뜨거나 첫번째로 응답이 오는 페이지 정보를 띄울 수도 있음)
  - State 타입은 isLoading이 true이면서 동시에 error 값이 설정되는 상태를 허용함
  - 무효한 상태가 존재하면 코드가 제대로 동작하지 않을수도 있음

- 개선된 타입 설계

  ```typescript
  interface RequestPending {
    state: "pending";
  }
  interface RequestError {
    state: "error";
    error: string;
  }
  interface RequestSuccess {
    state: "ok";
    pageText: string;
  }
  type RequestState = RequestPending | RequestError | RequestSuccess;

  interface State {
    currentPage: string;
    requests: { [page: string]: RequestState };
  }
  ```

- 위 개선된 코드는 무효한 상태를 허용하지 않도록 개선되고, 모든 요청이 정확히 하나의 상태로 맞아 떨어짐
- 유효한 타입, 무효한 타입을 모두 허용하는 상태의 타입은 오류를 발생하게 된다.
- 코드 작성 시간이 길어질지라도, 유효한 상태만 표현하는 타입을 지향해야 한다.
