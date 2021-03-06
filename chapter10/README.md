# 스키마 레지스트리

스키마란 정보를 구성하고 해석하는 것을 도와주는 프레임워크나 개념을 의미한다.
스키마는 정보를 손쉽게 이해하고 해석하는데 쓰이며 특히 데이터베이스의 구조를 정의하고 표현 방법이나 전반적인 명세나 제약 조건을 기술하는 표준 언어로 활용되고 있다.

## 스키마의 개념과 유용성

관계형 데이터베이스의 경우 스키마가 미리 정의되어 있고, 관계형 데이터베이스에 데이터를 추가하기 위해서는 반드시 사전에 정의된 스키마의 형태로 데이터를 입력해야 한다.
스키마를 검증하면 틀정 필드에 값을 잘못 넣을수가 없다. 토픽에 데이터가 잘못 들어가면 모든 시스템의 영향을 받을수 있다.

데이터 파이프라인 역활을 하는 카프카에선 하나의 토픽에 하나의 어플리케이션이 접근하는것이 아니라 여러 어플리케이션이 접근을 한다. 토픽에 값이 잘못들어가면 연결된 모든 시스템이 영향을 받고 이는 심각한 문제로 이어진다.

카프카의 데이터 흐름은 대부분 브로드캐스트 방식이다. 다시 말하면 카프카는 데이터를 전송하는 프로듀서를 일방적으로 신회할 수 밖에 없는 방식이다.
컨슘하는 관리자에게 반드시 데이터 구조를 설명해야 된다. 하지만 대상이 많으면 구조를 설명하기 힘들다. 이런 상황을 해결할때도 스키마를 활용할수 있다.
그리고 스키마 변경상황을 전파할떄도 유용하게 쓰일수 있다.
파싱처리도 중복해서 하지 않아도 된다. 

단점은 스키마를 정의하는것이 번거럽고 불편한 일이다.

## 카프카와 스키마 레지스트리

데이터 처리시 유연성 확보와 커뮤니케이션의 불편함을 해결할수 있다.

### 스키마 레지스트리 개요

카프카에서 스키마를 활용하는 방법은 스키마 레지스트리라는 별도의 애플리케이션을 이용 하는 것이다.

카프카 0.8.2 버전 출시와 함께 세상에 처음 공개됨 아파치 라이센스가 아니 컨플루언트 커뮤니티 라이센스를 갖고 있는데 비상업적인 용도에 한해 스키마 레지스트리를 무료로 사용할수 있다.

스키마 레지스트리를 이용하기 위해서는 스키마 레지스트리가 지원하는 데이터 포멧을 사용해야 하는데 가장 대표적인 포맷은 에이브로이다.

### 스키마 레지스트리에 에이브로 지원

AVRO는 시스템 프로그래밍 언어 프로세싱 프레임워크 사이에서 데이터 교환을 도와주는 오픈소스 직렬화 시스템이다.
하둡 프로젝트에서 처음 시작된 에이브로는 시스템 프로그래밍 언어 프로세싱 프레임워크 사이에서 데이터 교환을 도와주는 오픈소스 직렬화 시스템으로서 빠른 바이너리 데이터 포맷을 지원하며 
JSON 형태의 스키마를 정의할 수 있는 매우 간결한 데이터 포맷이다.

스키마 레지스트리는 AVRO를 제일 먼저 지원 했으면 최근에는 JSON, Protocol Buffer 포멧도 지원하고 있다.
따라서 관리자는 이 세가지 포멧중 한가지를 선택해 스키마 레지스트리를 사용할 수 있다.

## 스키마 레지스트리 호환성

스키마가 진화함에 따라 호환성 레벨 검사를 해야 되는데 BACKWARD, FORWARD, FULL등 의 호환성 레벨을 사용한다.

### BACKWARD 호환성

BACKWARD 호환성은 진화된 스키마를 적용한 컨슈머가 진화 전의 스키마가 적용된 프로듀서가 보낸 메시지를 읽을수 있도록 하는 호환성을 말한다.

BACKWARD는 자신과 동일한 버전과 하나 아래의 하위 버전을 수용한다는 뜻이다.

모든 하위 버전을 포함 할때는 BACKWARD_TRANSITIVE 호환성을 지정해야 된다.

### FORWARD 호환성

진화된 스키마가 적용된 프로규서가 보낸 메시지를 진화 전의 스키마가 적용된 컨슈머가 읽을 수 있게 하는 호환성을 말한다.

FORWARD 는 자신과 동일한 버전과 하나 위의 버전을 수용 한다는 뜻

모든 상위 버전을 수용하려면 FORWARD_TRANSITIVE 호환성을 지정해야 된다.

### FULL 호환성

BACKWARD 호환성과 FORWARD 호환성 모두를 지원한다. 

FULL 자신과 동일한 버전과 하나 아래/위의 버전을 수용한다는 뜻

모든 버전을 수용하려면 FULL_TRANSITIVE 호환성을 지정해야 된다.


# 참고

------

* [avro](https://avro.apache.org/)
* [protocol-buffers](https://developers.google.com/protocol-buffers/)
* [json-schema](https://json-schema.org/)