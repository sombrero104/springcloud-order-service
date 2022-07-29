<br/>

### order-service 주문 요청 예시 
- 요청: 상품 주문 
- Method: POST
- URL: http://127.0.0.1:8000/order-service/[사용자 아이디]/orders
- Body: 
~~~
{
    "productId": "CATALOG-0002",
    "qty": 10,
    "unitPrice": 900
}
~~~
- Response:
~~~
{
    "productId": "CATALOG-0002",
    "qty": 10,
    "unitPrice": 900,
    "totalPrice": 9000,
    "orderId": "868b506a-1925-4410-a68d-4e06d7241a6b"
}
~~~

### order-service 주문 내역 조회 예시 
- 요청: 상품 주문 내역 조회 
- Method: GET
- URL: http://127.0.0.1:8000/order-service/[사용자 아이디]/orders
- Response:
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

## Multiple Orders Service 
user-service 요청 처리의 부하 분산을 위해 <br/>
order-service를 하나 이상 기동한다고 가정했을 때 <br/>
order-service 데이터가 분산 저장되어 동기화 문제가 발생한다. <br/>

> 예를 들어 order-service를 2개 기동한 후 <br/>
> 한 사용자가 3번 주문을 해서 <br/>
> 첫번째 order-service에는 1개의 주문내역, <br/>
> 두번째 order-service에는 2개의 주문내역이 저장되었을 경우 <br/>
> 같은 사용자임에도 불구하고 주문내역을 조회할 때 <br/>
> 어떤 경우에는 조회 결과가 1개, 어떤 경우에는 2개로 조회 결과가 나오게 된다. <br/>

### 해결 방법?
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

<br/><br/><br/><br/>
