# LOCALCMS API 간편 가이드 (Quick Start)

> 🚧 **본 문서는 예시(샘플)입니다.**
> 현재 실제 **마이그레이션·운영 적용 전** 단계이며, 엔드포인트 경로·파라미터·정책(토큰 유효시간, 호출 한도 등)은 **확정되지 않았습니다.**
> 실제 연동 전 반드시 담당자 확인이 필요합니다.

가장 빠르게 "납부내역 조회"까지 해보는 3단계 안내입니다. 상세 명세는 각 문서를 참고하세요.

---

## 0. 준비물

| 항목 | 설명 |
|------|------|
| 이용기관 식별번호 (orgcode) | 담당자 발급 |
| KEY (uuid) | 담당자 발급 / 외부 노출 금지 |
| 고정 공인 IP | 호출 서버 IP를 화이트리스트에 등록 (담당자 요청) |
| HTTPS 환경 | 모든 호출은 `https://` 만 가능 |

> 인증은 **KEY 방식**(orgcode + uuid)과 **TOKEN 방식**(`authorization: Bearer {TOKEN}`) 두 가지가 있습니다.
> 아래는 가장 간단한 **KEY 방식** 기준입니다. (보안상 운영에서는 TOKEN 방식 권장)

---

## 1. 납부내역 조회 (가장 흔한 케이스)

납부자번호로 개별 납부내역을 실출금일 기준으로 조회하는 예시입니다.

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

가장 많이 쓰는 조합:

| 파라미터 | 값 | 의미 |
|----------|----|------|
| value_type | `1` | 납부자번호(custcd)로 조회 |
| sum_type | `0` | 개별 납부내역 |
| seltype | `0` | 실출금일 기준 |

---

## 2. 응답 확인

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
  }
]
```

- HTTP `200` 이면 정상, JSON 배열이 반환됩니다.
- 청구금액(`PaymentAmt`)과 실출금액(`PaidAmt`) 중 **하나만** 사용하세요. (미납내역 미제공)
- 계좌번호·주민(사업자)번호는 **마스킹**되어 옵니다. (정상 동작)

---

## 3. (선택) 토큰 방식으로 바꾸기

운영에서는 토큰 방식을 권장합니다. 먼저 토큰을 발급받고,

```bash
curl -X POST "https://주소/auth/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "orgcode=귀사의식별번호&uuid=귀사의KEY"
```

조회 시 `orgcode`·`uuid` 헤더 대신 토큰 헤더를 사용합니다. (URL도 `tpaylist`)

```bash
curl -X GET "https://주소/api/tpaylist/" \
  -H "authorization: Bearer {발급받은_TOKEN}" \
  -H "stdt: 20240101" -H "enddt: 20241231" \
  -H "custcd: 납부자번호" -H "value1: " \
  -H "value_type: 1" -H "sum_type: 0" -H "seltype: 0"
```

---

## 자주 막히는 곳

| 증상 | 원인 / 조치 |
|------|-------------|
| `403` | 미등록 IP → 호출 서버 IP 화이트리스트 등록 확인 |
| `401` | orgcode·uuid 오류 또는 토큰 만료 → 인증 정보 확인 / 토큰 재발급 |
| `429` | 호출 한도 초과 → 간격을 두고 재시도 |
| 빈 배열 `[]` | 조건에 맞는 데이터 없음 → 기간·파라미터 확인 |
| 한글 깨짐 | UTF-8 인코딩 설정 확인 |

---

## 다음 단계

| 하고 싶은 것 | 참고 문서 |
|--------------|-----------|
| 납부내역 조회 상세 (합산/휴대폰번호 조회 등) | [lpaylist.md](./lpaylist.md) |
| 납부자 등록 | [lcustint.md](./lcustint.md) |
| 언어별 호출 예제 (PHP/Python/Node/Java/C#) | [EXAMPLES.md](./EXAMPLES.md) |
| 자주 묻는 질문 | [FAQ.md](./FAQ.md) |
