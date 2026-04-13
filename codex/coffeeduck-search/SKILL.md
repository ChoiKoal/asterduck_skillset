---
name: coffeeduck-search
description: Search and explore coffees on CoffeeDuck — coffee list, name search, flavor notes, varieties. Trigger when user wants to find coffee, browse flavor notes, or look up coffee information. No authentication required.
---

# CoffeeDuck Search — 커피 검색 & 탐색

CoffeeDuck 플랫폼에서 커피를 검색하고 탐색하는 스킬.
커피 검색, 상세 조회, 플레이버 노트, 품종 정보 제공.
인증 불필요 (모든 조회 API는 공개).

## Trigger

다음 키워드에 반응: "커피 검색", "커피 찾기", "커피 목록", "커피 정보", "플레이버 노트", "coffeeduck search"

## API Base URL

```
https://coffeeduckbe-production.up.railway.app/api
```

## 기능

### 1. 커피 목록 조회 (검색 + 페이지네이션)

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffee?page=1&per_page=10"
```

**검색 파라미터:**

| 파라미터 | 설명 | 예시 |
|---------|------|------|
| search | 검색 키워드 | `search=Ethiopia` |
| search_type | 검색 범위 | `all` / `name` / `supplier` / `country` / `varietal` / `flavor_notes` / `flavor_note_ids` |
| flavor_note_ids | 플레이버 노트 ID (쉼표 구분) | `flavor_note_ids=1,5,12` |
| sort_by | 정렬 기준 | `created_at` / `name` / `avg_rating` |
| sort_dir | 정렬 방향 | `asc` / `desc` |
| page | 페이지 번호 (기본: 1) | `page=2` |
| per_page | 페이지당 개수 (기본: 10) | `per_page=20` |

**검색 예시:**
```bash
# 이름으로 검색
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffee?search=Ethiopia&search_type=name"

# 국가로 검색
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffee?search=Kenya&search_type=country"

# 품종으로 검색
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffee?search=Gesha&search_type=varietal"

# 플레이버 노트 ID로 필터링
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffee?flavor_note_ids=1,5&search_type=flavor_note_ids"

# 평점순 정렬
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffee?sort_by=avg_rating&sort_dir=desc"
```

**응답:**
```json
{
  "success": true,
  "coffees": [
    {
      "id": 1,
      "name": "Ethiopia Yirgacheffe",
      "supplier": "Fritz Coffee",
      "created_at": "2026-01-15",
      "avg_rating": 4.2,
      "comment_count": 5,
      "flavor_notes": [
        {"name": "blueberry", "color": "#4B0082"}
      ]
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 10,
    "total_count": 42,
    "total_pages": 5,
    "has_next": true,
    "has_prev": false
  }
}
```

### 2. 커피 상세 조회

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffee/{coffee_id}"
```

**응답:**
```json
{
  "success": true,
  "coffee": {
    "id": 1,
    "name": "Ethiopia Yirgacheffe",
    "harvest_year": 2025,
    "supplier": "Fritz Coffee",
    "origin": {
      "country": "Ethiopia",
      "region": "Yirgacheffe",
      "producer": null,
      "farm_name": "Konga",
      "altitude_meters": 1900
    },
    "processing": {
      "method": "Washed",
      "fermentation": null
    },
    "varietal": ["heirloom"],
    "flavor_notes": [
      {"id": 1, "name": "blueberry", "color": "#4B0082"}
    ]
  }
}
```

### 3. 커피 이름 검색 (정확 매칭 + 유사 결과)

등록 전 중복 확인에 활용.

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffee/search-by-name?name=Ethiopia"
```

**응답:**
```json
{
  "success": true,
  "coffee": { "id": 1, "name": "Ethiopia Yirgacheffe", "..." : "..." },
  "similar": [
    { "id": 5, "name": "Ethiopia Sidamo", "..." : "..." }
  ],
  "search_name": "Ethiopia"
}
```
- `coffee` — 정확히 일치하는 커피 (없으면 null)
- `similar` — 이름이 유사한 커피 (최대 5개)

### 4. 플레이버 노트 목록

전체 플레이버 노트 마스터 데이터.

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/flavor-notes"
```

**응답:**
```json
{
  "success": true,
  "flavor_notes": [
    {
      "id": 1,
      "name": "blueberry",
      "parentNote_id": 5,
      "level": 3,
      "color": "#4B0082"
    }
  ]
}
```

플레이버 노트는 계층 구조:
- **Level 1**: 대분류 (Fruity, Floral, Nutty 등)
- **Level 2**: 중분류 (Berry, Citrus 등)
- **Level 3**: 세부 향 (Blueberry, Lemon 등) — 테이스팅 노트에서 선택하는 단위

### 5. 인기 플레이버 노트 (Top 12)

커뮤니티 테이스팅 데이터 기반 인기 향.

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffeeduck/flavor-notes/popular"
```

### 6. 커피별 사용자 향미 통계

특정 커피에 대해 사용자들이 선택한 향미 노트 분포.

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffee/{coffee_id}/user-flavor-notes"
```

### 7. 품종 목록

등록된 커피 품종 전체 조회.

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/varieties"
```

### 8. 커피 상세 목록 (전체)

모든 커피의 상세 정보 일괄 조회 (페이지네이션 없음).

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffee/detailed"
```

## 다국어 지원

`Accept-Language` 헤더로 응답 언어 변경 가능:
```bash
# 한국어
curl -s -H "Accept-Language: ko" "https://coffeeduckbe-production.up.railway.app/api/flavor-notes"

# 일본어
curl -s -H "Accept-Language: ja" "https://coffeeduckbe-production.up.railway.app/api/flavor-notes"
```

## 주의사항

- 모든 검색 API는 인증 불필요
- `search-by-name`은 정확 매칭 우선, 유사 결과 보조 반환
- 플레이버 노트 ID는 테이스팅 노트 등록 시 `flavor_note_ids` 배열로 사용
