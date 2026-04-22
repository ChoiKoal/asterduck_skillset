# WineDuck Cellar — 와인 셀러 관리

WineDuck 플랫폼에서 내 와인 셀러(보유 와인)를 대화형으로 관리하는 스킬.
보유 목록 조회, 통계 확인, 음용 임박 체크, 사진 기반 셀러 추가, 소비(오픈) 기록까지 지원.
인증 필요 (asterduck-auth 스킬과 함께 사용).

## Trigger

다음 키워드에 반응: "셀러", "와인 셀러", "내 와인", "보유 와인", "셀러 추가", "셀러에 넣어", "셀러 등록", "음용 적기", "와인 오픈", "와인 마셨어", "다 마셨어", "wine cellar", "my cellar"

## API Base URL

```
https://coffeeduckbe-production.up.railway.app/api
```

> 다른 WineDuck 스킬과 달리 `/api/wineduck` 가 아니라 `/api/cellar` 입니다.

## 인증

셀러 API는 전부 로그인 토큰 필요. asterduck-auth 스킬로 먼저 로그인.

```
Authorization: Bearer {토큰}
```

## 기능

### 1. 셀러 목록 조회

보유 중인 와인 리스트. 와인 정보(이름/생산자/빈티지/품종/국가/지역/아펠라시옹)가 JOIN되어 풍부하게 반환.

```bash
# 기본: 현재 보유(in_cellar) 와인, 최신순, 20개
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://coffeeduckbe-production.up.railway.app/api/cellar"

# 레드 와인만 / 음용 임박순 / 2페이지
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://coffeeduckbe-production.up.railway.app/api/cellar?wine_type=red&sort=drink_soon&page=2"

# "Barolo" 검색 (이름 또는 생산자 부분 매칭)
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://coffeeduckbe-production.up.railway.app/api/cellar?q=Barolo"

# 이미 마신 와인만
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://coffeeduckbe-production.up.railway.app/api/cellar?status=consumed"
```

#### Query Params

| 파라미터 | 기본값 | 값 |
|----------|--------|-----|
| `status` | `in_cellar` | `in_cellar` / `consumed` / `gifted` / `received` |
| `wine_type` | (전체) | `red` / `rose` / `white_sparkling` |
| `sort` | `newest` | `newest` / `oldest` / `drink_soon` / `purchase_date` |
| `q` | — | 와인명(canonical_name) 또는 생산자(producer) 부분 매칭 |
| `page` | 1 | 페이지 번호 |
| `per_page` | 20 | 최대 100 |

> ⚠️ **`wine_type` 주의**: 화이트 와인과 스파클링 와인은 같은 카테고리 `white_sparkling`로 묶입니다. `white` 단독 값은 사용할 수 없습니다.

#### 응답 구조 (각 entry 객체)

**셀러 엔트리 필드:**

| 필드 | 설명 |
|------|------|
| `id` | 셀러 엔트리 ID |
| `wine_id` | 연결된 와인 ID |
| `quantity` | 보유 수량 (병) |
| `purchase_date` | 구매일 (YYYY-MM-DD, nullable) |
| `purchase_price` | 구매가 (nullable) |
| `currency` | 통화 (`KRW`/`USD`/`EUR`/`JPY` 등) |
| `storage_location` | 보관 위치 (ex: "셀러 A칸 3번", nullable) |
| `drink_from` | 음용 시작 적기 (YYYY-MM-DD, nullable) |
| `drink_until` | 음용 마감 적기 (YYYY-MM-DD, nullable) |
| `status` | `in_cellar` / `consumed` / `gifted` / `received` |
| `consumed_at` | 소비 일시 (consume API 호출 시 자동 기록) |
| `consumed_quantity` | 소비한 수량 |
| `tasting_id` | 연결된 테이스팅 노트 ID (nullable) |
| `note` | 개인 메모 (nullable) |
| `created_at` / `updated_at` | 등록/수정 일시 |

**와인 정보 (JOIN):**

| 필드 | 설명 |
|------|------|
| `canonical_name` | 와인 정식 이름 (ex: "Gevrey-Chambertin Vieilles Vignes") |
| `producer` | 생산자/도멘 (ex: "Domaine Geantet-Pansiot") |
| `wine_type` | `red` / `rose` / `white_sparkling` |
| `vintage_year` | 빈티지 연도 (NV는 null) |
| `grapes_text` | 품종 (쉼표 구분 텍스트) |

**국가/지역/아펠라시옹 (JOIN):**

| 필드 | 설명 |
|------|------|
| `country_name` | 국가 영문명 (ex: "France") |
| `country_name_ko` | 국가 한글명 (ex: "프랑스") |
| `country_iso_code` | ISO 3166-1 alpha-2 (ex: "FR") |
| `region_name` / `region_name_ko` | 지역명 (ex: "Côte de Nuits" / "코트 드 뉘") |
| `appellation_name` / `appellation_name_ko` | 아펠라시옹명 (ex: "Gevrey-Chambertin") |
| `appellation_classification` | 등급 (ex: "AOC" / "Grand Cru" / "1er Cru") |

**응답 예시:**
```json
{
  "success": true,
  "entries": [
    {
      "id": 23,
      "wine_id": 142,
      "quantity": 1,
      "purchase_date": "2026-04-15",
      "purchase_price": 48000,
      "currency": "KRW",
      "storage_location": "거실 셀러 2단",
      "drink_from": "2026-04-01",
      "drink_until": "2029-12-31",
      "status": "in_cellar",
      "note": "생일 기념 구매",
      "canonical_name": "Wachau Riesling Federspiel",
      "producer": "Domäne Wachau",
      "wine_type": "white_sparkling",
      "vintage_year": 2023,
      "grapes_text": "Riesling",
      "country_name": "Austria",
      "country_name_ko": "오스트리아",
      "country_iso_code": "AT",
      "region_name": "Wachau",
      "region_name_ko": "바하우",
      "appellation_name": "Wachau",
      "appellation_name_ko": "바하우",
      "appellation_classification": "DAC"
    }
  ],
  "pagination": { "page": 1, "per_page": 20, "total": 23, "total_pages": 2 }
}
```

### 2. 셀러 통계

총 병 수, 총 가치(보유 와인의 구매가 × 수량 합), 국가별 분포.

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://coffeeduckbe-production.up.railway.app/api/cellar/stats"
```

**응답:**
```json
{
  "success": true,
  "total_bottles_in_cellar": 23,
  "total_value": 987000,
  "country_breakdown": [
    { "country_name": "France", "country_name_ko": "프랑스", "iso_code": "FR", "bottle_count": 11 },
    { "country_name": "Italy", "country_name_ko": "이탈리아", "iso_code": "IT", "bottle_count": 5 }
  ]
}
```

### 3. 음용 임박 목록

이번 달 말 기준, `drink_until`이 도래한 보유 와인.

```bash
# 최대 10병
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://coffeeduckbe-production.up.railway.app/api/cellar/expiring"

# n으로 개수 조정 (1~100)
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://coffeeduckbe-production.up.railway.app/api/cellar/expiring?n=20"
```

응답은 목록과 동일한 와인 JOIN 구조. `drink_until ASC` 정렬.

### 4. 셀러 엔트리 상세

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://coffeeduckbe-production.up.railway.app/api/cellar/23"
```

### 5. 셀러 등록 — 와인 사진으로 바로 추가

**이 스킬의 핵심 플로우.** 사용자가 와인 사진을 주고 "셀러에 추가해줘"라고 하면 아래 순서로 진행.

#### Step 1. 사진에서 와인 정보 추출 (에이전트의 비전 능력 사용)

Claude/GPT-4 등 비전 가능한 에이전트는 **직접** 라벨을 읽어 다음 정보를 파싱:

- 와인 이름
- 생산자 (있으면)
- 빈티지 연도 (있으면, NV면 null)
- 와인 타입 (red / rose / white_sparkling — 라벨 또는 병 색상으로 추론)
- 품종 (표기되어 있으면)
- 국가 / 지역 / 아펠라시옹

> 비전이 없는 환경에서는 BE의 보조 엔드포인트 `POST /api/wineduck/analyze-label` (body: `{ "images": [base64...] }`)를 호출해 GPT-4o로 파싱 가능.

#### Step 2. 중복 확인 — wineduck-search

파싱한 `name + vintage_year`로 기존 와인을 검색.

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines/search?name=Barolo&vintage=2019"
```

**판단:**
- ✅ **이름 + 빈티지 일치** → 그 `wine_id`로 **Step 4로 바로 이동** (와인 재등록 X)
- 🔶 **이름 유사 + 빈티지 다름** → 사용자에게 "같은 와인의 다른 빈티지가 있어. 새 빈티지로 등록할까?" 확인 후 Step 3
- ❌ **검색 결과 없음** → Step 3

#### Step 3. 와인 신규 등록 — wineduck-wine 스킬 위임

wineduck-wine 스킬을 사용해 canonical_name 규칙(Type A/B/C)에 맞춰 등록하고, 응답의 `wine_id`를 확보.

#### Step 4. 셀러 등록 API 호출

```bash
curl -s -X POST "https://coffeeduckbe-production.up.railway.app/api/cellar" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "wine_id": 142,
    "quantity": 1,
    "purchase_date": "2026-04-22",
    "purchase_price": 48000,
    "currency": "KRW",
    "storage_location": "거실 셀러 2단",
    "drink_from": "2026-04-01",
    "drink_until": "2029-12-31",
    "note": "생일 기념 구매"
  }'
```

**응답:**
```json
{
  "success": true,
  "message": "셀러에 와인이 등록되었습니다.",
  "entry_id": 23
}
```

HTTP 201 반환.

#### Body 필드

| 필드 | 필수 | 설명 |
|------|------|------|
| `wine_id` | ✅ | 기존 와인 ID (Step 2~3에서 확보) |
| `quantity` | ❌ | 병 수 (기본 1, 1 이상 정수) |
| `status` | ❌ | 기본 `in_cellar`. `received`는 선물 받은 경우 |
| `purchase_date` | ❌ | YYYY-MM-DD |
| `purchase_price` | ❌ | 숫자 |
| `currency` | ❌ | 기본 `KRW`. `USD`/`EUR`/`JPY` 등 |
| `storage_location` | ❌ | 자유 텍스트 |
| `drink_from` | ❌ | 음용 시작 적기 YYYY-MM-DD |
| `drink_until` | ❌ | 음용 마감 적기 YYYY-MM-DD |
| `note` | ❌ | 메모 |

#### 사용자에게 확인받기

등록 전 반드시 파싱 결과를 보여주고 확인:

> "라벨에서 이렇게 읽었어:
> - Barolo Marenca (Italy / Piedmont / Barolo DOCG)
> - 생산자: Giovanni Rosso
> - 빈티지: 2019
> - 레드 / Nebbiolo
>
> 이 와인을 셀러에 1병 추가할게. 구매일·가격·음용적기도 있으면 알려줘."

### 6. 셀러 엔트리 수정

```bash
curl -s -X PUT "https://coffeeduckbe-production.up.railway.app/api/cellar/23" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "quantity": 2, "storage_location": "거실 셀러 3단" }'
```

수정 가능 필드 (부분 수정, 하나 이상 필수):
`quantity`, `purchase_date`, `purchase_price`, `currency`, `storage_location`,
`drink_from`, `drink_until`, `status`, `note`

> `consumed_at`, `consumed_quantity`, `tasting_id`는 수정 API로 변경 불가 — **소비 API 전용**.

### 7. 소비(오픈) 기록

사용자가 "어제 #23 마셨어" / "다 마셨어" 등을 말하면 consume API로 처리. 수량이 원자적으로 차감됨 (`SELECT FOR UPDATE` 락).

```bash
# 전량 소비
curl -s -X POST "https://coffeeduckbe-production.up.railway.app/api/cellar/23/consume" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "quantity": 1, "status": "consumed" }'

# 2병 보유 중 1병만 소비 (부분 소비 → 수량만 차감, status 유지)
curl -s -X POST "https://coffeeduckbe-production.up.railway.app/api/cellar/23/consume" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "quantity": 1, "status": "consumed" }'

# 테이스팅 노트와 연결하여 소비
curl -s -X POST "https://coffeeduckbe-production.up.railway.app/api/cellar/23/consume" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{ "quantity": 1, "status": "consumed", "tasting_id": 87 }'
```

**Body:**

| 필드 | 필수 | 설명 |
|------|------|------|
| `quantity` | ✅ | 소비할 병 수 (1 이상) |
| `status` | ✅ | `consumed` (마심) 또는 `gifted` (선물함) |
| `tasting_id` | ❌ | 연결할 테이스팅 노트 ID — wineduck-tasting으로 먼저 기록한 뒤 그 ID를 전달 권장 |

**응답:**
```json
{ "success": true, "message": "소비가 기록되었습니다.", "remaining_quantity": 0 }
```

> 전량 소비(`remaining_quantity == 0`)일 때만 `status` / `consumed_at` / `consumed_quantity` / `tasting_id` 필드가 기록됨. 부분 소비는 수량만 차감.

### 8. 소프트 삭제

```bash
curl -s -X DELETE "https://coffeeduckbe-production.up.railway.app/api/cellar/23" \
  -H "Authorization: Bearer $TOKEN"
```

`is_active = 0` 처리 (복구 가능하지만 현재 복구 API 없음). 잘못 등록했을 때만 사용하고, 다 마신 경우는 **삭제가 아니라 consume**을 써야 셀러 히스토리가 보존됨.

## 대화 예시

### 사진 → 셀러 추가

> 사용자: (라벨 사진 첨부) "이거 셀러에 추가해줘. 어제 8만원에 샀어"

에이전트 처리:
1. 비전으로 라벨 파싱 → `Gevrey-Chambertin 2022, Domaine Geantet-Pansiot`
2. `wineduck-search?name=Gevrey-Chambertin&vintage=2022` 호출 → 결과 없음
3. wineduck-wine 스킬 → 등록 → `wine_id: 155`
4. `POST /api/cellar` `{ wine_id: 155, quantity: 1, purchase_date: "2026-04-21", purchase_price: 80000 }`
5. 사용자에게 entry_id + 요약 회신

### 보유 와인 리스트

> 사용자: "내 셀러 보여줘. 레드만."

에이전트:
```
GET /api/cellar?wine_type=red&per_page=50
```

→ 각 와인의 `canonical_name` · `vintage_year` · `producer` · `appellation_name_ko` · `quantity` · `drink_until` 요약 테이블로 출력.

### 음용 임박 체크

> 사용자: "이번 달 안에 마셔야 할 와인 뭐 있어?"

```
GET /api/cellar/expiring?n=20
```

→ `drink_until` 기준 오름차순 리스트 반환.

### 소비 기록

> 사용자: "어제 #23 Wachau Riesling 다 마셨어. 되게 좋았어. 테이스팅 노트도 남길래."

에이전트:
1. wineduck-tasting 스킬로 테이스팅 노트 등록 → `tasting_id: 87`
2. `POST /api/cellar/23/consume` `{ quantity: 1, status: "consumed", tasting_id: 87 }`

## 주의사항

- **삭제 ≠ 소비**: 다 마신 와인은 **consume API**로 기록해야 "얼마나 마셨는지" 히스토리가 남음. DELETE는 잘못 등록한 경우에만.
- **wine_id 없이는 등록 불가**: 셀러 API는 와인 자체를 등록하지 않음. 반드시 wineduck-search → (없으면) wineduck-wine 순으로 wine_id 확보.
- **sort=drink_soon은 `drink_until`이 null인 엔트리를 뒤로 밀지 않음** — MySQL의 NULL 정렬은 오름차순에서 제일 앞. 음용 임박만 보고 싶으면 `/cellar/expiring`을 쓰는 게 정확.
- **통화(currency)는 등록 시 결정**: 수정 API로 바꿀 수는 있지만 `purchase_price` 숫자는 통화와 함께 해석해야 함.
- **본인 엔트리만 접근 가능**: 모든 API가 `user_id`로 필터링. 다른 사용자의 엔트리 조회/수정 시도는 404.
