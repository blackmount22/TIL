
![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e0991752-5e90-4295-a726-c6943334f6db/Untitled.png)

NICE 에서 제공해주는 본인인증 서비스를 적용해야 하는 상황이 있었다.

API 개발가이드는 일단 JAVA 언어 기반으로만 예시코드를 제공해주는 상황으로 C# 적용 시 암호화 처리 후 직접 암호화가 잘 되었는지 JAVA 코드라 비교해야 하는 작업이 필요했다.

아래는 API 처리 흐름도와 C# 소스이다.

### ※ 본인인증 API 처리 흐름도

(1) 기관 인증 토큰 API 최초 1회 호출 반영구적(50년)

(2) 1번 ‘기관 인증 토큰’으로 NICE 본인확인 호출을 위한 암호화 토큰 API 호출

(3) 2번에서 발급받은 암호화 토큰으로 대칭키(Symmetric Key, IV)와 무결성키(hmac Key)를 생성

(4) 요청할 데이터(request_data)를 대칭키로 암호화 하고 암호화 데이터를 체크

→ 요청 데이터 암호화는 **AES128/CBC/PKCS7** 패딩 후, Base64 Encoding으로 사용

(5) 생성한 데이터를 form에 추가하여 전송

(6) return URL 을 통해 받은 암호화를 복호화 처리

**‘기관 인증 토큰 API 호출’** 과 **********’암호화 Token 요청 API’********** 총 2번의 API를 호출하고 암호화 키를 생성하여 데이터를 암호화 처리해서 본인인증 폼 레이어를 호출하는 방식이다

각 API 전송 시 필요한 데이터는 아래 사이트에서 신청한 아이디로 로그인 한 후 MyAppList  > 상세보기에서 필요한 데이터 값을 조회 할 수 있다

**사이트 URL : [https://www.niceapi.co.kr/#/](https://www.niceapi.co.kr/#/)**

```cs
string __niceClientID = 사이트에서 제공해주는 값;
string __niceClientSecret = 사이트에서 제공해주는 값;

#region [ 최초 50년(반영구적) Token 생성 / 토큰 만료 시 처리 ]

public string GetNiceApiAccessToken()
{
    string hostName = "https://svc.niceapi.co.kr:22001";
    string requestUrl = "/digital/niceid/oauth/oauth/token";

    string postData = "grant_type=client_credentials&scope=default";
    byte[] sendData = UTF8Encoding.UTF8.GetBytes(postData);

    string url = hostName + requestUrl;

    HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(url);
    request.ContentType = "application/x-www-form-urlencoded";
    request.Method = "POST";
    request.ContentLength = sendData.Length;

    string text = __niceClientID + ":" + __niceClientSecret;

    byte[] byteUTF8 = Encoding.UTF8.GetBytes(text);
    string base64UTF8 = Convert.ToBase64String(byteUTF8);

    request.Headers.Add("Authorization", "Basic " + base64UTF8);

    try
    {
        Stream requestStream = request.GetRequestStream();
        requestStream.Write(sendData, 0, sendData.Length);
        requestStream.Close();

        HttpWebResponse response = (HttpWebResponse)request.GetResponse();
        StreamReader responseStream = new StreamReader(response.GetResponseStream());
        string rawData = responseStream.ReadToEnd();
        parser = new JsonTextParser();
        JObject obj = JObject.Parse(rawData);

        JToken jToken = obj["dataBody"];
        accessToken = jToken.SelectToken("access_token").ToString();

        message = rawData;

        return "OK";
    }
    catch (System.Exception ex)
    {
        message = ex.Message.ToString();
        return ex.Message.ToString();
    }
}

#endregion 
```

```cs
string __accessToken = 인증 기관 토큰 (위 소스에서 부여받은 값);
string __productID = 사이트에서 제공해주는 값;

#region [ 암호화 토큰 발급 ]

private string GetNiceApiEncryptToken()
{
    string hostName = "https://svc.niceapi.co.kr:22001";
    string requestUrl = "/digital/niceid/api/v1.0/common/crypto/token";
    
    string url = hostName + requestUrl;

    req_dtim = DateTime.Now.ToString("yyyyMMddHHmmss");
    req_no = Convert.ToBase64String(Guid.NewGuid().ToByteArray());

    HttpWebRequest request = (HttpWebRequest)HttpWebRequest.Create(url);

    double unixTimeStamp = DateTime.Now.Subtract(new DateTime(1970, 1, 1)).TotalSeconds;
    string text = __accessToken + ":" + (int)unixTimeStamp + ":" + __niceClientID;
    byte[] byteUTF8 = Encoding.UTF8.GetBytes(text);
    string base64UTF8 = Convert.ToBase64String(byteUTF8);

    request.Headers.Add("Authorization", "bearer " + base64UTF8);
    request.Headers.Add("client_id", client_id);
    request.Headers.Add("productID", productID);
    request.ContentType = "application/json";
    request.Method = "POST";

    var streamWriter = new StreamWriter(request.GetRequestStream());

    try
    {
        string strData = "{\"dataHeader\": {\"CNTY_CD\":\"ko\"}, \"dataBody\":{\"req_dtim\":\"" + req_dtim + "\",\"req_no\":\"" + req_no + "\",\"enc_mode\":\"1\"}}";
        JObject jsonData = JObject.Parse(strData);

        streamWriter.Write(strData);
        streamWriter.Flush();
        streamWriter.Close();

        HttpWebResponse response = (HttpWebResponse)request.GetResponse();
        StreamReader responseStream = new StreamReader(response.GetResponseStream());
        string rawData = responseStream.ReadToEnd();

        JsonTextParser parser = new JsonTextParser();
        JObject obj = JObject.Parse(rawData);
        
        JToken jToken = obj["databody"];
        string token_val = jToken.SelectToken("token_val").ToString();
        string token_version_id = jToken.SelectToken("token_version_id").ToString();
        string siteCode = jToken.SelectToken("site_code").ToString();

        JToken jTokenHeader = obj["dataHeader"];
        resultCode = jTokenHeader.SelectToken("GW_RSLT_CD").ToString();

        // 성공시 키 발급 처리
        if (resultCode == "1200")
        {
            // 대칭키
            string strValue = req_dtim.Trim() + req_no.Trim() + token_val.Trim();

            SHA256Managed sha256 = new SHA256Managed();
            byte[] encryptBytes = sha256.ComputeHash(Encoding.ASCII.GetBytes(strValue));

            string strKeyValue = Convert.ToBase64String(encryptBytes);

            // 대칭키(Symmetric Key, IV), 무결성 키 생성
            string sKey = strKeyValue.Substring(0, 16);
            string IV = strKeyValue.Substring(strKeyValue.Length - 16);
            string encryptedHMacKey = strKeyValue.Substring(0, 32);

            string strRequestNo = Convert.ToBase64String(Guid.NewGuid().ToByteArray());

            string strRequestData = "{\"requestno\":\"" + strRequestNo + "\",\"returnurl\":\"" + HttpContext.Current.Request.Url.AbsoluteUri + "\",\"sitecode\":\"" + siteCode + "\",\"methodtype\":\"post\",\"popupyn\":\"Y\"}";

            try
            {
                RijndaelManaged rijndaeManaged = new RijndaelManaged();
                rijndaeManaged.Mode = CipherMode.CBC;
                rijndaeManaged.Padding = PaddingMode.PKCS7;
                rijndaeManaged.Key = Encoding.ASCII.GetBytes(sKey);
                rijndaeManaged.IV = Encoding.ASCII.GetBytes(IV);

                byte[] encDataByte = EncryptStringToBytes(strRequestData, rijndaeManaged.Key, rijndaeManaged.IV);
                enc_data = Convert.ToBase64String(encDataByte);

                byte[] hmacKeyByte = Encoding.UTF8.GetBytes(encryptedHMacKey);
                HMACSHA256 sha = new HMACSHA256(hmacKeyByte);
                byte[] integrityValueByte = sha.ComputeHash(Encoding.ASCII.GetBytes(enc_data));

                integrity_value = Convert.ToBase64String(integrityValueByte);

            }
            catch (System.Exception ex)
            {
                message = ex.Message.ToString();
                return ex.Message.ToString();
            }
        }

        return "";

    }
    catch (System.Exception ex)
    {
        message = ex.Message.ToString();
        return ex.Message.ToString();
    }
}

#endregion

#region [ Byte를 EncryptString으로 변환 ]

protected byte[] EncryptStringToBytes(string plainText, byte[] Key, byte[] IV)
{
    // Check arguments. 
    if (plainText == null || plainText.Length <= 0)
        throw new ArgumentNullException("plainText");
    if (Key == null || Key.Length <= 0)
        throw new ArgumentNullException("Key");
    if (IV == null || IV.Length <= 0)
        throw new ArgumentNullException("Key");
    byte[] encrypted;

    using (RijndaelManaged rijAlg = new RijndaelManaged())
    {
        rijAlg.Key = Key;
        rijAlg.IV = IV;

        // Create a decrytor to perform the stream transform.
        ICryptoTransform encryptor = rijAlg.CreateEncryptor(rijAlg.Key, rijAlg.IV);

        // Create the streams used for encryption. 
        using (MemoryStream msEncrypt = new MemoryStream())
        {
            using (CryptoStream csEncrypt = new CryptoStream(msEncrypt, encryptor, CryptoStreamMode.Write))
            {
                using (StreamWriter swEncrypt = new StreamWriter(csEncrypt))
                {

                    //Write all data to the stream.
                    swEncrypt.Write(plainText);
                }
                encrypted = msEncrypt.ToArray();
            }
        }
    }

    // Return the encrypted bytes from the memory stream. 
    return encrypted;

}

#endregion
```
