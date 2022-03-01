# 카프카 버전 업그레이드와 확장
## 카프카 버전 업그레이드를 위한 준비

내가 사용하고 있는 카프카 버전 확인 방법

```shell
kafka-topics.sh --verion 
```

메이저 버전 업그레이드는 메시지 포멧변경, 브로커에서의 기본값 변화, 과거에 지원되었던 명령어 종료, 일부 JMX 메트릭 지원 종료 등 의 문제가 있을수 있으므로 이슈가 없는지 꼼꼼히 확인해야 된다.

마이너 버전 업그레이드는 비교적 용이하게 업그레이드 할 수 있다.

업그레이드 방법은 다운 타임을 가질수 있는 업그레이드 방법과 다운타임을 가질수 없는 경우 두가지로 나눌수 있다.

다운 타임을 가질수 없다면 브로커 한대씩 롤링 업그레이드를 하는것이 좋다.

## 주키퍼 의존성이 있는 카프카 롤링 업데이트

2.1 버전에서 2.6 버전으로 업데이트 2.1 버전에서는 사용할수 없는 옵션들이 존재함 잘 확인해서 토픽을 생성하고 확인해야 된다.

server.properties
```properties
# 브로커 통신에 사용하는 프로토콜 버전(CURRENT_KAFKA_VERSION)
inter.broker.protocol.version=2.1
# 메시지 포멧 버전(CURRENT_KAFKA_VERSION)
log.message.format.version=2.1
```

브로커 통신에 사용하는 프로토콜 버전

위와 같은 설정을 해주고 브로커 한대씩 업그래이드 및 재시작을 한다.

업그레이드가 완료 되면 위에 설정된 옵션을 삭제하고 브로커 한대씩 재시작을 해준다.

## 카프카의 확장

초기에 클러스터의 규모를 산정해서 구축하지만 사용량이 증가하면 별의미가 없을수도 있다.
하지만 쉽게 확장할수 있게 디자인 되어 있어서 걱정할 필요가 없다.

신규로 브로커 추가 해도 토픽의 내용은 분산되지 않는다 수동으로 토픽의 내용을 분산 시켜야 된다.

또 부하 분산이 목적인 경우는 브로커만 추가했다고 끝나는 것이 아니라 새롭게 추가된 브로커에 기존에 파티션들을 할당해야 한다.

kafka-reassign-partitions.sh 를 사용하면 파티션을 이동 시킬수 있다.

reassign-partitions-topic.json
```json
{"topics":
     [{"topic": "peter-scaleout1"}],
     "version":1
}
```

위와 같은 json 파일을 생성 시켜서 명령어를 사용하여 분산시킬 브로커 리스트를 지정한다.

```shell
/usr/local/kafka/bin/kafka-reassign-partitions.sh --bootstrap-server peter-kafka01.foo.bar:9092 --generate --topics-to-move-json-file reassign-partitions-topic.json --broker-list "1,2,3,4"

```

위 명령어를 실행하면 제한된 파티션 배치를 출력 해준다.

move.json
```json
{
  "version": 1,
  "partitions": [
    {
      "topic": "peter-scaleout1",
      "partition": 0,
      "replicas": [
        2
      ],
      "log_dirs": [
        "any"
      ]
    },
    {
      "topic": "peter-scaleout1",
      "partition": 1,
      "replicas": [
        3
      ],
      "log_dirs": [
        "any"
      ]
    },
    {
      "topic": "peter-scaleout1",
      "partition": 2,
      "replicas": [
        4
      ],
      "log_dirs": [
        "any"
      ]
    },
    {
      "topic": "peter-scaleout1",
      "partition": 3,
      "replicas": [
        1
      ],
      "log_dirs": [
        "any"
      ]
    }
  ]
}
```

다시 kafka-reassign-partitions.sh 를 사용해서 파티션을 배치시킨다.

```shell
/usr/local/kafka/bin/kafka-reassign-partitions.sh --bootstrap-server peter-kafka01.foo.bar:9092 --reassignment-json-file move.json --execute

```

분산 배치 작업을 할때는 카프카 사용량이 적은 시간때 사용해야 된다 내부적으로 리플리케이션 동작이 일어나기 때문에 브로커의 부하가 걸린다.

# 참고

------
* [Semantic Versioning](https://semver.org/lang/ko/)
* [kafka upgrade](https://kafka.apache.org/documentation/#upgrade)

