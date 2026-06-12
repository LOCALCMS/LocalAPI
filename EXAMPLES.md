# LOCALCMS API — 언어별 호출 예제

> 🚧 **본 문서는 예시(샘플)입니다.**
> 현재 실제 **마이그레이션·운영 적용 전** 단계이며, 엔드포인트 경로·파라미터·정책(토큰 유효시간, 호출 한도 등)은 **확정되지 않았습니다.**
> 실제 연동 전 반드시 담당자 확인이 필요합니다.


각 엔드포인트를 언어별로 호출하는 예제입니다.

> ⚠️ 모든 호출은 **HTTPS** 로만 가능합니다. `주소`, `귀사의식별번호`, `귀사의KEY`, `{TOKEN}` 부분을 실제 발급값으로 바꿔 사용하세요.
>
> ⚠️ API KEY(uuid)·토큰은 소스코드에 하드코딩하지 말고 **환경변수**로 관리하는 것을 권장합니다. (예제는 가독성을 위해 직접 표기)

## 인증 방식 요약

| 방식 | 인증 헤더 | 비고 |
|------|-----------|------|
| KEY 방식 | `orgcode`, `uuid` | 영구 키. 서버 간 통신에만 사용 |
| TOKEN 방식 | `authorization: Bearer {TOKEN}` | `/auth/token` 으로 발급. 만료 시 재발급 |

> TOKEN 방식은 토큰 자체가 이용기관을 식별하므로 `orgcode`·`uuid` 헤더가 필요 없습니다.

## 목차

1. [토큰 발급](#1-토큰-발급) — `POST /auth/token`
2. [납부내역 조회](#2-납부내역-조회)
   - [2-A. KEY 방식](#2-a-key-방식--get-apilpaylist) — `GET /api/lpaylist/`
   - [2-B. TOKEN 방식](#2-b-token-방식--get-apitpaylist) — `GET /api/tpaylist/`
3. [납부자 등록](#3-납부자-등록)
   - [3-A. KEY 방식](#3-a-key-방식--post-apilcustins) — `POST /api/lcustins/`
   - [3-B. TOKEN 방식](#3-b-token-방식--post-apitcustins) — `POST /api/tcustins/`

---

## 1. 토큰 발급

`POST /auth/token` — `orgcode`, `uuid` 를 form-urlencoded 바디로 전송합니다. 응답으로 Access Token을 받습니다.

### cURL

```bash
curl -X POST "https://주소/auth/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "orgcode=귀사의식별번호&uuid=귀사의KEY"
```

### PHP

```php
<?php
$curl = curl_init();
curl_setopt_array($curl, array(
  CURLOPT_URL => 'https://주소/auth/token',
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_CUSTOMREQUEST => 'POST',
  CURLOPT_POSTFIELDS => http_build_query(array(
    'orgcode' => '귀사의식별번호',
    'uuid'    => '귀사의KEY',
  )),
  CURLOPT_HTTPHEADER => array(
    'Content-Type: application/x-www-form-urlencoded',
  ),
));
$response = curl_exec($curl);
$code = curl_getinfo($curl, CURLINFO_HTTP_CODE);
curl_close($curl);
echo $code . "\n" . $response;
```

### Python (requests)

```python
import requests

url = "https://주소/auth/token"
data = {"orgcode": "귀사의식별번호", "uuid": "귀사의KEY"}
res = requests.post(url, data=data, timeout=10)
print(res.status_code)
print(res.text)
```

### Node.js (fetch, v18+)

```javascript
const body = new URLSearchParams({
  orgcode: "귀사의식별번호",
  uuid: "귀사의KEY",
});

const res = await fetch("https://주소/auth/token", {
  method: "POST",
  headers: { "Content-Type": "application/x-www-form-urlencoded" },
  body,
});
console.log(res.status);
console.log(await res.text());
```

### Java (11+ HttpClient)

```java
import java.net.URI;
import java.net.http.*;

HttpClient client = HttpClient.newHttpClient();
String form = "orgcode=귀사의식별번호&uuid=귀사의KEY";

HttpRequest req = HttpRequest.newBuilder()
    .uri(URI.create("https://주소/auth/token"))
    .header("Content-Type", "application/x-www-form-urlencoded")
    .POST(HttpRequest.BodyPublishers.ofString(form))
    .build();

HttpResponse<String> res = client.send(req, HttpResponse.BodyHandlers.ofString());
System.out.println(res.statusCode());
System.out.println(res.body());
```

### C# (HttpClient)

```csharp
using var client = new HttpClient();
var form = new FormUrlEncodedContent(new[]
{
    new KeyValuePair<string,string>("orgcode", "귀사의식별번호"),
    new KeyValuePair<string,string>("uuid", "귀사의KEY"),
});

var res = await client.PostAsync("https://주소/auth/token", form);
Console.WriteLine((int)res.StatusCode);
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

---

## 2. 납부내역 조회

조회 파라미터(`stdt`, `enddt`, `custcd`, `value1`, `value_type`, `sum_type`, `seltype`)를 **HTTP 헤더**로 전송합니다.
예시는 "납부자번호로 개별 납부내역을 실출금일 기준 조회" (`value_type=1`, `sum_type=0`, `seltype=0`) 입니다.

### 2-A. KEY 방식 — `GET /api/lpaylist/`

#### cURL

```bash
curl -X GET "https://주소/api/lpaylist/" \
  -H "orgcode: 귀사의식별번호" \
  -H "uuid: 귀사의KEY" \
  -H "stdt: 20240101" \
  -H "enddt: 20241231" \
  -H "custcd: 납부자번호" \
  -H "value1: " \
  -H "value_type: 1" \
  -H "sum_type: 0" \
  -H "seltype: 0"
```

#### PHP

```php
<?php
$curl = curl_init();
curl_setopt_array($curl, array(
  CURLOPT_URL => 'https://주소/api/lpaylist/',
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_CUSTOMREQUEST => 'GET',
  CURLOPT_HTTPHEADER => array(
    'orgcode: 귀사의식별번호',
    'uuid: 귀사의KEY',
    'stdt: 20240101',
    'enddt: 20241231',
    'custcd: 납부자번호',
    'value1: ',
    'value_type: 1',
    'sum_type: 0',
    'seltype: 0',
  ),
));
$response = curl_exec($curl);
curl_close($curl);
echo $response;
```

#### Python (requests)

```python
import requests

url = "https://주소/api/lpaylist/"
headers = {
    "orgcode": "귀사의식별번호",
    "uuid": "귀사의KEY",
    "stdt": "20240101",
    "enddt": "20241231",
    "custcd": "납부자번호",
    "value1": "",
    "value_type": "1",
    "sum_type": "0",
    "seltype": "0",
}
res = requests.get(url, headers=headers, timeout=10)
print(res.status_code)
print(res.json())
```

#### Node.js (fetch, v18+)

```javascript
const res = await fetch("https://주소/api/lpaylist/", {
  method: "GET",
  headers: {
    "orgcode": "귀사의식별번호",
    "uuid": "귀사의KEY",
    "stdt": "20240101",
    "enddt": "20241231",
    "custcd": "납부자번호",
    "value1": "",
    "value_type": "1",
    "sum_type": "0",
    "seltype": "0",
  },
});
console.log(res.status);
console.log(await res.json());
```

#### Java (11+ HttpClient)

```java
import java.net.URI;
import java.net.http.*;

HttpClient client = HttpClient.newHttpClient();
HttpRequest req = HttpRequest.newBuilder()
    .uri(URI.create("https://주소/api/lpaylist/"))
    .header("orgcode", "귀사의식별번호")
    .header("uuid", "귀사의KEY")
    .header("stdt", "20240101")
    .header("enddt", "20241231")
    .header("custcd", "납부자번호")
    .header("value1", "")
    .header("value_type", "1")
    .header("sum_type", "0")
    .header("seltype", "0")
    .GET()
    .build();

HttpResponse<String> res = client.send(req, HttpResponse.BodyHandlers.ofString());
System.out.println(res.statusCode());
System.out.println(res.body());
```

#### C# (HttpClient)

```csharp
using var client = new HttpClient();
var req = new HttpRequestMessage(HttpMethod.Get, "https://주소/api/lpaylist/");
req.Headers.Add("orgcode", "귀사의식별번호");
req.Headers.Add("uuid", "귀사의KEY");
req.Headers.Add("stdt", "20240101");
req.Headers.Add("enddt", "20241231");
req.Headers.Add("custcd", "납부자번호");
req.Headers.Add("value1", "");
req.Headers.Add("value_type", "1");
req.Headers.Add("sum_type", "0");
req.Headers.Add("seltype", "0");

var res = await client.SendAsync(req);
Console.WriteLine((int)res.StatusCode);
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

### 2-B. TOKEN 방식 — `GET /api/tpaylist/`

KEY 방식과 동일하되, `orgcode`·`uuid` 헤더 대신 `authorization: Bearer {TOKEN}` 을 사용합니다. 나머지 조회 파라미터는 동일합니다.

#### cURL

```bash
curl -X GET "https://주소/api/tpaylist/" \
  -H "authorization: Bearer {TOKEN}" \
  -H "stdt: 20240101" \
  -H "enddt: 20241231" \
  -H "custcd: 납부자번호" \
  -H "value1: " \
  -H "value_type: 1" \
  -H "sum_type: 0" \
  -H "seltype: 0"
```

#### PHP

```php
<?php
$curl = curl_init();
curl_setopt_array($curl, array(
  CURLOPT_URL => 'https://주소/api/tpaylist/',
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_CUSTOMREQUEST => 'GET',
  CURLOPT_HTTPHEADER => array(
    'authorization: Bearer {TOKEN}',
    'stdt: 20240101',
    'enddt: 20241231',
    'custcd: 납부자번호',
    'value1: ',
    'value_type: 1',
    'sum_type: 0',
    'seltype: 0',
  ),
));
$response = curl_exec($curl);
curl_close($curl);
echo $response;
```

#### Python (requests)

```python
import requests

url = "https://주소/api/tpaylist/"
headers = {
    "authorization": "Bearer {TOKEN}",
    "stdt": "20240101",
    "enddt": "20241231",
    "custcd": "납부자번호",
    "value1": "",
    "value_type": "1",
    "sum_type": "0",
    "seltype": "0",
}
res = requests.get(url, headers=headers, timeout=10)
print(res.status_code)
print(res.json())
```

#### Node.js (fetch, v18+)

```javascript
const res = await fetch("https://주소/api/tpaylist/", {
  method: "GET",
  headers: {
    "authorization": "Bearer {TOKEN}",
    "stdt": "20240101",
    "enddt": "20241231",
    "custcd": "납부자번호",
    "value1": "",
    "value_type": "1",
    "sum_type": "0",
    "seltype": "0",
  },
});
console.log(res.status);
console.log(await res.json());
```

#### Java (11+ HttpClient)

```java
import java.net.URI;
import java.net.http.*;

HttpClient client = HttpClient.newHttpClient();
HttpRequest req = HttpRequest.newBuilder()
    .uri(URI.create("https://주소/api/tpaylist/"))
    .header("authorization", "Bearer {TOKEN}")
    .header("stdt", "20240101")
    .header("enddt", "20241231")
    .header("custcd", "납부자번호")
    .header("value1", "")
    .header("value_type", "1")
    .header("sum_type", "0")
    .header("seltype", "0")
    .GET()
    .build();

HttpResponse<String> res = client.send(req, HttpResponse.BodyHandlers.ofString());
System.out.println(res.statusCode());
System.out.println(res.body());
```

#### C# (HttpClient)

```csharp
using var client = new HttpClient();
var req = new HttpRequestMessage(HttpMethod.Get, "https://주소/api/tpaylist/");
req.Headers.Add("authorization", "Bearer {TOKEN}");
req.Headers.Add("stdt", "20240101");
req.Headers.Add("enddt", "20241231");
req.Headers.Add("custcd", "납부자번호");
req.Headers.Add("value1", "");
req.Headers.Add("value_type", "1");
req.Headers.Add("sum_type", "0");
req.Headers.Add("seltype", "0");

var res = await client.SendAsync(req);
Console.WriteLine((int)res.StatusCode);
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

---

## 3. 납부자 등록

`orgcode`, `uuid`(또는 토큰), `Content-Type` 은 **헤더**로, 나머지 등록 정보는 **form-urlencoded 바디**로 전송합니다.
자동채번 이용기관은 `custcd` 를 공란으로 둡니다. (대부분 공란)

### 3-A. KEY 방식 — `POST /api/lcustins/`

#### cURL

```bash
curl -X POST "https://주소/api/lcustins/" \
  -H "orgcode: 귀사의식별번호" \
  -H "uuid: 귀사의KEY" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode "orgcode=귀사의식별번호" \
  --data-urlencode "custcd=" \
  --data-urlencode "custnm=홍길동" \
  --data-urlencode "telno1=02-1111-1111" \
  --data-urlencode "hphoneno=010-1234-1234" \
  --data-urlencode "address1=대한민국" \
  --data-urlencode "address2=좋은집 1층" \
  --data-urlencode "mailaddr=email@email.com" \
  --data-urlencode "remark=비고사항" \
  --data-urlencode "bankcd=001" \
  --data-urlencode "accountno=1-1111-1111-1" \
  --data-urlencode "depositer=홍길동" \
  --data-urlencode "depjuminno=111111" \
  --data-urlencode "payday=05" \
  --data-urlencode "paymoney=10000" \
  --data-urlencode "paystartdt=20240101"
```

#### PHP

```php
<?php
$curl = curl_init();
curl_setopt_array($curl, array(
  CURLOPT_URL => 'https://주소/api/lcustins/',
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_CUSTOMREQUEST => 'POST',
  CURLOPT_HTTPHEADER => array(
    'orgcode: 귀사의식별번호',
    'uuid: 귀사의KEY',
    'Content-Type: application/x-www-form-urlencoded',
  ),
  CURLOPT_POSTFIELDS => http_build_query(array(
    'orgcode'    => '귀사의식별번호',
    'custcd'     => '',
    'custnm'     => '홍길동',
    'telno1'     => '02-1111-1111',
    'hphoneno'   => '010-1234-1234',
    'address1'   => '대한민국',
    'address2'   => '좋은집 1층',
    'mailaddr'   => 'email@email.com',
    'remark'     => '비고사항',
    'bankcd'     => '001',
    'accountno'  => '1-1111-1111-1',
    'depositer'  => '홍길동',
    'depjuminno' => '111111',
    'payday'     => '05',
    'paymoney'   => '10000',
    'paystartdt' => '20240101',
  )),
));
$response = curl_exec($curl);
echo curl_getinfo($curl, CURLINFO_HTTP_CODE) . "\n";  // 200 확인
echo $response;
curl_close($curl);
```

#### Python (requests)

```python
import requests

url = "https://주소/api/lcustins/"
headers = {
    "orgcode": "귀사의식별번호",
    "uuid": "귀사의KEY",
    "Content-Type": "application/x-www-form-urlencoded",
}
data = {
    "orgcode": "귀사의식별번호",
    "custcd": "",
    "custnm": "홍길동",
    "telno1": "02-1111-1111",
    "hphoneno": "010-1234-1234",
    "address1": "대한민국",
    "address2": "좋은집 1층",
    "mailaddr": "email@email.com",
    "remark": "비고사항",
    "bankcd": "001",
    "accountno": "1-1111-1111-1",
    "depositer": "홍길동",
    "depjuminno": "111111",
    "payday": "05",
    "paymoney": "10000",
    "paystartdt": "20240101",
}
res = requests.post(url, headers=headers, data=data, timeout=10)
print(res.status_code)   # 200 확인
print(res.text)
```

#### Node.js (fetch, v18+)

```javascript
const body = new URLSearchParams({
  orgcode: "귀사의식별번호",
  custcd: "",
  custnm: "홍길동",
  telno1: "02-1111-1111",
  hphoneno: "010-1234-1234",
  address1: "대한민국",
  address2: "좋은집 1층",
  mailaddr: "email@email.com",
  remark: "비고사항",
  bankcd: "001",
  accountno: "1-1111-1111-1",
  depositer: "홍길동",
  depjuminno: "111111",
  payday: "05",
  paymoney: "10000",
  paystartdt: "20240101",
});

const res = await fetch("https://주소/api/lcustins/", {
  method: "POST",
  headers: {
    "orgcode": "귀사의식별번호",
    "uuid": "귀사의KEY",
    "Content-Type": "application/x-www-form-urlencoded",
  },
  body,
});
console.log(res.status);   // 200 확인
console.log(await res.text());
```

#### Java (11+ HttpClient)

```java
import java.net.URI;
import java.net.http.*;
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;
import java.util.*;
import java.util.stream.Collectors;

Map<String, String> params = new LinkedHashMap<>();
params.put("orgcode", "귀사의식별번호");
params.put("custcd", "");
params.put("custnm", "홍길동");
params.put("telno1", "02-1111-1111");
params.put("hphoneno", "010-1234-1234");
params.put("address1", "대한민국");
params.put("address2", "좋은집 1층");
params.put("mailaddr", "email@email.com");
params.put("remark", "비고사항");
params.put("bankcd", "001");
params.put("accountno", "1-1111-1111-1");
params.put("depositer", "홍길동");
params.put("depjuminno", "111111");
params.put("payday", "05");
params.put("paymoney", "10000");
params.put("paystartdt", "20240101");

String form = params.entrySet().stream()
    .map(e -> URLEncoder.encode(e.getKey(), StandardCharsets.UTF_8) + "="
            + URLEncoder.encode(e.getValue(), StandardCharsets.UTF_8))
    .collect(Collectors.joining("&"));

HttpClient client = HttpClient.newHttpClient();
HttpRequest req = HttpRequest.newBuilder()
    .uri(URI.create("https://주소/api/lcustins/"))
    .header("orgcode", "귀사의식별번호")
    .header("uuid", "귀사의KEY")
    .header("Content-Type", "application/x-www-form-urlencoded")
    .POST(HttpRequest.BodyPublishers.ofString(form, StandardCharsets.UTF_8))
    .build();

HttpResponse<String> res = client.send(req, HttpResponse.BodyHandlers.ofString());
System.out.println(res.statusCode());   // 200 확인
System.out.println(res.body());
```

#### C# (HttpClient)

```csharp
using var client = new HttpClient();
var req = new HttpRequestMessage(HttpMethod.Post, "https://주소/api/lcustins/");
req.Headers.Add("orgcode", "귀사의식별번호");
req.Headers.Add("uuid", "귀사의KEY");

req.Content = new FormUrlEncodedContent(new[]
{
    new KeyValuePair<string,string>("orgcode", "귀사의식별번호"),
    new KeyValuePair<string,string>("custcd", ""),
    new KeyValuePair<string,string>("custnm", "홍길동"),
    new KeyValuePair<string,string>("telno1", "02-1111-1111"),
    new KeyValuePair<string,string>("hphoneno", "010-1234-1234"),
    new KeyValuePair<string,string>("address1", "대한민국"),
    new KeyValuePair<string,string>("address2", "좋은집 1층"),
    new KeyValuePair<string,string>("mailaddr", "email@email.com"),
    new KeyValuePair<string,string>("remark", "비고사항"),
    new KeyValuePair<string,string>("bankcd", "001"),
    new KeyValuePair<string,string>("accountno", "1-1111-1111-1"),
    new KeyValuePair<string,string>("depositer", "홍길동"),
    new KeyValuePair<string,string>("depjuminno", "111111"),
    new KeyValuePair<string,string>("payday", "05"),
    new KeyValuePair<string,string>("paymoney", "10000"),
    new KeyValuePair<string,string>("paystartdt", "20240101"),
});
// Content-Type 은 FormUrlEncodedContent 가 자동 설정합니다.

var res = await client.SendAsync(req);
Console.WriteLine((int)res.StatusCode);   // 200 확인
Console.WriteLine(await res.Content.ReadAsStringAsync());
```

### 3-B. TOKEN 방식 — `POST /api/tcustins/`

> ⚠️ **확인 필요:** 원본 명세에는 토큰 방식 등록 엔드포인트(`tcustins`)가 명시되어 있지 않습니다. 실제 제공 여부를 확인한 뒤 사용하세요. 제공된다면 KEY 방식과 동일하며, `orgcode`·`uuid` 헤더만 `authorization: Bearer {TOKEN}` 으로 교체하면 됩니다.

#### cURL

```bash
curl -X POST "https://주소/api/tcustins/" \
  -H "authorization: Bearer {TOKEN}" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  --data-urlencode "orgcode=귀사의식별번호" \
  --data-urlencode "custcd=" \
  --data-urlencode "custnm=홍길동" \
  --data-urlencode "telno1=02-1111-1111" \
  --data-urlencode "hphoneno=010-1234-1234" \
  --data-urlencode "bankcd=001" \
  --data-urlencode "accountno=1-1111-1111-1" \
  --data-urlencode "depositer=홍길동" \
  --data-urlencode "depjuminno=111111" \
  --data-urlencode "payday=05" \
  --data-urlencode "paymoney=10000" \
  --data-urlencode "paystartdt=20240101"
```

#### Python (requests)

```python
import requests

url = "https://주소/api/tcustins/"
headers = {
    "authorization": "Bearer {TOKEN}",
    "Content-Type": "application/x-www-form-urlencoded",
}
data = {
    "orgcode": "귀사의식별번호",
    "custcd": "",
    "custnm": "홍길동",
    "telno1": "02-1111-1111",
    "hphoneno": "010-1234-1234",
    "bankcd": "001",
    "accountno": "1-1111-1111-1",
    "depositer": "홍길동",
    "depjuminno": "111111",
    "payday": "05",
    "paymoney": "10000",
    "paystartdt": "20240101",
}
res = requests.post(url, headers=headers, data=data, timeout=10)
print(res.status_code)
print(res.text)
```

> 나머지 언어(PHP / Node.js / Java / C#)는 위 **3-A KEY 방식 등록** 예제에서
> `orgcode`·`uuid` 헤더를 `authorization: Bearer {TOKEN}` 한 줄로 교체하면 동일하게 동작합니다.

---

## 응답 처리 참고

- 정상 처리는 HTTP `200` 으로 확인합니다.
- 에러 응답 코드(`400/401/403/429` 등)는 각 명세 문서의 "응답 코드" 표를 참고하세요.
- 한글이 포함된 바디는 UTF-8 인코딩으로 전송됩니다. (Java 예제는 `StandardCharsets.UTF_8` 명시)
- TOKEN 방식 호출 중 `401` 이 반환되면 토큰이 만료된 것이므로, `/auth/token` 으로 재발급 후 재시도하세요.
