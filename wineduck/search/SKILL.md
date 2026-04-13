# WineDuck Search — 와인 검색 & 탐색

WineDuck 플랫폼에서 와인을 검색하고, 지역/아펠라시옹을 탐색하는 스킬.
인증 없이 사용 가능한 공개 API 기반.

## Trigger

다음 키워드에 반응: "와인 찾아", "와인 검색", "아펠라시옹", "지역 탐색", "어떤 와인", "wineduck search", "wine search", "와인 있어?"

## API Base URL

```
https://coffeeduckbe-production.up.railway.app/api/wineduck
```

## 기능

### 1. 와인 이름 검색

이름과 빈티지로 와인을 검색. 인증 불필요.

```bash
# 이름으로 검색 (부분 매칭)
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines/search?name=Gevrey"

# 이름 + 빈티지 검색
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines/search?name=Barolo&vintage=2020"
```

**응답:**
```json
{
  "success": true,
  "wines": [
    {
      "id": 42,
      "canonical_name": "Gevrey-Chambertin Vieilles Vignes",
      "producer": "Domaine Geantet-Pansiot",
      "wine_type": "red",
      "vintage_year": 2023,
      "country_code": "FR"
    }
  ]
}
```

### 2. 자동완성 검색

2글자 이상 입력 시 빠른 매칭 (prefix → contains 2단계).

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines/autocomplete?q=Pou"
```

### 3. 국가 목록 조회

등록된 와인 생산국 전체 목록.

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/countries"
```

### 4. 지역 목록 조회

특정 국가의 와인 생산 지역 조회. 하위 지역(sub-region) 포함.

```bash
# 프랑스(country_id=1)의 지역 목록
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/countries/1/regions"
```

### 5. 아펠라시옹 목록 조회

특정 지역의 아펠라시옹(원산지 명칭) 목록.

```bash
# 코트 드 뉘(region_id=5)의 아펠라시옹
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/regions/5/appellations"

# 부르고뉴(region_id=4)처럼 하위 지역이 있는 경우, 하위 지역 포함 조회
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/regions/4/appellations?include_sub=true"
```

### 6. 와인 상세 조회

특정 와인의 전체 정보 + 테이스팅 통계.

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines/42"
```

### 7. 와인 목록 (필터링)

국가, 지역, 아펠라시옹, 타입별 와인 목록.

```bash
# 프랑스 레드와인
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines?country_id=1&wine_type=red"

# 특정 아펠라시옹의 와인
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines?appellation_id=9"

# 페이지네이션
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines?page=1&per_page=20"
```

## 탐색 계층 구조

WineDuck의 지역 탐색은 최대 4단계 계층:

```
국가 (Country)
  └─ 지역 (Region) — e.g. Bordeaux, Burgundy
       └─ 하위 지역 (Sub-region) — e.g. Côte de Nuits, Médoc
            └─ 아펠라시옹 (Appellation) — e.g. Gevrey-Chambertin, Pauillac
```

### 주요 국가 ID
| ID | 국가 |
|----|------|
| 1 | France |
| 2 | Italy |
| 3 | Spain |
| 4 | United States |
| 5 | Germany |
| 10 | New Zealand |

## 대화형 검색 예시

사용자가 자연어로 와인을 찾으면 적절한 API를 조합해서 검색:

- "부르고뉴 피노 누아 추천해줘" → 국가=FR, 지역=Burgundy 필터 + 와인 목록 조회
- "바롤로 2019년산 있어?" → name=Barolo&vintage=2019 검색
- "프랑스에 어떤 지역이 있어?" → countries/1/regions 조회
- "코트 드 뉘에 어떤 아펠라시옹이 있어?" → regions/5/appellations 조회

## wine_type 값

| 값 | 설명 |
|----|------|
| red | 레드 와인 |
| rose | 로제 와인 |
| white_sparkling | 화이트 / 스파클링 와인 |
