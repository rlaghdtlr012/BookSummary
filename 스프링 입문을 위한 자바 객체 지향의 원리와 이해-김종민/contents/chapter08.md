## 8. 자바 8 함다와 인터페이스 스펙 변화

- 람다의 도입 이유

  - 자바 8에서는 함수형 프로그래밍 지원을 위한 람다(Lambda)를 도입했다.

  - 빅데이터를 프로그램적으로 다룰 수 있는 방법 -> 멀티 코어를 활용한 분산처리. 즉, 병렬화 기술이 필요.

  - 하나의 CPU에 다수의 코어를 삽입하는 멀티 코어 프로세서의 등장으로 병렬화 프로그래밍 가능

  - 자바 8에서는 병렬화를 위해 컬랙션(배열, List, Set, Map)을 강화했고, 이러한 컬렉션을 더 효율적으로 사용하기 위해 Stream을 강화했다.

  - 또, 스트림을 효율적으로 활용하기 위해 함수형 프로그래밍이, 함수형 프로그래밍을 위해 람다, 람다를 위해 인터페이스의 변화가 수반됨

  - 빅데이터 지원 -> 병렬화 강화 -> 컬렉션 강화 -> 스트림 강화 -> 람다 도입 -> 인터페이스 명세 변경 -> FP 도입

- 람다란 무엇인가?

  - 람다란 "코드 블록"을 지칭

  - 기존에 코드 반드시 블록은 메서드 내에 존재해야 함. 그래서 코드 블록만 갖고 싶어도 기존에는 코드 블록을 위한 메서드, 메서드를 위한 객체를 만들어야 했다.

  - 하지만 자바 8부터는 코드 블록인 람다를 메서드의 인자나 반환값으로 사용할 수 있게 됐다.

  - 즉, 코드 블록을 변수처럼 사용할 수 있다는 것

```java
// 기존 코드
// (run이라는) 코드 블록을 사용하기 위해서 별도의 클래스와 객체 그리고 메서드를 생성해야 하는 방식
public class B0001 {
    public static void main(String[] args) {
        MyTest mt = new MyTest();

        Runnable r = mt;
        r.run();
    }
}

class MyTest implements Runnable {
    public void run() {
        System.out.println("Hello World");
    }
}
```

```java
// 람다를 도입하여 새로운 방식의 코드 블록을 사용
// 코드 블록을 사용하는 데 있어서 더 이상 익명 객체를 생성할 필요가 없다.
public class B0002 {
    public static void main(String[] args) {
        Runnable r = () -> {
            System.out.println("Hello World");
        }
        r.run();
    }
}
```

    - Runnable 타입으로 참조 변수 r을 만들어서 new Runnable()은 컴파일러가 알아낼 수 있다. 그래서 굳이 코드로 작성할 필요가 없다.

    - public void run 메서드가 단순하게 ()로 변경될 수 있었던 이유는 Runnable 인터페이스가 가진 추상 메서드가 run 메서드 하나이기 때문에

<br>

- 함수형 인터페이스

  - 추상 메서드를 하나만 갖고 있는 인터페이스를 지칭
  - 함수형 인터페이스만을 람다식으로 변경 가능
  - 예시 코드

    ```java
    public class B005 {
        public static void main(String[] args) {
            // 추상 메서드 구현
            MyFunctionalInterface mfi = (int a) -> {
                return a * a;
            };

            // 간략화, 최적화 된 코드
            // MyFunctionalInterface mfi = a -> a * a;
            int b = mfi.runSometing(5);
            System.out.println(b);
        }

        // 애노테이션과 추상 메서드 정의
        @FunctionalInterface
        interface MyfunctionalInterface {
            public abstract int runSometing(int count);
        }
    }
    ```

  - @FunctionalInterface 어노테이션 -> 컴파일러는 인터페이스가 함수형 인터페이스의 조건에 맞는지 검사. -> 즉, 추상 메서드를 하나만 가지고 있는지 검사한다.
    <br>

- 메서드 호출 인자로 람다 사용

  ```java
  public class B006 {
      public static void main(String[] args) {
          // 메서드 호출 인자로 람다 사용
          MyFunctionalInterface mfi = a -> a * a;
          doIt(mfi);

          // 람다식을 한번만 사용한다면 굳이 변수에 할당도 필요없음
          doIt(a -> a * a);

          // 메서드 반환값으로 람다 사용. 한 번만 사용하기에 굳이 runSomething에 대한 정의 필요 없음
          MyFunctionalInterface2 mfi2 = todo();
          int result = mfi2.runSomething(3);
          System.out.println(result);
      }

      public static void doIt(MyFunctionalInterface mfi) {
          int b = mfi.runSomething(5);
          System.out.println(b);
      }

      public static MyFunctionalInterface2 todo() {
        return num -> num * num;
      }
  }
  ```

  <br>

- 자바 8 API에서 제공하는 대표적인 함수형 인터페이스

  | 함수형 인터페이스 | 추상 메서드       | 용도                                              |
  | ----------------- | ----------------- | ------------------------------------------------- |
  | Runnable          | void run()        | *실행*할 수 있는 ㅣ인터페이스                     |
  | Supplier<T>       | T get()           | *제공*할 수 있는 인터페이스                       |
  | Consumer<T>       | void accept(T t)  | *소비*할 수 있는 인터페이스                       |
  | Function<T, R>    | R apply(T t)      | *입력을 받아서 출력*할 수 있는 인터페이스         |
  | Predicate<T>      | Boolean test(T t) | 입력을 받아 *참/거짓을 단정*할 수 있는 인터페이스 |
  | UnaryOperator<T>  | T apply(T t)      | *단항 연산*을 할 수 있는 인터페이스               |

  <br>

- 컬렉션 스트림에서 람다 사용

      - 컬렉션 스트림을 통해 더 적은 코드로 더 안정적인 코드 만들기 가능
      ```java
      public class B012 {
          public static void main(String[] args) {
              Integer[] ages = { 20, 25, 18, 27, 30, 21, 17, 19, 34, 28 };
              // 향상된 for문을 사용한 20세 미만 출입금지 코드
              for (int age : ages) {
                  if (age < 20) {
                      System.out.println("Age %d!!! Can't enter\n", age);
                  }
              }

              // ***스트림을 이용한 반복문!!***
              // filter로 age를 걸러준 후,
              // 스트림 내부 반복을 실행하는 forEach 메서드 사용(이미 filter된 새로운 Array에 반복문을 돌림)
              Arrays.stream(ages)
                  .filter(age -> age < 20)
                  .forEach(age -> System.out.println("Age %d!!! Can't enter\n", age));
          }
      }
      ```
      - 함수형 프로그래밍의 장점: 선언적 프로그래밍 가능. 기존의 "어떻게 하라" 식의 프로그래밍 방식이 아닌 "무엇을 원한다" 식의 선언적 프로그래밍.

  <br>

- 메서드 레퍼런스와 생성자 레퍼런스
  ```java
  // 아래 두 코드는 같은 의미
  Arrays.stream(ages).sorted().forEach(System.out::println);
  Arrays.stream(ages).sorted().forEach(age -> System.out.println(age));
  ```
- 인자를 아무런 가공 없이 그대로 출력 -> 이런 코드를 메서드 레퍼런스라고 함
  - 인스턴스::인스턴스메서드
  - 클래스::정적메서드
  - 클래스::인스턴스메서드
- 위 세 가지 케이스로 사용 가능
