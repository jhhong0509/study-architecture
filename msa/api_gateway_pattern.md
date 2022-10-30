# API Gateway Pattern

## API Gateway Pattern이란?

Facade 패턴과 비슷하지만 분산 시스템의 Reverse Proxy나 Gateway Routing 등에서 사용되는 패턴이다.

Facade 패턴처럼 **API를 위한 하나의 Entry Point를 제공**하며, **시스템 아키텍쳐를 캡슐화**해준다.

### API Gateway Pattern의 역할

- Reverse Proxy
- 백엔드 서비스로 라우팅
- 인증, SSL, 로깅 등의 횡단 관심사 제공
- 요청 취합: 여러 마이크로서비스들에 걸쳐 필요한 정보를 받고, 그 정보들을 가공해서 반환한다.

![img](https://miro.medium.com/max/1400/1*gW4JrHTr86HnTrouQYLgJQ.png)

## API Gateway Pattern의 오해

API Gateway Pattern에 대해 오해하고 있는 부분이 있다.

API Gateway를 말할 때 단순히 서비스로 라우팅하는, 단순한 구조를 상상하기 쉽다.

하지만 API Gateway는 보통의 MVC에서의 Presentation 계층 역할을 대체하고, 각각의 마이크로서비스가 서비스 계층이 된다.

따라서 **응답의 가공과 변환, Facade(API 조합), 보안 처리 등의 역할은 모두 API Gateway의 역할**이고, 따라서 개발자가 직접 API Gateway들을 하나하나 구성하는것이 옳다.

출처: [우아콘](https://www.youtube.com/watch?v=P2nM0_YptOA)