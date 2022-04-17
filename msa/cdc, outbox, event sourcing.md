## CDC란

CDC에서 여러 Data Store에 걸친 데이터가 있을 때, 이 데이터는 두 가지로 나눌 수 있다.

- System of Records
- Derived Data

우리가 하나의 데이터를 각자 서비스에 맞게 변환하여 저장해뒀다고 가정해보자.

이 때 우리는 더 신뢰성 있는 버전을 따라야 하는데, 이게 System of Records이다.

<br>

System of Records란 처음에 데이터가 서비스에서 생겼을 때의 이 데이터를 의미한다.

이 데이터는 다른 서비스에서 복제하고 변형해서 가져갈 수 있는데, 이 때 복제되고 가공된 데이터를 Derived Data라고 한다.

만약 Derived Data가 이상하다거나 소실되었을 때에는 System of Records에서 가져오면 된다.

<br>

Application의 상태를 일관되게 유지하기 위해 System of Records와 Derived Data는 항상 동기화 되어야 하며, System of Records의 변화는 Search Index나 Cache 등에도 적용되어야 한다.

따라서 MSA에선 Data Store에서 안전하게 가져올 방법이 필요한데, 이 때 CDC를 사용하는 것이다.

**CDC는 모든 데이터 변경을 관측하고, 이걸 다른 Data Store 등에 복제하는 방법**이다

CDC에서는 Transaction Log를 계속 관찰하며 변경이 생겼을 때 DownStream Service에 전파한다.

## Event Sourcing이란

DB에서 다음과 같은 Document를 가지고 있다고 가정해보자.

``` 
{
    id: 1,
    amount: 50,
    status: "issued"
}
```

id는 1이고, 50개를 주문한 것이다.

이제 30일이 지나고, Batch Job에 의해 지불이 되지 않은걸 보고 status를 overdue로 변경했다고 해보자.

``` 
{
    id: 1,
    amount: 50,
    status: "overdue"
}
```

또 그 뒤에 정상적으로 값을 지불하면 status를 paid 상태로 바꿔야 한다.

``` 
{
    id: 1,
    amount: 50,
    status: "paid"
}
```

자 여기서 뭐가 문제일까?

이렇게 하면 Entity의 변경 로그가 저장되지 않는다.

크게 중요해보이지 않을지 몰라도 연체되었다는 정보는 유용할 수 있다.

예를 들어 일반적으로 하나의 Document에서 그냥 status를 바꿨을 때, `연체된 사람들은 거래를 30일동안 못하게 해주세요`라는 요구사항이 들어오면 어떻게 처리할 것인가?

할수야 있겠지만 매우 귀찮다.

<br>

그렇다면 Event Sourcing을 사용했을 때를 보자.

IssuedEvent부터 시작되는 변경 사항들을 저장하고 있기 때문에 언제, 어떤 상태였는지 알 수 있다.

IssuedEvent를 저장하고, 미납 30일 후에는 OverdueEvent도 저장한다.

마지막으로 10일 후 결제를 하면 PaidEvent를 발행하고 status를 paid 상태로 변경한다.

그럼 다음과 같은 Event들이 **모두** 저장된다.

``` 
IssuedEvent {id:1, amount: 50, status:'issued'}

OverdueEvent {id:1, amount: 50, status:'overdue'}

PaidEvent {id:1, amount: 50, status:'paid'}

invoice {id:1, amount: 50, status:'paid'}
```

이 방법을 사용하면 Entity에 대한 모든 기록을 가지고 있다.

따라서 분석을 할때도, 특정 순간의 snapshot을 복원할때도 유용하다.

주로 분산 로그, CQRS, DDD와 함께 사용된다.

## Event Sourcing과 CDC의 목적

Event Sourcing과 CDC의 목적은 비슷한 점이 있다.

- 하나의 Data Store를 다른 Data Store로 전파한다.
- Event, Transaction Log 등으로 Application의 과거, 현재를 나타낸다.
- State를 다시 복구하거나, 재전송 하는 등의 기능을 제공한다.

Event Sourcing에서는 자신의 Data Store에 모든 변경 기록을 저장하고 가져오지만, CDC는 DB Transaction log에 의존한다.

## Outbox란

우선 기존에 자신의 Local DB를 수정하고, 이벤트를 발행할 때 Sequence Diagram은 다음과 같다.

![img](http://www.kamilgrzybek.com/wp-content/uploads/2019/03/Without-outbox.png)

EventBus가 유효하지 않다면 Event를 발행하지 못할 수도 있다.

하지만 이벤트 발행 로직은 Transaction에 포함되지 않기 때문에 메세지 발행은 실패하겠지만, Local DB에 처리되지 않은, 처리되지 않을 데이터가 남게된다.

이를 해결하기 위한 디자인 패턴이 바로 Outbox 패턴이다.

<br>

Outbox는 Guaranteed Delivery라는 패턴을 기반으로 하고, 다음과 같은 Sequence Diagram을 가진다.

![img](http://www.kamilgrzybek.com/wp-content/uploads/2019/03/Outbox.png)

만약 하나의 트랜잭션 안에서 데이터를 저장했다면 그 트랜잭션 안에서 메세지도 Outbox에 저장하는 것이다.

<br>

다음으로 해결해야할 문제는 Outbox테이블에서 주기적으로 저장된 데이터를 읽고, 처리하는 것이다.

이미 처리된 작업은 처리되었다고 표시해서 같은걸 다시 처리하는걸 막아야 한다.

하지만 Outbox테이블에 완료되었다는 메세지를 전송하지 못할수도 있다.

![img](http://www.kamilgrzybek.com/wp-content/uploads/2019/03/Outbox-message-processing.png)

이 상황에서 Outbox 테이블과의 연결이 복구된다면 같은 메세지가 또 발행될 것이다.

이건 어쩔 수 없다.

따라서 Outbox는 **최소 한 번 이상 발행을 보장**한다.

메세지가 일단 한 번 발행되는건 보장하지만, 여러번 발행될수도 있다는 소리다.
