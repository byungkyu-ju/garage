# ETC

## 추가로 배운것

### - [1. 빈약한 도메인 모델(anemic domain model)](https://martinfowler.com/bliki/AnemicDomainModel.html)

    ```code
    public class Car {
        private String name;
        private int position;
    }
    ```

- 위와 같이 Car Class가 있다. 객체에게 할 일을 위임한다는 의미는 Car의 현재위치를 확인할 때 확인하는 기능을 외부에서 별도로 구현하는 것이 아닌  
  **Car 객체 내부에서 확인한다** 라는 의미이다.  
  Car 밖에서 car.getPosition() == maxPosition를 수행하는 것이 아닌,  
  Car에 isInPosition()과 같은 메서드를 만들어, car.isInPosition(maxPosition)과 같이 객체에 메시지를 던질 수 있다.

  위와 같이 속성정보만 가지고 있는 객체는 **빈약한 도메인 모델**이라고 부를 수 있으며, 위 링크를 통해 추가로 내용을 확인할 수 있다.

