## 객체, 설계

- 소프트웨어 개발에서만큼은 실무 > 이론

### 01. 티켓판매 애플리케이션 구현하기

- 상황

  - 입장객이 연극에 대한 초대권을 가지고 있다면, 초대권을 티켓과 교환
  - 입장객의 가방에는 현금 or 티켓 or 초대권이 있음
  - 티켓 판매원은 입장객에 줄 티켓과 돈을 가지고 있음
  - 입장객은 초대권 or 티켓 or 현금이 있는 상황

    ```java
    // 연극을 여는 추체인 극장 클래스에 대한 코드
    public class Theater {
      private TicketSeller ticketSeller;

      // 극장 클래스 생성자 생성
      // 극장에는 티켓 판매원이 있을 것이기에 ticketSeller 의존성 주입
      public Theater(TicketSeller ticketSeller) {
        this.ticketSeller = ticketSeller;
      }

      // 입장객을 입장시켜주는 메서드 구현
      public void enter(Audience audience) {
        // 초대권이 있다면, 초대권과 티켓을 교환하여 입장객의 가방에 넣어줌
        if (audience.getBag().hasInvitation()) {
          Ticket ticket = ticketSeller.getTicketOffice().getTicket();
          audience.getBag().setTicket(ticket);
        } else {
          // 초대권이 없다면, 입장객이 티켓값을 지불하는 구조
          Ticket ticket = ticketSeller.getTicketOffice().getTicket();
          audience.getBag().minusAmount(ticket.getFee());
          ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
          audience.getBag().setTicket(ticket);
        }
      }
    }
    ```

  - 위 코드는 겉보기에는 문제 없지만 실상은 몇 가지 문제를 가지고 있음

### 02. 무엇이 문제인가

- 로버트 마틴이 정의한 모듈이 가져야하는 세 가지 목적
  - 실행중에 제대로 동작하는 것
  - 변경을 위해 존재하는 것(변경에 용이해야 함)
  - 코드를 읽는 사람과 의사소통 하는 것
- 즉, 모든 모듈은 제대로 실행돼야 하고, 변경에 용이해야 하며, 이해하기 쉬워야 한다.
- 01의 예시는 제대로 동작하지만, 변경에 취약하고 코드를 읽는 사람과 의사소통이 잘 되지 않는다는 문제가 있음
