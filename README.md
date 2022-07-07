<br/>

### order-service 요청 예시 
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

<br/><br/><br/><br/>
