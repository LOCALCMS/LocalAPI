# LOCALCMS API

LOCALCMS(로컬 씨엠에스)의 출금 결과 및 납부자 관리를 위한 API 문서입니다.

> ⚠️ 모든 통신은 반드시 **HTTPS** 로만 이루어집니다. API KEY(uuid)가 평문(HTTP)으로 전송되지 않도록 주의하세요.

---

## 인증 방식

본 API는 두 가지 인증 방식을 제공합니다.

| 방식 | 설명 | 권장 |
|------|------|:----:|
| 토큰 방식 (Access Token) | `/auth/token` 으로 토큰 발급 후, `authorization: Bearer {TOKEN}` 헤더로 호출 | ✅ 권장 |
| 키 방식 (API KEY) | 발급받은 `uuid`(KEY)를 매 요청 헤더에 포함 | 보조 |

- **등록된 IP에서만** 접속이 가능합니다. (IP 화이트리스트)
- API KEY(uuid)는 만료가 없는 영구 키이므로, **유출이 의심되면 즉시 재발급** 요청하세요.
- 가능하면 만료가 있는 토큰 방식 사용을 권장합니다.

---

## 엔드포인트 목록

| 기능 | 메서드 | 토큰 방식 | 키 방식 |
|------|:------:|-----------|---------|
| 납부내역 조회 | GET | `/api/tpaylist/` | `/api/kpaylist/` |
| 납부자 등록 | POST | `/api/tcustins/` | `/api/kcustins/` |

> 처리 결과는 인증 방식과 무관하게 동일합니다.

---

## 공통 응답 코드

| HTTP | 상황 | 예시 응답 |
|:----:|------|-----------|
| 200 | 정상 처리 | (각 API의 정상 응답 JSON) |
| 400 | 필수 파라미터 누락 / 포맷 오류 | `{"error":"INVALID_PARAM","field":"payday"}` |
| 401 | orgcode·uuid(토큰) 불일치 | `{"error":"UNAUTHORIZED"}` |
| 403 | 미등록 IP 접근 | `{"error":"IP_NOT_ALLOWED"}` |
| 429 | 요청 한도 초과 | `{"error":"RATE_LIMITED"}` |
| 500 | 서버 내부 오류 | `{"error":"INTERNAL_ERROR"}` |

---

## 토큰 발급 예시

```php
<?php
  $curl = curl_init();

  curl_setopt_array($curl, array(
    CURLOPT_URL => 'https://주소/auth/token',   // ★ 반드시 https
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_ENCODING => '',
    CURLOPT_MAXREDIRS => 10,
    CURLOPT_TIMEOUT => 0,
    CURLOPT_FOLLOWLOCATION => true,
    CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
    CURLOPT_CUSTOMREQUEST => 'POST',
    CURLOPT_POSTFIELDS => 'orgcode=orgcode입력&uuid=uuid입력',
    CURLOPT_HTTPHEADER => array(
      'Content-Type: application/x-www-form-urlencoded'
    ),
  ));

  $response = curl_exec($curl);
  curl_close($curl);
  echo $response;
?>
```

---

## 납부내역 조회 / KEY 방식 예시

```php
<?php
  $curl = curl_init();

  curl_setopt_array($curl, array(
    CURLOPT_URL => 'https://주소/api/kpaylist/',
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_ENCODING => '',
    CURLOPT_MAXREDIRS => 10,
    CURLOPT_TIMEOUT => 0,
    CURLOPT_FOLLOWLOCATION => true,
    CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
    CURLOPT_CUSTOMREQUEST => 'GET',
    CURLOPT_HTTPHEADER => array(
      'orgcode: 귀사의 이용기관 식별번호',
      'uuid: 귀사의 KEY',
      'stdt: 조회시작일',
      'enddt: 조회종료일',
      'custcd: 조회대상 납부자번호',
      'value1: 조회대상 키값1',
      'value_type: 조회대상 구분자',
      'sum_type: 조회종류',
      'seltype: 조회기준일'
    ),
  ));

  $response = curl_exec($curl);
  curl_close($curl);
  echo $response;
?>
```

### 처리 결과 예시

```json
{
  "OrgCode": "귀사의 이용기관 식별번호",
  "CustCd": "납부자번호",
  "PaymentDt": "20240105",
  "PaymentSeq": "1",
  "PaymentAmt": 10000,
  "PaidDt": "20240105",
  "PaidAmt": 10000,
  "RegDt": "2024-01-19 10:19:42"
}
```

> 검색조건(seltype): `0` 실출금일 / `1` 청구일
> 예) 15일이 일요일인 경우 16일 월요일에 출금 → 청구일 15일, 실출금일 16일

---

## 상세 명세

1. [납부내역조회 KEY 방식 요청 명세 및 샘플](https://github.com/LOCALCMS/LOCALAPI/blob/main/lpaylist.md)
2. [납부자등록 KEY 방식 요청 명세 및 샘플](https://github.com/LOCALCMS/LOCALAPI/blob/main/lcustint.md)

---

## 보안 유의사항

- 모든 호출은 **HTTPS** 로만 가능하며, HTTP 요청은 거부됩니다.
- 응답 내 **계좌번호·생년월일/사업자등록번호** 등 민감정보는 마스킹되어 반환됩니다.
- 조회 결과는 **인증된 orgcode 소속 납부자**로 한정됩니다. (타 기관 납부자 조회 불가)
- API KEY(uuid)는 절대 클라이언트(브라우저·앱)에 노출하지 마시고, 서버 간 통신에만 사용하세요.
