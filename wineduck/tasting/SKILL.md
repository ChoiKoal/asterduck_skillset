# WineDuck Tasting — 테이스팅 노트 관리

WineDuck 플랫폼에서 와인 테이스팅 노트를 대화형으로 등록하고 조회하는 스킬.
인증 필요 (asterduck-auth 스킬과 함께 사용).

## Trigger

다음 키워드에 반응: "테이스팅", "테이스팅 노트", "시음", "와인 기록", "와인 평가", "마셨는데", "맛있었", "wineduck tasting", "tasting note"

## API Base URL

```
https://coffeeduckbe-production.up.railway.app/api/wineduck
```

## 인증

모든 쓰기 API는 로그인 토큰이 필요. asterduck-auth 스킬로 먼저 로그인.

```
Authorization: Bearer {토큰}
```

## 기능

### 1. 테이스팅 노트 등록

#### 입력 방식 A: 구조화된 포맷

사용자가 항목별로 정리해서 제공:

```
와인: Gevrey-Chambertin 2023
날짜: 2026-04-13
당도: 1
산도: 4
바디: 3
타닌: 3
여운: 4
향강도: 4
총점: 4.0
한줄평: 체리와 오크가 조화로운 엘레강스한 피노
재구매: ㅇ
페어링: 양갈비
향: 체리, 오크, 바닐라, 흙
색상: ruby
음용적기: 1-3y
```

#### 입력 방식 B: 자연어

사용자가 자유롭게 말하면 AI가 파싱:

> "어제 쟝테팡시오 Gevrey-Chambertin 마셨는데, 산미 강하고 타닌 부드럽고 바디 미디엄이야. 체리랑 오크 향 나고. 4점 줄게. 재구매 할 듯. 스테이크랑 먹었어"

#### 자연어 파싱 가이드

| 표현 | 필드 | 값 |
|------|------|----|
| "산미 강하다/높다" | acidity | 4~5 |
| "산미 적당하다" | acidity | 3 |
| "산미 약하다/낮다" | acidity | 1~2 |
| "타닌 강하다/세다" | tannin | 4~5 |
| "타닌 부드럽다/실키하다" | tannin | 2~3 |
| "바디 풀" | body | 5 |
| "바디 미디엄" | body | 3 |
| "바디 라이트" | body | 1~2 |
| "여운 길다" | finish | 4~5 |
| "여운 짧다" | finish | 1~2 |
| "달다/달콤하다" | sweetness | 3~4 |
| "드라이하다" | sweetness | 1 |
| "N점" | rating | N |
| "재구매 ㅇ/할듯/살듯" | repurchase | true |
| "재구매 ㄴ/안살듯" | repurchase | false |

### 등록 프로세스

1. **와인 특정** — wineduck-search로 와인 검색하여 wine_id 확보
2. **데이터 파싱** — 구조화 or 자연어에서 테이스팅 데이터 추출
3. **확인** — "이렇게 등록할게, 맞아?" 파싱 결과를 사용자에게 보여주고 확인
4. **API 호출** — 확인 후 등록
5. **결과 보고** — 등록 성공/실패 알림

### 등록 API

```bash
curl -s -X POST https://coffeeduckbe-production.up.railway.app/api/wineduck/wines/{wine_id}/tastings \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "wine_type": "red",
    "tasted_at": "2026-04-13",
    "sweetness": 1,
    "acidity": 4,
    "body": 3,
    "tannin": 3,
    "finish": 4,
    "aroma_intensity": 4,
    "rating": 4.0,
    "one_liner": "체리와 오크가 조화로운 엘레강스한 피노",
    "repurchase": true,
    "pairing_food": "양갈비",
    "color": "ruby",
    "drink_window": "within_1_5y",
    "aromas": [
      {"source": "custom", "custom_label": "체리"},
      {"source": "custom", "custom_label": "오크"},
      {"source": "custom", "custom_label": "바닐라"}
    ]
  }'
```

**응답:**
```json
{
  "success": true,
  "message": "테이스팅 기록이 저장되었습니다.",
  "tasting_id": 15
}
```

### 2. 테이스팅 노트 조회

#### 특정 와인의 테이스팅 목록
```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines/{wine_id}/tastings"
```

#### 단일 테이스팅 상세
```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/tastings/{tasting_id}"
```

#### 내 테이스팅 목록 (인증 필요)
```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/users/{user_id}/tastings" \
  -H "Authorization: Bearer $TOKEN"
```

### 3. 테이스팅 노트 수정 (인증 필요, 본인만)

```bash
curl -s -X PUT "https://coffeeduckbe-production.up.railway.app/api/wineduck/tastings/{tasting_id}" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"rating": 4.5, "one_liner": "수정된 한줄평"}'
```

### 4. 테이스팅 노트 삭제 (인증 필요, 본인만)

```bash
curl -s -X DELETE "https://coffeeduckbe-production.up.railway.app/api/wineduck/tastings/{tasting_id}" \
  -H "Authorization: Bearer $TOKEN"
```

## 필드 레퍼런스

| 필드 | API key | 범위 | 필수 |
|------|---------|------|------|
| 와인 타입 | wine_type | red / rose / white_sparkling | ✅ |
| 시음 날짜 | tasted_at | YYYY-MM-DD (기본: 오늘) | ❌ |
| 당도 | sweetness | 1-5 | ❌ |
| 산도 | acidity | 1-5 | ❌ |
| 바디 | body | 1-5 | ❌ |
| 타닌 | tannin | 1-5 | ❌ |
| 여운 | finish | 1-5 | ❌ |
| 향 강도 | aroma_intensity | 1-6 | ❌ |
| 총점 | rating | 0-5 (소수점 가능) | ❌ |
| 한줄평 | one_liner | 텍스트 | ❌ |
| 재구매 의향 | repurchase | true / false | ❌ |
| 페어링 음식 | pairing_food | 텍스트 | ❌ |
| 색상 | color | 텍스트 (ruby, garnet, purple 등) | ❌ |
| 음용 적기 | drink_window | now / within_1_5y / over_5y | ❌ |
| 가격 | price_amount | 숫자 | ❌ |
| 통화 | currency_code | KRW / USD / EUR (기본: KRW) | ❌ |
| 향 | aromas | 배열 | ❌ |

## 향 (Aromas) 등록 형식

```json
// 커스텀 입력 — 사용자가 직접 표현한 향
{"source": "custom", "custom_label": "체리"}

// 택소노미 선택 — WineDuck 아로마 DB에서 선택
{"source": "taxonomy", "node_id": 123}
```

## 주의사항

- **사용자가 말한 정보만 등록** — 추측하여 값을 채우지 말 것
- 빠진 필드는 null (API에서 선택사항으로 처리)
- wine_type은 와인 메타데이터에서 자동 추론 가능
- **등록 전 반드시 파싱 결과 확인** — 잘못된 값 방지
