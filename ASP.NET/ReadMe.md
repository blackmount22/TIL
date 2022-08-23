# ✏ [ASP.NET]

---

### 1. 확장 메서드란 무엇인가

 확장 메서드는 C# 3.0 부터 추가된 기능으로 이미 **정의된 형식에 사용자 정의 함수를 확장**시키는 작업을 수행하는 메서드이다.

 간단하게 클래스에 손을 대지 않고 메서드를 추가할 수 있는 방법이라 보면 된다.

### 2. 확장 메서드 선언하는 방법

 (1) static 한정자로 메서드 선언

 (2) 해당 메서드의 첫 번째 매개 변수는 반드시 **this** 키워드와 함께 확장하고자 하는 클래스(형식) 의 인스턴스여야 한다.

- 확장 메서드 생성/호출 예제 **#문자열 공백 카운트**
    
    **확장 메서드 생성**
    
    ```csharp
    public static class ExtenstionString
    {
    	public static int GetSpaceCount(this String str)
    	{
    		if(!string.isNullOrEmpty(str))
    		{
    			string[] spaceArray = str.Split(" ");
    			return spaceArray.Length - 1;
    		} 
    		else 
    		{
    			return 0;
    		}
    	}
    }
    ```
    
    기본 클래스인 String 클래스를 활용하여 **확장 메서드 호출**
    
    ```csharp
    class Program
    {
    	static void Main(string[] args)
    	{
    		string str = "ABC DEL GHI JKL";
    		Console.WriteLine(str.GetSpaceCount());
    	}
    }
    
    [실행결과]
    3
    ```
    

정리하자면 확장 메서드란 **기존 형식의 코드 변경 없이 사용자 임의로 만든 메서드를 대상 형식에 추가** 할 수 있도록 도와주는 메서드이다.

---

### 1. LINQ란 무엇인가

LINQ란 Language Integrate Query의 약자로 통합 질의 언어이다,

기존의 Query는 Database의 데이터를 다루기 위해 사용된 언어이지만, LINQ는 **컬렉션 형태**로 되어있는 모든 데이터에 대해 질의를 할 수 있는 기술이다.

즉, MS-SQL 등 DB 데이터를 가져오는데 LINQ를 사용할 수 있는 것은 물론이고, 메모리상의 컬렉션 또는 XML에 대해서도 LINQ를 사용할 수 있다.

### 2. LINQ와 확장 메서드 관계

확장 메서드 LINQ에서 쿼리를 위한 메서드 Select, Where 등은 확장 메서드로 되어 있기 때문이다.

![Untitled (1)](https://user-images.githubusercontent.com/57103993/186149184-cddb63d8-482b-4855-af8b-973d19f943b2.png)

Aggregate, All, Any 등 파란색 화살표가 아래방향으로 향하는 것이 확장 메서드이다.

LINQ에서는 이처럼 많은 수의 확장 메서드를 사용한다.

### 3. Lambda Expression (람다 표현식)

익명 대리자, 무명 메서드를 좀 더 실용적으로 만든 것으로, 컴파일 과정에서 Delegate로 치환된다.

```csharp
(매개변수들) => (표현식)
```

```csharp
// **홀수** 출력 람다 표현식
// 컴파일 시 1,2번은 3번과 같이 Anonymous Delegate 형태
int[] numbers = {1,2,3,4,5,6,7,8,9}
1. IEnumerable<int> odds = numbers.Where(x=> x%2 == 1);
2. IEnumerable<int> odds = numbers.Where((int x) => x%2 == 1);
3. IEnumerable<int> odds = numbers.Where(delegate(int x) { return x%2 ==1; });

```

---

### 1. DataTable에서 확장 메서드 & LINQ & 람다를 활용

DataTable 에 LINQ 쿼리를 수행하려고 할 때, LINQ 쿼리는 DataTable, DataTable Row에 허용이 되지 않는다.

```csharp
// #1) 회원 아이디가 "kmmun" 인 DataRow 조회

DataTable dt = biz.[SP명]
dt.Where(x => x["MemID"].ToString() == "kmmun"); // DataTable에 접근이 되지 않는다.
dt.Rows[0].Where(x => x["MemID"].ToString() == "kmmun"); // DataRow에 접근이 되지 않는다.
```

AsEnumerable() EnumerableRowCollection 을 활용하여 LINQ 쿼리 작성

```csharp
public static EnumerableRowCollection<DataRow> AsEnumerable(this DataTable source);

DataTable dt = biz.[SP명]
dt.AsEnumerable().Where(x => x["MemID"].ToString() == "kmmun");
```

### 2. DataTable 형 변환, Null 체크

AsEnumerable() 행 컬렉션 LINQ 쿼리 후, CopyToDataTable()를 활용하여 DataTable로 형 변환 할 수 있다. 단, Where() 문 등 조건 이후 LINQ 쿼리 결과가 없을 수 있어 Row 개수를 체크 해준 후 CopyToDataTable() 해줘야한다.

```csharp
DataTable dt = biz.[SP명]
int rowCnt = dt.AsEnumerable().Where(x => x["MemID"].ToString() == "kmmun").Count()

if(rowCnt > 0) 
{
	dt = dt.AsEnumerable().Where(x => x["MemID"].ToString() == "kmmun").CopyToDataTable();
}
```

---

참조 

[http://taeyo.net/Columns/View.aspx?SEQ=207&PSEQ=31&IDX=3](http://taeyo.net/Columns/View.aspx?SEQ=207&PSEQ=31&IDX=3)

[http://daplus.net/c-datatable에-대한-linq-쿼리/](http://daplus.net/c-datatable%EC%97%90-%EB%8C%80%ED%95%9C-linq-%EC%BF%BC%EB%A6%AC/)
