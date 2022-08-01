<br/>

# order-service 

### 주문 요청 
- 요청: 상품 주문 
- Method: POST
- URL: http://127.0.0.1:8000/order-service/[사용자 아이디]/orders
- Request Body: 
~~~
{
    "productId": "CATALOG-0002",
    "qty": 10,
    "unitPrice": 900
}
~~~
- Response Body:
~~~
{
    "productId": "CATALOG-0002",
    "qty": 10,
    "unitPrice": 900,
    "totalPrice": 9000,
    "orderId": "868b506a-1925-4410-a68d-4e06d7241a6b"
}
~~~

### 주문 내역 조회 
- 요청: 상품 주문 내역 조회 
- Method: GET
- URL: http://127.0.0.1:8000/order-service/[사용자 아이디]/orders
- Response Body:
~~~
[
    {
        "productId": "CATALOG-0007",
        "qty": 10,
        "unitPrice": 900,
        "totalPrice": 9000,
        "createdAt": "2022-07-29T17:00:27.025+00:00",
        "orderId": "4f06ef6a-5d84-4f1e-9765-627b386b184b"
    },
    {
        "productId": "CATALOG-0009",
        "qty": 10,
        "unitPrice": 900,
        "totalPrice": 9000,
        "createdAt": "2022-07-29T17:00:36.731+00:00",
        "orderId": "bb0bd2e9-2265-4854-9c37-8045bc63d6b7"
    }
]
~~~
<br/><br/>

# Multiple Orders Service 
user-service 요청 처리의 부하 분산을 위해 <br/>
order-service를 하나 이상 기동한다고 가정했을 때 <br/>
order-service 데이터가 분산 저장되어 동기화 문제가 발생한다. <br/>

> 예를 들어 order-service를 2개 기동한 후 <br/>
> 한 사용자가 3번 주문을 해서 <br/>
> 첫번째 order-service에는 1개의 주문내역, <br/>
> 두번째 order-service에는 2개의 주문내역이 저장되었을 경우 <br/>
> 같은 사용자임에도 불구하고 주문내역을 조회할 때 <br/>
> 어떤 경우에는 조회 결과가 1개, 어떤 경우에는 2개로 조회 결과가 나오게 된다. <br/>

## 해결 방법?
- 하나의 데이터베이스 사용 
- 데이터베이스 간의 동기화 
    - Message Queuing Server 사용 (Apache Kafka, RabbitMQ) <br/>
    한쪽에 새로 업데이트 된 데이터를 Message Queuing Server로 전달 <br/>
    다른쪽 서비스는 Message Queuing Server로 구독 신청(변경된 데이터가 있으면 알려줌.)을 해서 <br/>
    변경된 데이터를 가져와서 자신의 데이터베이스에도 업데이트. <br/>
- Kafka Connector + DB
    - Message Queuing Server와 하나의 데이터베이스 사용. 
    - Message Queuing Server에 데이터를 전달한 후 <br/>
    Message Queuing Server에서 데이터베이스에 업데이트.
    - 많은 요청 처리 시 동시성 문제를 Kafka에 일임. 

<br/><br/>
    
# Apache Kafka
- Apache Software Foundation 의 Scalar 언어로 된 오픈 소스 메시지 브로커 프로젝트 
- Linked-in 에서 개발, 2011년 오픈 소스화 
    - 2014년 11월 링크드인에서 Kafka를 개발하던 엔지니어들이 Kafka 개발에 집중하기 위해 Confluent 라는 회사 창립 
- 실시간 데이터 피드를 관리하기 위해 통일된 높은 처리량, 낮은 지연 시간을 지닌 플랫폼 제공

## Kafka Broker
- 실행 된 Kafka 애플리케이션 서버
- 3대 이상의 Broker Cluster 구성 권장 
- Zookeeper 연동 
    - Broker Cluster 서버들을 관리해 주는 코디네이터 역할 (~~동물원 사육사~~)
    - 메타데이터 (Broker ID, Controller ID 등) 저장
    - Controller 정보 저장 
- n개 Broker 중 1대는 Controller 기능 수행 
    - Controller 역할
        - 각 Broker에게 담당 파티션 할당 수행 
        - Broker 정상 동작 모니터링 관리 
<br/>

# Ecosystem 1. Kafka Client
- https://docs.confluent.io/platform/current/clients/index.html
- Kafka 와 데이터를 주고 받기 위해 사용하는 Java Library
    - https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients
- Producer, Consumer, Admin, Stream 등 Kafka 관련 API 제공
- 다양한 3rd party library 존재 (C/C++, Node.js, Python, .NET 등)
    - https://cwiki.apache.org/confluence/display/kafka/clients

## Kafka 설치
https://kafka.apache.org 에서 다운로드 받은 후 압축 해제.

## Kafka 서버 기동
### Zookeeper 및 Kafka 서버 구동
~~~
$KAFKA_HOME/bin/zookeeper-server-start.sh $KAFKA_HOME/config/zookeeper.properties
$KAFKA_HOME/bin/kafka-server-start.sh $KAFKA_HOME/config/server.properties
~~~
producer 들이 메시지를 생성해서 보내게 되면 Topic 에 저장이 된다. <br/>
새로운 메시지는 Topic 에 등록한 consumer 에게 전달된다. <br/> 
### Topic 생성 
~~~
$KAFKA_HOME/bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092 --partitions 1
    => --create 옵션을 사용하여 quickstart-events 이름으로 토픽을 생성. 
~~~
### Topic 목록 확인
~~~
$KAFKA_HOME/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
~~~
### Topic 정보 확인
~~~
$KAFKA_HOME/bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
~~~

## Kafka Producer/Consumer 테스트 
### 메시지 생산 
~~~
$KAFKA_HOME/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic quickstart-events 
~~~
### 메시지 소비 
~~~
$KAFKA_HOME/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events --from-beginning 
~~~
#### [테스트 결과] 
<img src="./images/test_kafka_producer.png" width="54%" /><br/>
<img src="./images/test_kafka_consumer.png" width="54%" /><br/>
<br/>

# Ecosystem 2. Kafka Connect
- Kafka Connect를 통해 Data를 Import/Export 가능
- 코드 없이 Configuration으로 데이터를 이동
- Standalone mode, Distribution mode 지원
    - RESTful API 통해 지원
    - Stream 또는 Batch 형태로 데이터 전송 가능
    - 커스텀 Connector를 통한 다양한 Plugin 제공 (File, S3, Hive, Mysql 등)
        > Source System(Hive, jdbc..) <br/>
         -> Kafka Connect Source <br/>
         -> Kafka Cluster <br/>
         -> Kafka Connect Sink <br/>
         -> Target System(S3..) 


<br/><br/><br/><br/>
