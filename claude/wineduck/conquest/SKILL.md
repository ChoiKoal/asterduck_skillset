# WineDuck Conquest — 정복 도감 & 빈티지

내가 마신 와인을 국가/지역/아펠라시옹/빈티지 단위로 집계해 "세계 와인 지도 정복률"과 도감·뱃지를 보여주는 스킬.
로그인한 본인 데이터 기준이므로 **인증 필요**.

## Trigger

다음 키워드에 반응: "정복", "정복률", "도감", "내 와인 지도", "얼마나 마셨", "부르고뉴 몇 개", "빈티지별", "몇 년산 마셨", "뱃지", "conquest", "vintage", "얼마나 정복"

## API Base URL

```
https://coffeeduckbe-production.up.railway.app/api/wineduck
```

모든 엔드포인트는 `Authorization: Bearer {token}` 필요 (공용 [`auth`](../../auth/SKILL.md) 스킬로 토큰 발급). 응답은 `Cache-Control: no-store` (유저별 실시간).
지역/아펠라시옹/국가 이름은 `Accept-Language`(ko/ja/en)에 따라 현지화되어 내려옵니다.

## 개념 계층

```
Country(국가) → Region(지역) → Appellation(아펠라시옹) → Vintage(빈티지 연도)
```
- **정복** = 그 아펠라시옹의 와인을 하나라도 마신 것.
- **정복률** = (마신 아펠라시옹 수) / (그 지역 전체 아펠라시옹 수).
- **빈티지 정복** = 한 아펠라시옹 안에서 마신 연도(distinct vintage)를 별도로 수집.

## 기능

### 1. 전체 정복 요약

내 정복 총계 + 마신 지역별 정복률(rate 내림차순).

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/me/conquest" \
  -H "Authorization: Bearer $TOKEN"
```

**응답:**
```json
{
  "success": true,
  "totals": { "tasted": 70, "countries": 10, "regions": 31, "appellations": 25, "vintages": 14 },
  "regions": [
    { "region_id": 1, "name": "메독", "country": "프랑스", "country_id": 1,
      "appellations_total": 6, "appellations_conquered": 4, "rate": 0.6667 }
  ]
}
```

### 2. 지역 도감 (아펠라시옹 리스트)

한 지역의 전체 아펠라시옹과 내 정복 여부. `?vintage=` 로 특정 빈티지 기준 필터 가능.

```bash
# 지역 전체 도감
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/me/conquest/region/17" \
  -H "Authorization: Bearer $TOKEN"

# 특정 빈티지(2018) 기준 — 그 해 마신 아펠라시옹만 conquered
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/me/conquest/region/17?vintage=2018" \
  -H "Authorization: Bearer $TOKEN"
# vintage=NV → 빈티지 미상(Non-Vintage)만
```

**응답:**
```json
{
  "success": true,
  "region": { "id": 17, "name": "피에몬테", "country": "이탈리아" },
  "my_vintages": [2019, 2018],
  "appellations": [
    { "appellation_id": 26, "name": "바롤로", "classification": "docg", "conquered": true, "vintages_count": 1 },
    { "appellation_id": 28, "name": "바르베라 다스티", "classification": "docg", "conquered": false, "vintages_count": 0 }
  ]
}
```
- `my_vintages`: 그 지역에서 내가 마신 빈티지 목록(필터 UI용, 필터와 무관하게 전체).

### 3. 아펠라시옹 빈티지 타임라인

한 아펠라시옹에서 내가 마신 와인들(빈티지 내림차순).

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/me/conquest/appellation/8" \
  -H "Authorization: Bearer $TOKEN"
```

**응답:**
```json
{
  "success": true,
  "appellation": { "id": 8, "name": "샤블리", "region": "부르고뉴" },
  "vintages": [
    { "vintage": 2021, "wine_id": 218, "tasting_id": 306, "rating": 4.3,
      "wine_name": "윌리엄 페브르 샤블리", "tasted_at": "2026-06-27T20:00:00" }
  ]
}
```
- `vintage`가 `null`이면 빈티지 미상(NV).

### 4. 뱃지

규칙 기반 획득 뱃지 + 미획득 진행도.

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/me/badges" \
  -H "Authorization: Bearer $TOKEN"
```

**응답:**
```json
{
  "success": true,
  "badges": [
    { "code": "first_flag", "label": "첫 깃발", "earned": true },
    { "code": "three_countries", "label": "3개국 달성", "earned": false, "progress": { "cur": 2, "target": 3 } },
    { "code": "region_master", "label": "지역 마스터", "earned": true }
  ]
}
```

### 5. 빈티지 축 — 연도별 집계

내가 마신 빈티지 연도별 개수(연도 내림차순, NV는 마지막).

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/me/vintages" \
  -H "Authorization: Bearer $TOKEN"
```

**응답:**
```json
{ "success": true, "vintages": [ { "vintage": 2020, "count": 12 }, { "vintage": null, "count": 2 } ] }
```

### 6. 특정 빈티지로 마신 와인 전체

한 연도로 마신 와인 목록(지역 무관). `year`는 정수 또는 `NV`.

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/me/vintages/2017" \
  -H "Authorization: Bearer $TOKEN"
```

**응답:**
```json
{
  "success": true,
  "vintage": 2017,
  "wines": [
    { "tasting_id": 9001, "wine_id": 501, "wine_name": "샤토 파비 생테밀리옹", "wine_type": "red",
      "rating": 4.5, "region": "보르도", "appellation": "생테밀리옹", "country": "프랑스",
      "tasted_at": "2026-07-03T20:15:00" }
  ]
}
```

## 사용 예시 (자연어)

> "나 부르고뉴 얼마나 정복했어?" → `/me/conquest`에서 부르고뉴 rate 확인
> "피에몬테에서 아직 안 마신 아펠라시옹 뭐야?" → `/me/conquest/region/17`에서 `conquered:false`
> "2017년산으로 마신 와인 보여줘" → `/me/vintages/2017`
> "내 뱃지 뭐 땄어?" → `/me/badges`

## 연동

기록을 늘리려면 [`tasting`](../tasting/SKILL.md)(빠른 기록은 `vintage_year` 지원) / [`wine`](../wine/SKILL.md)로 와인·시음을 등록하면 정복률·빈티지에 즉시 반영됩니다.
