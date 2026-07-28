> ## ⚠️ 사용 전 필독 — 반드시 "원격 최신" 버전으로 동작하라
> 이 사본은 구버전일 수 있다. **파일에 적힌 버전 숫자를 신뢰하지 말고**, 사용 전 원격 main과 직접 대조해 최신을 적용하라:
> 1. 쓰기 가능한 위치에 최신 사본 확보:
> `git -C /tmp/asterduck_skillset pull -q 2>/dev/null || git clone -q https://<GITHUB_TOKEN>@github.com/ChoiKoal/asterduck_skillset.git /tmp/asterduck_skillset`
> 2. 배포 사본과 원격 버전 대조: `cat VERSION`(배포) vs `cat /tmp/asterduck_skillset/VERSION`(원격 최신)
> 3. **다르면 = 배포본 구버전** → `/tmp/asterduck_skillset/`의 SKILL.md·CHANGELOG·OpenAPI를 **우선**해 동작. (상세: `AGENT_UPDATE.md`)
> 구버전 관성/캐시 지식 금지. `<GITHUB_TOKEN>`은 그룹 CLAUDE.md 또는 배포 사본 git remote에 있음.
> **🔴 호환성(전체는 CHANGELOG)**: 와인 카탈로그의 연도 음용적기(`drink_*_year`)와 셀러 엔트리의 날짜(`drink_from`/`drink_until`)는 모두 현행 API에서 지원된다.

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

### 시음 등록 전 와인이 카탈로그에 없어 새로 등록하는 경우

시음할 와인이 검색에 안 잡히면 **wineduck-wine 스킬로 먼저 등록**하고, 이때 **`appellation_id`/`region_id`/`country_id`를 반드시 채울 것**. 정복 도감(conquest)은 와인의 아펠라시옹 연결을 기준으로 집계하므로, 미연결로 등록하면 도감에 안 뜬다.

#### 마신 와인 빠른 등록 (quick-tasting)

`POST /api/wineduck/quick-tasting`으로 와인 검색·등록·시음을 한 번에 처리할 때, 신규 와인이 생성되면:

- (a) BE가 **와인명에서 아펠라시옹을 자동매칭**한다 — 단 **라틴 아펠라시옹명이 와인명에 그대로 들어있을 때만**(예: `Chablis`, `Gevrey-Chambertin`, `Barolo`).
- (b) **도멘/퀴베명만 있는 와인은 자동매칭이 안 된다** → `appellation_id`/`region_id`/`country_id` 파라미터를 **명시적으로 함께 보내는 것을 권장**.

파라미터 계약: 셋 중 일부만 보내도 됨 — 아펠라시옹을 주면 지역/국가가 자동 보강되고, 지역을 주면 국가가 자동 보강된다. 유효하지 않은 ID면 400.

ID 매핑 방법(아펠라시옹 정본 검색 → ID 확인 → 없으면 상위 아펠라시옹으로)은 **wineduck-wine 스킬**의 "지역/아펠라시옹 ID 매핑" 절차를 따른다. 이름을 알면 `GET /wineduck/appellations/search?q=<이름>`으로 `appellation_id`를 먼저 resolve한 뒤 파라미터로 넘긴다. **일치하는 후보가 없거나 모호하면 추측하지 말고 비워둔다** — 미연결이 오연결보다 낫다.

##### 와인 매칭·빈티지 규칙

`wine_id` 없이 보내면 BE가 **정규화된 `(canonical_name, wine_type, vintage_year)` 3필드로 기존 와인을 매칭**한다:

- **같은 이름이라도 빈티지가 다르면 별도 와인 row** — 샤블리 2021과 2022는 서로 다른 와인으로 생성·집계된다. 정복 도감의 빈티지 축이 이 단위로 쌓이므로, 시음마다 빈티지를 정확히 보낼 것.
- **NV(논빈티지)**: `vintage_year` 생략 또는 `null` = NV. NV는 NV끼리만 매칭되고, NV ↔ 연도 와인은 별개 row다.
- ⚠️ **`wine_id`와 함께 쓸 때는 생략과 명시적 `null`이 다르다** — 필드를 **생략**하면 카탈로그 정본을 쓰지만, `vintage_year: null`을 **명시**하면 "NV다"라는 주장으로 취급돼 카탈로그에 연도가 있으면 409가 난다. 정본을 쓰려면 필드 자체를 빼라.
- **빈티지 검증**: 빈 문자열은 NV로 정규화된다. `0`·`1899` 등 허용 범위(1900~현재+1) 밖 정수는 **400** (DB 접근 전 거부).
- 같은 이름·같은 빈티지라도 `wine_type`이 다르면 별도 row.

##### wine_id 지정 시 — 카탈로그가 정본 (409 주의)

`wine_id`를 지정하면 name/type/vintage는 **카탈로그 값이 정본**이다. 요청에 함께 실은 `wine_name`/`wine_type`/`vintage_year`가 카탈로그와 다르면 저장되지 않고 409가 반환된다:

```json
{
  "success": false,
  "code": "WINE_ID_FIELD_MISMATCH",
  "conflict_fields": ["vintage_year"],
  "canonical_values": { "wine_name": "Chablis ...", "wine_type": "white", "vintage_year": 2021 }
}
```

`conflict_fields`는 불일치한 필드만, `canonical_values`는 **카탈로그 정본 3필드 전체**(name/type/vintage)를 담는다.

**대응 규칙:**
1. `wine_id`를 쓸 때는 name/type/vintage를 **같이 보내지 않는 것**이 가장 안전하다 — `wine_id` 단독 전송이 허용되며 카탈로그 정본이 그대로 쓰인다.
2. 409를 받으면 `canonical_values`(카탈로그 정본)를 사용자에게 보여주고 확인할 것 — (a) 카탈로그 값대로 기록하려면 `wine_id`만 남기고 재요청, (b) 실제로 다른 빈티지/타입을 마신 거면 `wine_id`를 **빼고** 이름+빈티지로 보내 새 row를 만든다. 확인 없이 임의 재시도 금지.

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
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/users/{user_id}/tastings?limit=10&offset=0&wine_type=white&sort=newest" \
  -H "Authorization: Bearer $TOKEN"
```
`limit`은 기본 10, 1~200이고 `offset`은 0 이상이다. `wine_type` 필터는 정식값 `red`/`white`/`sparkling`/`rose`만 받으며, `white`·`sparkling` 필터는 저장된 레거시 `white_sparkling`도 함께 반환한다. `sort`는 `newest`/`oldest`/`rating_high`; 응답은 `tastings`, `total`, `overall_total`, `limit`, `offset`, `has_more`다.

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
| 와인 타입 | wine_type | red / white / sparkling / rose (레거시 white_sparkling 호환) | ✅ |
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
| 가격 | price_amount | 숫자 | ❌ |
| 통화 | currency_code | KRW / USD / EUR (기본: KRW) | ❌ |
| 향 | aromas | 배열 | ❌ |

> **`wine_type`**: 신규 기록에는 정식값 `red`/`white`/`sparkling`/`rose`를 사용한다. 기존 `white_sparkling`은 레거시 데이터·호환 요청으로 계속 허용된다.

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
