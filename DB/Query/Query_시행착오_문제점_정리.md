
### 1. WHERE 절 IF 문 처리
---
Query 작성 중 WHERE 절에 IF 문이 필요한 경우가 생긴다.
일반적인 프로그램 코드로 생각할 경우 아래와 같이 코드를 작성할 것이다.

```sql
IF (Column = @Test)
BEGIN
  WHERE Column2 = 'A'
END
ELSE IF (Column = @Test2)
BEGIN
  WHERE Column2 = 'B'
END
```

하지만 IF 문 조건과 WHERE 문구를 잘 생각하보면 괄호 속 OR 조건으로 묶일 수 있다.

```sql
WHERE -- 기본 조건
AND (
  (Column = @Test AND Column2 = 'A') OR
  (Column = @Test2 AND Column2 = 'B')
)
```
<br>
