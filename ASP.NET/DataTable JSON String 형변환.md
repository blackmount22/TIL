DataTable 데이터를 JSON 으로 형변환하는 방법은 아래와 같이 3가지를 들 수 있습니다.

1.  StringBuilder 사용
2.  JavaScriptSerializer 사용
3.  JSON.NET DLL 을 이용

이름과, 나이를 저장한 테이블을 예제로 사용해서 알아보겠습니다.

``` C#
public DataTable getData()
{
    DataTable dt = new DataTable();
    dt.Columns.Add("Name", typeof(string));
    dt.Columns.Add("Age", typeof(int));
    dt.Rows.Add("홍길동", 20);
    dt.Rows.Add("고길동", 30);
}
```

1.  StringBuilder 사용하여 직접 그리기

``` C#
// DataTable dt
StringBuilder sb = new StringBuilder();

if(dt.Rows.Count >0)
{
    sb.Append("[");

    for(int i=0;i<dt.Rows.Count;i++)
    {
        sb.Append("{");
        for (int j = 0; j < dt.Columns.Count; j++)   
    {  
        if (j < dt.Columns.Count - 1)   
        {  
            sb.Append("\"" + dt.Columns[j].ColumnName.ToString() + "\":" + "\"" + dt.Rows[i][j].ToString() + "\",");  
        }   
        else if (j == table.Columns.Count - 1)   
        {  
            sb.Append("\"" + dt.Columns[j].ColumnName.ToString() + "\":" + "\"" + dt.Rows[i][j].ToString() + "\"");  
        }  
      }  
      if (i == dt.Rows.Count - 1)   
      {  
          sb.Append("}");  
      }   
      else   
      {  
          sb.Append("},");  
      }
    }

    sb.Append("]");
}

sb.ToString();
```

1.  JavaScriptSerializer 이용

``` C#
using System.Web.Script.Serialization;

JavaScriptSerializer jsonSerializer = new JavaScriptSerializer();
List<Dictionary<string, object>> parentRow = new List<Dictionary<string, object>>();

Dictionary < string, object > childRow;  
foreach(DataRow row in table.Rows) 
{  
    childRow = new Dictionary < string, object > ();  
    foreach(DataColumn col in table.Columns) 
    {  
        childRow.Add(col.ColumnName, row[col]);  
    }  
    parentRow.Add(childRow);  
}  

return jsSerializer.Serialize(parentRow);
```

1.  Newtonsoft, [JSON.Net](http://JSON.Net) DLL 활용

``` C#
using Newtonsoft.JSON;

string JSONString = "";
JSONString = JSONConvert.SerializeObject(table);
return JSONString;
```

결과

``` C#
[{"Name":"홍길동","Age":20},{"Name":"고길동","Age":30}]
```
