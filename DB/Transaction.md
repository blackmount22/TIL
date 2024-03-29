Transaction(트랜잭션) 이란?
---
- 여러 쿼리를 논리적으로 하나의 작업으로 묶어주는 것

```jsx
Ex) 거래가 일어날 때의 과정 
	1. UPDATE문 : 구매자 계좌에서 10,000원 빼기
	2. UPDATE문 : 판매자 계좌에서 10,000원 더하기
위 1,2 작업을 하나의 논리적인 작업 단위로 묶어 주는 것.
```

- 하나의 트랜잭션은 커밋 혹은 롤백된다.
- Transaction의 성질 **ACID**
1. **‘A’ :** Atomicity(원자성) - 트랜잭션은 DB에 모두 반영되거나, 전형 반영되지 않아야 한다.
2. **‘C’ :** Consistency(일관성) **-** 트랜잭션 작업처리결과는 항상 일관성 있어야 한다.
    
    데이터베이스는 항상 일관된 상태로 유지되어야한다
    
3. **‘I’ :** Isolation(독립성) - 둘 이상의 트랜잭션이 동시 실행되고 있을 때, 어떤 트랜잭션도 다른 트랜잭션 연산에 끼어들 수  없다.
4. **‘D’ :** Durability(지속성) - 트랜잭션이 성공적으로 완료되었으면 결과는 영구히 반영되어야한다.

    위 성질을 보장하기 위해 트랜잭션 격리 수준을 설정할 수 있다. 하지만 격리 수준이 높아질수록 성능이 떨어지는 문제점 야기

- READ-UNCOMMITTED : 격리 수준이 가장 낮은 단계로, 커밋 전의 트랜잭션의 데이터 변경 내용을 다른 트랜잭션이 읽는 것을 허용
- READ_COMMITTED : 두번째 격리 레벨, 커밋이 완료된 트랜잭션의 변경사항만 다른 트랜잭션에서 조회 가능
- REPEATABLE-READ : 세번째 격리 레벨, 커밋이 완료된 데이터만 읽을 수 있으며, 한 번 조회 데이터는 다른 트랜잭션이 변경하거나 삭제하는 것을 막는다.
- SERIALIZABLE : 마지막 격리 수준이 높은 트랜잭션, 한 트랜잭션에서 사용하는 데이터를 다른 트랜잭션에서 접근이 불가
