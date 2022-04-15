## Service Mesh(Sidecar 패턴)란

Sidecar란 이름 그대로 **독립적으로 옆에 붙어 실행되는 Application**이다.

기존의 Application에서 필요한 추가 기능을 별도의 Application으로 구축하고, 이를 동일한 Process나 Container에 배치하는 것이다.

Sidecar에서 생긴 장애, 수정 등이 Application이 영향을 받지 않게 된다.

기존 Application과 같은 리소스에 접근할 수 있고, 보통 같은 Container이기 때문에 지연 시간이 매우 짧다.

예를 들어 NGINX를 Sidecar로 붙일 수도 있고, 로깅을 위해 log stash를 같은 Container에 넣을 수 있다.

<br>

원래 MSA에선 각각의 서비스에서 Communication에 대한 로직을 구현해야했기 때문에 비지니스 로직에 집중하기 힘들었다.

거대한 MSA에선 수백, 수천개의 Service가 존재하고 그 사이의 통신도 매우 복잡하다.

다음과 같은 장점으로 인해 Service Mesh를 주로 사용한다.

### 장점

- 서비스간의 연동 대신 비지니스 로직에 집중할 수 있다.
- 문제를 파악하고, 분석하기 쉽다.
- 서비스에 장애가 발생했을 때 요청을 다시 Routing할 수 있기 때문에 Down Time이 발생했을 때 Application 복구 능력이 향상된다.
- 성능에 대한 Metric을 제공한다.

### 주요 기능

- Routing 제어
- Circuit Breaker
- Load Balancing
- 보안 기능
- Metric 기능 제공

### 구조

지금까진 외부 API를 그냥 호출했다면, 이젠 Service Mesh가 제공하는 Proxy끼리 통신하게 된다.

Service의 Traffic을 Network단에서 처리할 수 있고, Proxy에서 Routing도 가능하다.

![img](https://daddyprogrammer.org/wp-content/uploads/2020/09/mesh-3.png)

다양한 기능을 제공하기 위해서는 TCP기반 Proxy로는 부족하기 때문에 L7기반의 경량화된 Proxy를 사용한다.

<br>

Service Mesh는 다음과 같이 Control Plane과 Data Plane으로 구성된다.

![img](https://daddyprogrammer.org/wp-content/uploads/2020/09/service-mesh.png)

### Control Plane

Traffic을 제어하는 Policy나 Configuration에 따라 설정 값을 전달 / 관리하는 Controller역할을 한다.

### Data Plane

Proxy를 통해 오고가는 네트워크 Communication을 조정 및 제어한다.

Service Discovery, Load Balancing, Circuit Breaker 등의 기능을 제공한다.

Envoy Proxy라는 이름의 Data Plane이 가장 많이 사용된다.

### Envoy Proxy

Data Plane에서 사용되는 C++로 개발된 고성능 Proxy로, 다음 기능들을 수행한다.

- Dynamic Service Discovery
- Load Balancing
- TLS Termination
- HTTP/2와 gRPC 등 다양한 Proxy
- Circuit Breaker
- Health Check
- Fault Injection
- 다양한 Metrics
- %기반 점진적 Update

### VS API Gatway

API Gateway와 하는 일이 꽤 비슷한데, 다음과 같은 차이점이 있다.

- API Gateway는 외부의 요청을 받는 역할을 하지만, Service Mesh는 내부에서 요청을 받는다.
- 분산형 아키텍쳐이기 때문에 SPOF가 될 염려가 없다.

현재 API Gateway의 기능들을 하나하나 지원해나가고 있다.

## Istio

Istio는 오픈소스 Service Mesh로, **Application 네트워킹을 자동화 할 수 있고, 유연해질 수 있도록 돕는 Service Mesh**이다.

서비스간의 Load Balancing이나, Service Discovery 등 다양한 기능을 지원하는 Control Plane 덕분에 다음 기능들을 간단하게 사용할 수 있다.

- Service간의 Communication을 TLS로 암호화 및 Identity 기반 인증 인가
- HTTP, gRPC, Webflux, TCP를 자동으로 Load Balancing
- 자유롭고 다양한 Routing Rule, Retries, Failovers, Fault Injections
- Access 제어, 속도 제어 등을 지원하는 Policy와 설정 API
- Cluster 내부에서 자동 Metrics, Logging, Trace

Istio는 확장성을 위해 설계되었고, Istio의 Control Plane은 Kubernetes에서 실행된다.

해당 Cluster에 배포된 Application을 Service Mesh에 추가하고, Service Mesh를 다른 Cluster로 확장하거나, 심지어 Kubernetes 외부의 VM과 연결할수도 있다.