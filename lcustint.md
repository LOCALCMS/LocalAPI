# lcustint 요청 명세 (납부자 등록 / KEY 방식)

> 🚧 **본 문서는 예시(샘플)입니다.**
> 현재 실제 **마이그레이션·운영 적용 전** 단계이며, 엔드포인트 경로·파라미터·정책(토큰 유효시간, 호출 한도 등)은 **확정되지 않았습니다.**
> 실제 연동 전 반드시 담당자 확인이 필요합니다.


납부자를 등록하는 API입니다.

> ⚠️ 반드시 **HTTPS** 로만 호출하세요. uuid(KEY) 및 등록 정보가 평문으로 노출되지 않도록 주의합니다.

## 참고

### 필수여부
- `O` : 필수 / `X` : 필수아님 / `OX` : 조건에 따라 달라짐

### 포맷
- `A` : 영문 / `N` : 숫자 / `H` : 한글 / `(N)` : 자리수
- `YYYY` : 네자리 년도 / `MM` : 두자리 월 / `DD` : 두자리 일

---

## HEADER

| Parameter    | Type   | 필수여부 | 포맷(Byte) | 설명 |
|--------------|--------|:--------:|:----------:|------|
| orgcode      | String | O | N(10)  | 귀사의 이용기관 식별번호 |
| uuid         | String | O | AN(64) | 귀사의 KEY |
| Content-Type | String | O |        | `application/x-www-form-urlencoded` (고정값) |

---

## BODY (POST)

| Parameter   | Type   | 필수여부 | 포맷(Byte)  | 설명 |
|-------------|--------|:--------:|:-----------:|------|
| orgcode     | String | O | N(10)       | 귀사의 이용기관 식별번호 |
| custcd      | String | X | AN(20)      | 납부자번호 / 자동채번 이용기관은 공란 |
| custnm      | String | X | AHN(40)     | 납부자명 |
| telno1      | String | X | AN(15)      | 전화번호 |
| hphoneno    | String | X | AN(15)      | 휴대번호 |
| zipcode     | String | X | AN(7)       | 우편번호 |
| address1    | String | X | AHN(80)     | 주소 |
| address2    | String | X | AHN(80)     | 상세주소 |
| mailaddr    | String | X | AN(40)      | 이메일 |
| remark      | String | X | AHN(300)    | 비고/메모 |
| bankcd      | String | X | N(3)        | 은행코드 (3자리) |
| accountno   | String | X | AN(300)     | 계좌번호 |
| depositer   | String | X | AHN(30)     | 예금주명 |
| depjuminno  | String | X | AN(20)      | 생년월일/사업자등록번호 |
| payday      | String | X | N(2)        | 출금일 (2자리, 예: 5일 → `05`) |
| paymoney    | String | X | N(11)       | 출금액 |
| paystartdt  | String | X | YYYYMMDD(8) | 출금시작일 |

---

## 응답 코드

| HTTP | 상황 | 예시 응답 |
|:----:|------|-----------|
| 200 | 정상 등록 | (아래 JSON) |
| 400 | 필수 파라미터 누락 / 포맷 오류 | `{"error":"INVALID_PARAM","field":"bankcd"}` |
| 401 | orgcode·uuid 불일치 | `{"error":"UNAUTHORIZED"}` |
| 403 | 미등록 IP 접근 | `{"error":"IP_NOT_ALLOWED"}` |
| 409 | 이미 등록된 납부자 | `{"error":"DUPLICATE_CUST"}` |
| 429 | 요청 한도 초과 | `{"error":"RATE_LIMITED"}` |

> 응답으로 반환되는 `accountno`, `depjuminno` 등 민감정보는 마스킹되어 반환됩니다.
> (요청 시 정상 처리 여부는 HTTP 코드 `200` 으로 확인하세요.)

---

## 예제 (PHP)

```php
<?php
  $curl = curl_init();

  curl_setopt_array($curl, array(
    CURLOPT_URL => "https://주소/api/lcustins/",
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_ENCODING => "",
    CURLOPT_MAXREDIRS => 10,
    CURLOPT_TIMEOUT => 0,
    CURLOPT_FOLLOWLOCATION => true,
    CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
    CURLOPT_CUSTOMREQUEST => "POST",
    CURLOPT_HTTPHEADER => array(
      "orgcode: 귀사의 이용기관 식별번호",
      "uuid: 귀사의 KEY",
      "Content-Type: application/x-www-form-urlencoded"  // 고정값
    ),
    CURLOPT_POSTFIELDS => http_build_query(array(
      "orgcode"    => "귀사의 이용기관 식별번호",
      "custcd"     => "",              // 자동채번 이용기관은 공란 (대부분 공란)
      "custnm"     => "홍길동",
      "telno1"     => "02-1111-1111",
      "hphoneno"   => "010-1234-1234",
      "zipcode"    => "",
      "address1"   => "대한민국",
      "address2"   => "좋은집 1층",
      "mailaddr"   => "email@email.com",
      "remark"     => "비고사항",
      "bankcd"     => "001",
      "accountno"  => "1-1111-1111-1",
      "depositer"  => "홍길동",
      "depjuminno" => "111111",
      "payday"     => "05",
      "paymoney"   => "10000",
      "paystartdt" => "20240101"
    )),
  ));

  $response = curl_exec($curl);
  echo $response;
  echo curl_getinfo($curl, CURLINFO_HTTP_CODE); // 200 확인
?>
```

---

## 응답 명세 (JSON)

> 등록 요청한 정보를 반환합니다. 민감정보(`accountno`, `depjuminno`)는 마스킹 처리됩니다.

| 필드 | 설명 |
|------|------|
| orgcode | 귀사의 이용기관 식별번호 |
| custcd | 납부자번호 (자동채번 시 서버 발급값) |
| custnm | 납부자명 |
| telno1 | 전화번호 |
| hphoneno | 휴대번호 |
| zipcode | 우편번호 |
| address1 | 주소 |
| address2 | 상세주소 |
| mailaddr | 이메일 |
| remark | 비고/메모 |
| bankcd | 은행코드 |
| accountno | 계좌번호 (마스킹) |
| depositer | 예금주명 |
| depjuminno | 생년월일/사업자등록번호 (마스킹) |
| payday | 출금일 |
| paymoney | 출금액 |
| paystartdt | 출금시작일 |

```json
[
  [
    {
      "orgcode": "1111111111",
      "custcd": null,
      "custnm": "홍길동",
      "telno1": "02-1111-1111",
      "hphoneno": "010-1234-1234",
      "zipcode": null,
      "address1": "대한민국",
      "address2": "좋은집 1층",
      "mailaddr": "email@email.com",
      "remark": "비고사항",
      "bankcd": "001",
      "accountno": "*********1-1",
      "depositer": "홍길동",
      "depjuminno": "11****",
      "payday": "05",
      "paymoney": "10000",
      "paystartdt": "20240101"
    }
  ]
]
```
