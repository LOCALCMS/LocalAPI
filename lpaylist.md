# lpaylist 요청 명세 (납부내역 조회 / KEY 방식)

> 🚧 **본 문서는 예시(샘플)입니다.**
> 현재 실제 **마이그레이션·운영 적용 전** 단계이며, 엔드포인트 경로·파라미터·정책(토큰 유효시간, 호출 한도 등)은 **확정되지 않았습니다.**
> 실제 연동 전 반드시 담당자 확인이 필요합니다.


납부자의 납부내역을 조회하는 API입니다.

> ⚠️ 반드시 **HTTPS** 로만 호출하세요. uuid(KEY)가 평문으로 노출되지 않도록 주의합니다.

## 참고

### 필수여부
- `O` : 필수 / `X` : 필수아님 / `OX` : 조건에 따라 달라짐

### 포맷
- `A` : 영문 / `N` : 숫자 / `H` : 한글 / `(N)` : 자리수
- `YYYY` : 네자리 년도 / `MM` : 두자리 월 / `DD` : 두자리 일

---

## HEADER (요청 파라미터)

| Parameter   | Type   | 필수여부 | 포맷(Byte)   | 설명 |
|-------------|--------|:--------:|:------------:|------|
| orgcode     | String | O  | N(10)       | 귀사의 이용기관 식별번호 |
| uuid        | String | O  | AN(64)      | 귀사의 KEY |
| stdt        | String | O  | YYYYMMDD(8) | 조회 시작일 / 오늘 기준 직전년도 01월 01일부터 조회 가능 |
| enddt       | String | O  | YYYYMMDD(8) | 조회 종료일 |
| custcd      | String | OX | AN(20)      | 조회대상 납부자번호 / `value_type=1` 일 때 필수 |
| value1      | String | OX | N(11)       | 조회대상 키값1 (현재 휴대폰번호만 제공) / `value_type=2` 일 때 필수 |
| value_type  | String | O  | N(1)        | 조회대상 구분자 (아래 표 참고) |
| sum_type    | String | O  | N(1)        | 조회종류 / `0` 개별납부내역, `1` 합산납부내역 |
| seltype     | String | O  | N(1)        | 조회기준일 / `0` 실출금일, `1` 청구일 |

### value_type × 필수 파라미터

| value_type | 필수 파라미터 | 의미 |
|:----------:|---------------|------|
| 0 | (없음) | 전체 납부자 조회 |
| 1 | custcd | 납부자번호 단건 조회 |
| 2 | value1 | 키값1(휴대폰번호) 조회 / 동일번호 다건 가능 |

---

## 응답 코드

| HTTP | 상황 | 예시 응답 |
|:----:|------|-----------|
| 200 | 정상 | (아래 JSON) |
| 400 | 필수 파라미터 누락 / 포맷 오류 | `{"error":"INVALID_PARAM","field":"stdt"}` |
| 401 | orgcode·uuid 불일치 | `{"error":"UNAUTHORIZED"}` |
| 403 | 미등록 IP 접근 | `{"error":"IP_NOT_ALLOWED"}` |
| 429 | 요청 한도 초과 | `{"error":"RATE_LIMITED"}` |

> 조회 결과는 인증된 orgcode 소속 납부자로 한정됩니다. (타 기관 납부자 조회 불가)

---

## 예제 1 — 납부자번호로 개별 납부내역 조회

```php
<?php
  $curl = curl_init();

  curl_setopt_array($curl, array(
    CURLOPT_URL => "https://주소/api/lpaylist/",
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_ENCODING => "",
    CURLOPT_MAXREDIRS => 10,
    CURLOPT_TIMEOUT => 0,
    CURLOPT_FOLLOWLOCATION => false,
    CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
    CURLOPT_CUSTOMREQUEST => "GET",
    CURLOPT_HTTPHEADER => array(
      "orgcode: 귀사의 이용기관 식별번호",
      "uuid: 귀사의 KEY",
      "stdt: 20240101",
      "enddt: 20241231",
      "custcd: 납부자번호",
      "value1: ",
      "value_type: 1",   // 납부자번호로
      "sum_type: 0",     // 개별납부내역을
      "seltype: 0"       // 실출금일 기준으로
    ),
  ));
  $response = curl_exec($curl);
  echo $response;
?>
```

### 응답 명세 (개별, sum_type=0)

> 현재 미납내역은 제공하지 않으므로 (★) `PaymentAmt` 또는 (★) `PaidAmt` 중 하나만 참고하세요.
> 계좌번호(`Acc`)는 비식별(마스킹) 처리되어 반환됩니다.

| 필드 | 설명 |
|------|------|
| OrgCode | 귀사의 이용기관 식별번호 |
| CustCd | 납부자번호 |
| PaymentDt | 청구년월일 |
| PaymentSeq | 납부회차 |
| PaymentAmt | 청구금액 (★) |
| PaidDt | 실출금일 |
| PaidAmt | 실출금액 (★) |
| Status | 상태값 |
| remark | 비고 |
| Bnm | 은행명 |
| Acc | 계좌번호(비식별 처리) |

```json
[
  {
    "OrgCode": "1111111111",
    "CustCd": "1020304050",
    "PaymentDt": "20240105",
    "PaymentSeq": "1",
    "PaymentAmt": 10000,
    "PaidDt": "20240105",
    "PaidAmt": 10000,
    "Status": "0",
    "remark": "None",
    "Bnm": "은행명",
    "Acc": "*********1234"
  },
  {
    "OrgCode": "1111111111",
    "CustCd": "1020304050",
    "PaymentDt": "20240205",
    "PaymentSeq": "2",
    "PaymentAmt": 10000,
    "PaidDt": "20240210",
    "PaidAmt": 10000,
    "Status": "0",
    "remark": "None",
    "Bnm": "은행명",
    "Acc": "*********1234"
  }
]
```

---

## 예제 2 — 키값1(휴대폰번호)로 합산 납부내역 조회

> 참고: 키값1은 휴대폰번호입니다. 동일 번호를 쓰는 납부자가 2명인 경우를 가정합니다.

```php
<?php
  $curl = curl_init();

  curl_setopt_array($curl, array(
    CURLOPT_URL => "https://주소/api/lpaylist/",
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_ENCODING => "",
    CURLOPT_MAXREDIRS => 10,
    CURLOPT_TIMEOUT => 0,
    CURLOPT_FOLLOWLOCATION => false,
    CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
    CURLOPT_CUSTOMREQUEST => "GET",
    CURLOPT_HTTPHEADER => array(
      "orgcode: 귀사의 이용기관 식별번호",
      "uuid: 귀사의 KEY",
      "stdt: 20240101",
      "enddt: 20241231",
      "custcd: ",
      "value1: 휴대폰번호",
      "value_type: 2",   // 키값1(value1)로
      "sum_type: 1",     // 합산납부내역을
      "seltype: 0"       // 실출금일 기준으로
    ),
  ));
  $response = curl_exec($curl);
  echo $response;
?>
```

### 응답 명세 (합산, sum_type=1)

> 현재 미납내역은 제공하지 않으므로 (★) `paymentamt_sum` 또는 (★) `paidamt_sum` 중 하나만 참고하세요.

| 필드 | 설명 |
|------|------|
| orgcode | 귀사의 이용기관 식별번호 |
| custcd | 납부자번호 |
| custnm | 납부자명 |
| paymentamt_sum | 조회기간 총 청구금액 (★) |
| paidamt_sum | 조회기간 총 출금금액 (★) |
| data_count | 총 납부회차 |

```json
[
  {
    "orgcode": "1111111111",
    "custcd": "A000000001",
    "custnm": "홍길동",
    "paymentamt_sum": "60000",
    "paidamt_sum": "60000",
    "data_count": "3"
  },
  {
    "orgcode": "1111111111",
    "custcd": "B000000001",
    "custnm": "홍길동",
    "paymentamt_sum": "100000",
    "paidamt_sum": "100000",
    "data_count": "2"
  }
]
```
