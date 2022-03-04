# GET

GET 메소드는 클라이언트에서 서버로 어떠한 리소스로 부터 정보를 요청하기 위해 사용되는 메소드

즉, 데이터를 읽거나, 검색 할때 사용되는 method로 예를 들면 게시판의 게시물을 조회할 때 사용된다.

GET은 요청을 전송할 때 URL 주소 끝에 파라미터로 포함되어 전송되며 이 부분을 Query String(쿼리스트링) 이라 부른다.

GET Method는 오로지 데이터를 읽을 때만 사용되며 수정할 때는 사용하지 않는다.

URL 예시) www.example.com?name1=john&name2=mount

### GET 특징

- GET 요청은 캐시가 가능하다.
- GET 요청은 브라우저 히스토리에 남는다.
- 파라미터에 내용이 노출되기 때문에 민감한 데이터를 다룰 때 GET 요청을 사용해서는 안된다.
- GET 요청은 데이터 길이에 대한 제한이 있다.

# POST

POST 메소드는 클라이언트에서 서버로 리소스를 생성/업데이트 하기 위해 데이터를 보내는데 사용되는 메소드

GET과 달리 POST는 전송할 데이터를 HTTP 메세지 Body에 담아서 전송한다.

그리고 Body의 타입은 요청 헤더의 Content-Type에 따라 결정된다

HTTP 메세지의 Body는 길이의 제한 없이 데이터를 전송할 수 있다. 그래서 POST 요청은 GET과 달리 대용량 데이터를 전송할 수 있다.

URL 예시) www.example.com

### POST 특징

- POST 요청은 캐시 되지 않는다.
- POST 요청은 브라우저 히스토리에 남지 않는다.
- POST 요청은 데이터 길이에 제한이 없다.

### GET / POST 차이점

- 사용 목적:
    - GET 메소드 : 서버의 리소스에서 데이터를 요청할 떄 (SELECT)
    - POST 메소드: 서버의 리소스를 새로 생성하거나 업데이트 할 때 사용 (CREATE)
- Body 유무:
    - GET: URL 파라미터에 요청하는 데이터를 담아 보내기 때문에 HTTP 메시지에 Body가 없다
    - POST: Body에 데이터를 담아 보내기 때문에 HTTP 메시지에 Body가 존재
- Idempotent (멱등성):
    - GET 요청은 멱등이며, POST는 아니다


### Idempotent(멱등성)

- Idempotent (멱등)의 수학적 개념은 다음과 같다

    → 수학이나 전산학에서 연산의 한 성질을 나타내는 것으로, 연산을 여러 번 적용하더라도 결과가 달라지지 않는 성질

    - GET은 리소스를 조회한다는 점에서 여러 번 요청하더라도 응답이 똑같다.
    - POST는 서버에게 동일한 요청을 여러 번 전송해도 응답은 항상 다를 수 있다.(Non-Idempotent)


GET 과 POST는 이와 같이 큰 차이가 있기에 적절한 용도에 맞게 사용해야 합니다.

참조 페이지 : [https://velog.io/@songyouhyun/Get과-Post의-차이를-아시나요](https://velog.io/@songyouhyun/Get%EA%B3%BC-Post%EC%9D%98-%EC%B0%A8%EC%9D%B4%EB%A5%BC-%EC%95%84%EC%8B%9C%EB%82%98%EC%9A%94)
