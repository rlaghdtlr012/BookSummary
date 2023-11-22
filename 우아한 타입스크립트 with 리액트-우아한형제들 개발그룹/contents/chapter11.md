## 11.1 CSS-in-JS

### Css-in-JS와 인라인 스타일 차이

- CSS-in-JS를 사용하면 CSS-in-CSS보다 더 선언적이고 유지보수하기 쉬운 방식으로 표현 간으
- 인라인 스타일: HTML 요소 내부에 직접 스타일을 적용하는 방식. style={} 방식
- 인라인 스타일

  - ex: sass/scss, less, stylus

  ```javascript
  const textStyles = {
    color: white,
    backgroundColor: black,
  };

  const someComponent = () => {
    return <p style={textStyles}>inline style!</p>;
  };

  // 브라우저 상의 DOM에서 아래와 같음
  <p style="color: white; background-color: black;">inline style!</p>;
  ```

- CSS-in-JS

  - ex: styled-componets, emotion

  ```javascript
  import styled from "styled-components";
  const Text = styled.div`
      color: white,
      background: black
  `

  const Example = () <Text>{Hello CSS-in-JS}</Text>

  // 브라우저 DOM에서 아래와 같음
  <style>
  .hash136s21 {
    background-color: black;
    color: white;
  }
  </style>
  <p class="hash136s21">Hello CSS-in-JS</p>
  ```

- CSS-in-JS는 DOM 상단에 style 태그 추가, 실제로 css가 생성되기 때문에 미디어 쿼리 등의 css 기능을 사용할 수 있음

#### CSS-in-JS의 장점

- 컴포넌트로 생각할 수 있음: 스타일을 컴포넌트 단위로 추상화하여 생각할 수 있음. 별도의 스타일 시트를 유지보수할 필요 없이 각 컴포넌트의 스타일로 관리 가능
- **부모와 분리할 수 있다: 일반적인 CSS와 다르게 부모의 CSS 속성을 상속받지 않는다. 따라서 컴포넌트의 스타일은 부모와 독립되어 동작한다**
- 스코프를 가진다: CSS는 하나의 전역 네임스페이스를 가지기 때문에 선택자 충돌을 피하기 힘듦. CSS는 서드파티 코드를 통합할 때, 네이밍상 도움이 안되지만, CSS-in-JS는 CSS로 컴파일 될 때 고유한 이름을 생성하여 선택자 충돌을 방지해줌
- 자동으로 벤더 프리픽스(웹 브라우저마다 지원되는 CSS 속성이나 기능이 다를 때, 특정 브라우저에서 제대로 동작하도록 하기 위해 추가되는 접두사)가 붙어서 브라우저 호환성을 높여줌
- JS와 CSS 사이에서 상수와 함수를 쉽게 공유할 수 있게 해줌

### CSS-in-JS의 등장배경

- 규모가 크고 동적인 웹 어플리케이션에서 파생되는 문제점인 css 선택자의 복잡도 증가,유지보수하기 어려움 등의 문제를 해결하기 위해
- css의 7가지 문제점
  - Global Namespaces: 모든 스타일이 전역 공간을 공유하므로 중복되지 않는 css 클래스 이름을 고민해야함
  - Dependencies: css의 의존성과 js의 의존성이 달라서 사용하지 않는 스타일이 포함되거나 꼭 필요한 스타일이 누락되는 문제 팔생
  - Dead Code Elimination: 기능 추가, 삭제 과정에서 불필요한 css를 삭제하기 어려움
  - Minification: 클래스 이름을 최소화하기 어려움
  - Sharing Constants: 자바스크립트와 상태 값을 공유할 수 없다(현재는 css variable이라는 기능이 생김)
  - Non-deterministic Resolution: CSS 로드 순서에 따라 스타일 우선순위가 달라짐
  - Isolation: CSS의 외부 수정을 관리하기 어려움
- CSS-in-JS의 등장으로 스타일을 포함한 컴포넌트 단위로 코드를 쪼갤 수 있게됨

### CSS-in-JS 사용하기

```javascript
import styled from "styled-components";

const Button =
  styled.button <
  { primary: boolean } >
  `
        color: white,
        background: black,
        font-size: inherit,
        color: ${({ primary }) => (primary ? "red" : "blue")}
    `;
```

- 템플릿 리터럴을 활용한 동적 스타일 정의 가능
