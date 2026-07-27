> ## ⚠️ 사용 전 필독 — 반드시 "원격 최신" 버전으로 동작하라
> 이 사본은 구버전일 수 있다. **파일에 적힌 버전 숫자를 신뢰하지 말고**, 사용 전 원격 main과 직접 대조해 최신을 적용하라:
> 1. 쓰기 가능한 위치에 최신 사본 확보:
> `git -C /tmp/asterduck_skillset pull -q 2>/dev/null || git clone -q https://<GITHUB_TOKEN>@github.com/ChoiKoal/asterduck_skillset.git /tmp/asterduck_skillset`
> 2. 배포 사본과 원격 버전 대조: `cat VERSION`(배포) vs `cat /tmp/asterduck_skillset/VERSION`(원격 최신)
> 3. **다르면 = 배포본 구버전** → `/tmp/asterduck_skillset/`의 SKILL.md·CHANGELOG·OpenAPI를 **우선**해 동작. (상세: `AGENT_UPDATE.md`)
> 구버전 관성/캐시 지식 금지. `<GITHUB_TOKEN>`은 그룹 CLAUDE.md 또는 배포 사본 git remote에 있음.
> **🔴 호환성(전체는 CHANGELOG)**: 음용적기 = 와인 카탈로그 연도속성 `drink_from_year`/`peak_year`/`drink_until_year`. 셀러 등록 시 날짜(`drink_from`/`drink_until`) **폐기**.


---
name: wineduck-conquest
description: WineDuck conquest & vintage — see how much of the world wine map you've conquered by country/region/appellation/vintage, plus dex (appellation list), badges, and per-vintage wine browsing. Trigger when the user asks about their conquest rate, region dex, vintages tasted, or badges. Requires authentication (own data).
---

# WineDuck Conquest — 정복 도감 & 빈티지

내가 마신 와인을 국가/지역/아펠라시옹/빈티지 단위로 집계해 "세계 와인 지도 정복률"과 도감·뱃지를 보여주는 스킬.
로그인한 본인 데이터 기준이므로 **인증 필요**.

## Trigger

다음 키워드에 반응: "정복", "정복률", "도감", "내 와인 지도", "얼마나 마셨", "부르고뉴 몇 개", "빈티지별", "몇 년산 마셨", "뱃지", "conquest", "vintage", "얼마나 정복"

## API Base URL

```
https://coffeeduckbe-production.up.railway.app/api/wineduck
```

모든 엔드포인트는 `Authorization: Bearer {token}` 필요 (공용 `asterduck-auth` 스킬로 토큰 발급). 응답은 `Cache-Control: no-store` (유저별 실시간).
지역/아펠라시옹/국가 이름은 `Accept-Language`(ko/ja/en)에 따라 현지화되어 내려옵니다.

## 개념 계층

```
Country(국가) → Region(지역) → Appellation(아펠라시옹) → Vintage(빈티지 연도)
```
- **정복** = 그 아펠라시옹의 와인을 하나라도 마신 것.
- **정복률** = (마신 아펠라시옹 수) / (그 지역 전체 아펠라시옹 수).
- **빈티지 정복** = 한 아펠라시옹 안에서 마신 연도(distinct vintage)를 별도로 수집.
- 빈티지는 **와인 row 단위**로 잡힌다 — quick-tasting이 `(이름, 타입, 빈티지)`로 와인을 매칭/생성하므로, 같은 와인의 다른 빈티지를 마시면 각각 별도 row로 쌓여 빈티지 정복이 온전히 집계된다. (NV는 `vintage_year=null` 별도 row)

> ⚠️ 와인이 아펠라시옹에 연결돼 있어야 정복 집계에 잡힘. 미연결 와인은 도감에 안 뜸 — 등록(wine·quick-tasting) 시 아펠라시옹 매핑 필수.

## 기능

### 1. 전체 정복 요약 — `GET /me/conquest`
내 정복 총계 + 마신 지역별 정복률(rate 내림차순).
```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/me/conquest" -H "Authorization: Bearer $TOKEN"
```
```json
{ "success": true,
  "totals": { "tasted": 70, "countries": 10, "regions": 31, "appellations": 25, "vintages": 14 },
  "regions": [ { "region_id": 1, "name": "메독", "country": "프랑스", "country_id": 1,
                 "appellations_total": 6, "appellations_conquered": 4, "rate": 0.6667 } ] }
```

### 2. 지역 도감 — `GET /me/conquest/region/{id}` (옵션 `?vintage=<year|NV>`)
그 지역 전체 아펠라시옹 + 정복 여부. `?vintage=`로 특정 빈티지 기준 필터.
```bash
curl -s ".../api/wineduck/me/conquest/region/17" -H "Authorization: Bearer $TOKEN"
curl -s ".../api/wineduck/me/conquest/region/17?vintage=2018" -H "Authorization: Bearer $TOKEN"
```
```json
{ "success": true, "region": { "id": 17, "name": "피에몬테", "country": "이탈리아" },
  "my_vintages": [2019, 2018],
  "appellations": [ { "appellation_id": 26, "name": "바롤로", "classification": "docg", "conquered": true, "vintages_count": 1 },
                    { "appellation_id": 28, "name": "바르베라 다스티", "conquered": false, "vintages_count": 0 } ] }
```

### 3. 아펠라시옹 빈티지 타임라인 — `GET /me/conquest/appellation/{id}`
```bash
curl -s ".../api/wineduck/me/conquest/appellation/8" -H "Authorization: Bearer $TOKEN"
```
```json
{ "success": true, "appellation": { "id": 8, "name": "샤블리", "region": "부르고뉴" },
  "vintages": [ { "vintage": 2021, "wine_id": 218, "tasting_id": 306, "rating": 4.3,
                  "wine_name": "윌리엄 페브르 샤블리", "tasted_at": "2026-06-27T20:00:00" } ] }
```
`vintage`가 `null`이면 빈티지 미상(NV).

### 4. 뱃지 — `GET /me/badges`
```bash
curl -s ".../api/wineduck/me/badges" -H "Authorization: Bearer $TOKEN"
```
```json
{ "success": true, "badges": [ { "code": "first_flag", "label": "첫 깃발", "earned": true },
   { "code": "three_countries", "label": "3개국 달성", "earned": false, "progress": { "cur": 2, "target": 3 } } ] }
```

### 5. 빈티지 연도별 집계 — `GET /me/vintages`
```bash
curl -s ".../api/wineduck/me/vintages" -H "Authorization: Bearer $TOKEN"
```
```json
{ "success": true, "vintages": [ { "vintage": 2020, "count": 12 }, { "vintage": null, "count": 2 } ] }
```

### 6. 특정 빈티지 와인 — `GET /me/vintages/{year}` (year = 정수 또는 `NV`)
```bash
curl -s ".../api/wineduck/me/vintages/2017" -H "Authorization: Bearer $TOKEN"
```
```json
{ "success": true, "vintage": 2017,
  "wines": [ { "tasting_id": 9001, "wine_id": 501, "wine_name": "샤토 파비 생테밀리옹", "wine_type": "red",
               "rating": 4.5, "region": "보르도", "appellation": "생테밀리옹", "country": "프랑스",
               "tasted_at": "2026-07-03T20:15:00" } ] }
```

### 7. 정복 보드 — 현재 전체 카탈로그

신규 클라이언트의 정식 전체 정복 화면이다. 좌표 없이 국가 → 지역 → 아펠라시옹 계층, 직접/롤업 정복, 셀러 병수, 빈티지, 즐겨찾기를 제공한다.

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/me/conquest/board?vintage=2018" \
  -H "Authorization: Bearer $TOKEN"
# vintage=NV 또는 생략 가능
```

- `self_conquered`: 해당 아펠라시옹 직접 시음 여부; `conquered`: 하위 정복까지 롤업
- `cellar_bottles`: 활성·보유중인 잔여 병수(`quantity - consumed_quantity`, 0 미만 제외). 셀러 보유만으로 정복되지는 않음
- `vintages`: 중복 제거, 빈티지 미상은 문자열 `NV`
- 스키마 준비 전에는 503과 `WINEDUCK_BOARD_SCHEMA_NOT_READY` 반환
- `GET /me/conquest/map`은 공개 호환성을 위해 유지되는 deprecated 좌표 기반 경로이며 신규 흐름은 board 사용

## 사용 예시 (자연어)
> "나 부르고뉴 얼마나 정복했어?" · "피에몬테 아직 안 마신 곳?" · "2017년산 마신 와인?" · "내 뱃지?"

## 연동
기록을 늘리려면 `wineduck-tasting`(빠른 기록은 `vintage_year` 지원) / `wineduck-wine`로 등록 → 정복률·빈티지 즉시 반영.
