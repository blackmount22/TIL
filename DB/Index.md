
## INDEX

📢 Index는 RDBMS에서 검색 속도를 높이기 위한 기술이다.

TABLE의 컬럼을 색인화 (따로 파일로 저장)하여 검색 시 해당 TABLE의 레코드를 Full Scan 하는게 아니라 색인화 되어있는 Index 파일을 검색하여 검색 속도를 빠르게 한다.

RDBMS에서 사용하는 Index 는 B-Tree 에서 파생된 B+Tree 를 사용해서 색인화한다.

보통 SELECT 쿼리의 WHERE 절이나 JOIN 예약어를 사용했을 때 인덱스가 사용되어 SELECT 쿼리의 검색 속도를 빠르게 하는데 목적을 두고 있다.

---

### Index 작동 원리

`SELECT *`
`FROM Employee`
`WHERE empNo = 9000;`

데이터 파일의 블록이 10만개 일 때, 위 SQL 문을 수행 시에 

1. 서버 프로스세가 파싱 과정을 마친 후 DB Buffer Cache에 empNo가 9000인 정보가 있는지 확인한다.

1. 정보가 없으면 하드 디스크 파일에서 9000정보를 가진 블록을 복사해서 DB Buffer Cache로 가져온 후 9000 정보만 골라내서 사용자에게 보여줌

이때 두 가지 경우로 나눌 수 있는데,

- Index 없는 경우: 9000 정보가 어떤 블록에 들어 있는지 모르므로 10만개 전부 db Buffer Cache로 복사한 후 하나 하나 찾는다.
- Index 있는 경우: WHERE 절의 컬럼이 index가 만들어져 있는지 확인 후, 인덱스에 먼저 가서 9000 정보가 어떤 ROWID를 가지고 있는지 확인한 후 해당 ROWID에 있는 블록만 찾아가서 db Buffer Cache에 복사한다.
