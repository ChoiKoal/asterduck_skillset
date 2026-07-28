> ## ⚠️ 사용 전 필독 — 반드시 "원격 최신" 버전으로 동작하라
> 이 사본은 구버전일 수 있다. **파일에 적힌 버전 숫자를 신뢰하지 말고**, 사용 전 원격 main과 직접 대조해 최신을 적용하라:
> 1. 쓰기 가능한 위치에 최신 사본 확보:
> `git -C /tmp/asterduck_skillset pull -q 2>/dev/null || git clone -q https://<GITHUB_TOKEN>@github.com/ChoiKoal/asterduck_skillset.git /tmp/asterduck_skillset`
> 2. 배포 사본과 원격 버전 대조: `cat VERSION`(배포) vs `cat /tmp/asterduck_skillset/VERSION`(원격 최신)
> 3. **다르면 = 배포본 구버전** → `/tmp/asterduck_skillset/`의 SKILL.md·CHANGELOG·OpenAPI를 **우선**해 동작. (상세: `AGENT_UPDATE.md`)
> 구버전 관성/캐시 지식 금지. `<GITHUB_TOKEN>`은 그룹 CLAUDE.md 또는 배포 사본 git remote에 있음.
> **🔴 호환성(전체는 CHANGELOG)**: 와인 카탈로그의 연도 음용적기(`drink_*_year`)와 셀러 엔트리의 날짜(`drink_from`/`drink_until`)는 모두 현행 API에서 지원된다.

# WineDuck Wine — 와인 등록 & 관리

WineDuck 플랫폼에 새로운 와인을 등록하고 관리하는 스킬.
중복 검색, 와인명 규칙 가이드, 아펠라시옹 매핑을 포함.
인증 필요 (asterduck-auth 스킬과 함께 사용).

## Trigger

다음 키워드에 반응: "와인 등록", "와인 추가", "새 와인", "wine register", "wineduck wine", "와인 올려", "와인 만들어"

## API Base URL

```
https://coffeeduckbe-production.up.railway.app/api/wineduck
```

## 인증

와인 등록은 로그인 토큰 필요. asterduck-auth 스킬로 먼저 로그인.

```
Authorization: Bearer {토큰}
```

## 등록 프로세스

### Step 1: 중복 확인 (필수!)

등록 전 **반드시** wineduck-search 스킬로 기존 와인 검색.

```bash
# 이름 + 빈티지로 검색
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines/search?name=Gevrey&vintage=2023"
```

- **이름 + 빈티지 일치** → "이미 등록되어 있습니다" 알림
- **이름 유사 + 빈티지 다름** → "같은 와인의 다른 빈티지가 있습니다. 새로 등록할까요?" 확인
- **검색 결과 없음** → Step 2 진행

### Step 2: 와인 정보 수집

사용자에게 필요한 정보 확인:

| 필드 | 필수 | 설명 |
|------|------|------|
| canonical_name | ✅ | 와인 이름 (아래 명명 규칙 참조) |
| wine_type | ❌ | `red` / `white` / `sparkling` / `rose` (레거시 `white_sparkling` 호환) |
| producer | ❌ | 생산자 (도멘/네고시앙/와이너리) |
| vintage_year | ❌ | 빈티지 연도 (NV는 null) |
| grapes_text | ❌ | 품종 (쉼표 구분) |
| drink_from_year | ❌ | 음용 적기 시작 연도 (예: 2026) |
| drink_until_year | ❌ | 음용 적기 종료 연도 (예: 2034) |
| peak_year | ❌ | 음용 피크 연도 (예: 2030) |
| country_id 또는 country_iso_code | ❌ | 기존 국가 ID 또는 ISO 코드(예: `FR`) |
| region_id 또는 region_name | ❌ | 기존 지역 ID 또는 정확한 마스터 이름 |
| appellation_id 또는 appellation_name | 🔴 필수급(강력권장) | 아펠라시옹 ID — **와인명에 아펠라시옹이 드러나면(Chablis, Marsannay, Gevrey-Chambertin, Barolo 등) 반드시 채울 것.** region만 채우고 appellation을 비워두면 안 됨(부르고뉴/지역만 뜨고 세부가 사라짐). 마스터에 정확한 1er Cru/Grand Cru 항목이 없으면 상위 아펠라시옹(예: Chablis 1er Cru → `Chablis`)으로라도 매핑. **미연결 와인은 정복 도감에 안 뜬다** (아래 Step 2.5 필수). |

> **음용 적기(drink window)** — 와인 자체의 일반 권장 음용 시기를 *연도*로 저장한다. 모두 선택값이며 연도(1900~2100) 정수. 알면 채우고, 모르면 생략. 정합성: `drink_from_year ≤ peak_year ≤ drink_until_year` (어긋나면 400). NV/빈티지 미상이면 비워둔다.
>
> ⚠️ **혼동 금지 — 비슷한 이름의 다른 필드 2개가 있다:**
> - **여기(와인 등록)**: `drink_from_year` / `drink_until_year` / `peak_year` — **연도(절대값)**, 와인 카탈로그의 일반 권장 적기.
> - **시음 노트 등록 스킬의 `drink_window`**: enum `now` / `within_1_5y` / `over_5y` — 내가 *마셔본* 주관적 인상(상대값). ← 와인 등록엔 쓰지 않는다.
> 와인을 *등록*할 때는 항상 연도 필드(`drink_*_year`)를 쓰고, enum `drink_window`는 시음 노트에서만 쓴다.

### Step 2.5: 아펠라시옹 매핑 (정복 도감 집계 기반 — 필수)

와인을 정복 도감에 잡히게 하려면 아펠라시옹 연결이 필수다. 등록 전 반드시:

1. **정본 resolve** — `GET /wineduck/appellations/search?q=<아펠라시옹명>`으로 아펠라시옹 마스터를 검색한다(인증 불필요, 상위 10건, prefix 우선 결정적 순서). 이 검색이 **와인 등록의 정본 아펠라시옹 resolve 단계**다. 필요하면 아래 "지역/아펠라시옹 ID 매핑"의 `countries` → `regions`(+`include_sub`) → `appellations` 계층 조회로 보강할 수 있다.
2. **후보 확정** — 반환된 후보의 `name`/`region`/`country`가 **라벨(생산지)과 일치하는지 대조**해 `id`(=`appellation_id`)를 확정한다. 표기 흔들림(악센트/한글/띄어쓰기) 주의, 신규 남발 금지.
3. **없으면 상위 아펠라시옹으로** — 정확한 1er/Grand Cru 항목이 마스터에 없으면 상위 아펠라시옹(예: Chablis 1er Cru → `Chablis`)으로라도 연결. region만 남기고 appellation을 비우지 말 것.
4. **확신이 없으면 비워라** — 후보가 없거나 모호하거나 라벨과 일치하지 않으면 **`appellation_id`를 아예 넣지 않는다(생략).** 추측은 금지다. **매핑이 없는 것이 틀린 매핑보다 낫다** — 틀린 아펠라시옹은 정복 도감을 오염시키고 운영자가 사후에 정리해야 한다.

> 아펠라시옹 미연결로 등록된 와인은 정복 도감에서 집계되지 않아 도감이 비어 보인다. 그렇다고 **틀린 아펠라시옹을 추측으로 채우지 말 것** — 미연결(생략)은 나중에 채우면 되지만, 틀린 연결은 잘못된 정복으로 집계되어 더 나쁘다.

```bash
# 아펠라시옹 정본 검색 (와인 등록 resolve 단계)
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/appellations/search?q=Gevrey"
```

**응답:**
```json
{
  "success": true,
  "items": [
    {
      "id": 100,
      "name": "Gevrey-Chambertin",
      "name_ko": "주브레샹베르탱",
      "region": "코트 드 뉘",
      "country": "프랑스",
      "classification": "village"
    }
  ]
}
```

> `q`는 필수이며 빈/공백 문자열이나 허용 길이(100자) 초과 시 400이다. 결과는 상위 10건, prefix 매칭이 먼저 오는 결정적 순서다. `region`/`country`는 로케일이 반영된 표시 문자열이다(ko 로케일이면 위처럼 한글) — `Accept-Language` 규칙과 동일하며, 라벨 대조는 `name`(원어) 기준으로 한다.

> 백엔드 필수값은 `canonical_name` 하나다. 다만 검색·정복·탐색 품질을 위해 타입과 지리 연결을 가능한 한 채운다. 이름/ID 방식 모두 지원하지만 마스터에 없는 지리는 400이며 이 API가 새 국가·지역·아펠라시옹을 만들지 않는다.

### Step 3: 등록 API 호출

```bash
curl -s -X POST https://coffeeduckbe-production.up.railway.app/api/wineduck/wines \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "canonical_name": "Gevrey-Chambertin Vieilles Vignes",
    "wine_type": "red",
    "producer": "Domaine Geantet-Pansiot",
    "vintage_year": 2023,
    "grapes_text": "Pinot Noir",
    "drink_from_year": 2026,
    "drink_until_year": 2034,
    "peak_year": 2030,
    "country_id": 1,
    "region_id": 5,
    "appellation_id": 9
  }'
```

**응답 (HTTP 201):**
```json
{
  "success": true,
  "message": "와인이 등록되었습니다.",
  "wine_id": 42,
  "resolved": {
    "country_id": 1, "region_id": 5, "appellation_id": 9,
    "country_code": "FR", "region": "Côte de Nuits"
  }
}
```

## 와인명 작성 규칙 (canonical_name)

와인 이름은 3가지 유형으로 나뉨:

### Type A: 아펠라시옹 = 와인명 (구세계)

프랑스, 이탈리아 등 "어디서 만들었냐"가 이름인 와인.

- **생산자를 이름에 포함하지 않음** → producer 필드에 별도 저장
- 형식: `[아펠라시옹] [크뤼/리외디/특수표기]`

✅ 올바른 예시:
- `Gevrey-Chambertin Vieilles Vignes`
- `Barolo Marenca`
- `Pouilly-Fuissé 1er Cru Clos de France`
- `Marsannay Les Genetières`
- `Sancerre Monts de St-Rouelle`

❌ 잘못된 예시:
- ~~Geantet-Pansiot Gevrey-Chambertin~~ (생산자 포함 금지)

### Type B: 생산자 + 품종 = 와인명 (신세계)

미국, 호주, 뉴질랜드 등 "뭘로 만들었냐"가 이름인 와인.

- **반드시 생산자/브랜드를 앞에 붙임** (품종명 단독 사용 금지)
- 형식: `[생산자/브랜드] [품종명]`

✅ 올바른 예시:
- `Far Niente Cabernet Sauvignon`
- `The Hunting Lodge Pinot Noir`
- `Meroi Pinot Grigio`

❌ 잘못된 예시:
- ~~Cabernet Sauvignon~~ (품종만 단독 사용 금지)
- ~~Pinot Noir~~ (어떤 생산자 건지 모름)

### Type C: 브랜드 = 와인명

브랜드 자체가 와인의 이름인 경우.

- 형식: `[브랜드명]`
- 예시: `Opus One`, `Sassicaia`, `Tignanello`

### 특수 표기

- 보르도 등급: `Château Lafon-Rochet Saint-Estèphe 4ème Grand Cru Classé`
- 1er Cru: `Pouilly-Fuissé 1er Cru Clos de France`
- Vieilles Vignes: `Gevrey-Chambertin Vieilles Vignes`
- 싱글 빈야드: `Barolo Marenca`

### 절대 하지 말 것

- ❌ 빈티지를 이름에 포함 (vintage_year 필드 별도)
- ❌ Type A에서 생산자를 이름에 붙이기
- ❌ Type B에서 품종명만 단독 사용

## 지역/아펠라시옹 ID 매핑 (중요)

> 🚨 **반드시 기존 ID를 우선 매핑할 것.** 같은 region/appellation을 새로 만들면 데이터가 오염되고, 탐색 페이지에서 분리되어 표시됨.

### 매핑 규칙 (등록 전 필수 절차)

0. **(권장 시작점) 아펠라시옹 정본 검색** — 아펠라시옹명을 알면 `GET /api/wineduck/appellations/search?q=<이름>`으로 바로 후보 `id`를 resolve한다(상위 10건, prefix 우선). 후보의 `region`/`country`로 아래 계층 매핑도 함께 확정할 수 있다.
1. `GET /api/wineduck/countries` 로 국가 목록을 받아 **이름 정확 매칭**으로 country_id 확정
2. `GET /api/wineduck/countries/{country_id}/regions` 로 지역 목록을 받아 매칭
   - 부르고뉴처럼 sub-region이 있으면 `?include_sub=true` 옵션 사용 가능
   - 이름이 미묘하게 다른 케이스(예: "Côte Chalonnaise" vs "Cote Chalonnaise" vs "코트 샬로네즈") **우선 한글명/영문명 둘 다 비교**해서 동일 지역 ID 사용
3. `GET /api/wineduck/regions/{region_id}/appellations` 로 아펠라시옹 매칭
4. **기존 ID가 없을 때만** 신규 추가 — 그것도 단순 추측 금지. 사용자에게 "이 아펠라시옹이 새로 등록되어야 하는데, 정말 신규가 맞아?" 확인 후 진행. **모호하면 `appellation_id`를 비워둔다(추측 금지, 미연결이 오연결보다 낫다).**

### ❌ 절대 하지 말 것
- "비슷해 보이지만 다른 이름이라 새로 만든다" — 중복 region/appellation의 주범
- "검색했더니 첫 페이지에 없어서 새로 만든다" — 페이지네이션/include_sub 확인 안 한 채로 신규 등록 금지
- 표기 흔들림(악센트, 한글 표기, 띄어쓰기)으로 다른 ID라고 판단

### 국가 조회
```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/countries"
```

### 지역 조회
```bash
# 프랑스의 지역 (sub-region 포함)
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/countries/1/regions"
```

### 아펠라시옹 조회
```bash
# 이름으로 정본 검색 (등록 resolve 단계 — 권장 시작점)
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/appellations/search?q=Gevrey"

# 코트 드 뉘의 아펠라시옹
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/regions/5/appellations"

# 부르고뉴 전체(하위 지역 포함)
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/regions/4/appellations?include_sub=true"
```

## wine_type 값

| 값 | 설명 |
|----|------|
| `red` | 레드 와인 |
| `rose` | 로제 와인 |
| `white` | 화이트 와인 |
| `sparkling` | 스파클링 와인 |
| `white_sparkling` | 레거시 통합값(하위 호환 전용) |
## 대화형 등록 예시

사용자가 자연어로 와인 등록을 요청하면:

> "Gevrey-Chambertin Vieilles Vignes 2023 등록해줘. 쟝테팡시오 도멘이야. 피노 누아."

→ 처리 순서:
1. `wines/search?name=Gevrey-Chambertin&vintage=2023` 로 중복 확인 — 카탈로그는 **빈티지별 별도 row**다(같은 이름 2021/2022는 다른 와인). quick-tasting에 `wine_id`를 지정하면 name/type/vintage는 카탈로그가 정본이며 불일치 시 `409 WINE_ID_FIELD_MISMATCH`가 반환된다 (상세는 wineduck-tasting 스킬)
2. Type A 판단 → canonical_name: "Gevrey-Chambertin Vieilles Vignes"
3. producer: "Domaine Geantet-Pansiot"
4. **아펠라시옹 resolve** — `appellations/search?q=Gevrey-Chambertin`로 후보 조회 → `name`/`region`/`country`가 라벨과 일치하는 `id=100` 확정 → appellation_id=100 (country/region은 이름 방식으로 함께 전달 가능). **일치하는 후보가 없으면 appellation_id 생략**
5. 사용자에게 확인 → 등록

## 주의사항

- **중복 확인 없이 등록하지 말 것** — 같은 와인이 여러 개 생기면 데이터 오염
- **기존 region/appellation ID 우선** — 검색 결과를 충분히 확인한 뒤에야 신규 추가. 표기 흔들림으로 신규 ID를 만들면 운영자가 사후에 정리해야 함
- producer + 아펠라시옹 + 빈티지가 같으면 이름이 약간 달라도 동일 와인으로 판단
- 지리 필드는 API 필수는 아니지만 누락 시 지리 탐색·정복 품질이 떨어진다. 가능한 경우 country/region/appellation을 함께 연결
