## 숫자

- 우리가 직관적으로 알고 있거나 이해하고 있는 "수학에서의 수"는 "js의 수"와 완벽하게 일관되지는 않는다.

- 자바스크립트에서의 숫자형은 'number' 하나 뿐(js의 가장 큰 장점 중 하나).

- int를 사용하지 않기에, 오버플로우도 발생하지 않음

  - js: 2147483547 + 1 // 문제없음
  - java: 2147483547 + 1 // 오버플로우

- 숫자 리터럴

  - 정수에 대한 숫자 리터럴: 연속한 10진수 숫자들
  - e를 사용하여 10의 거듭제곱 표현 가능

- NaN: Not a number의 뜻이지만, typeof NaN == number이다.

  - NaN은 문자열을 숫자로 변환하려고 했으나 실패했을 때, 결과값으로 반환될 수 있음.
  - 변환에 실패할 경우, 프로그램이 멈추는 대신 NaN을 반환
  - 만약 테스트 기댓값이 NaN이라면 실제 값이 NaN이라도 항상 실패.
  - 따라서 값이 **NaN인지 테스트하려면 Number.isNaN(value)를 통해 확인해야함**

- Number(숫자를 만드는 함수)

  - new 예약어를 앞에 붙이지 말고 그냥 Number로 감싸서 사용해야 함
  - javascript의 Number.MAX_SAFE_INTEGER는 약 9천조이며, 여기에 1을 더하더라도 Number.MAX_SAFE_INTEGER보다 커지지 않는다.
  - 즉, javascript의 "수"는 -Number.MAX_SAFE_INTEGER와 Number.MAX_SAFE_INTEGER 사이의 값들의 연산이다. 이 사이를 벗어나면, 계산의 정확성이 보장되지 않음
  - 즉, javascript는 Number.MAX_SAFE_INTEGER 이상의 수를 보장해주지 않는다.

- Math 객체
  - Math.floor(): 더 작은 정수 // Math.floor(-2.5) // -3
  - Math.trunc(): 0에 더 가까운 정수 // Math.trunc(-2.5) // -2
