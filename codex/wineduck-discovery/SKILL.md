---
name: wineduck-discovery
description: WineDuck community palate aggregates and personalized wine recommendations. Trigger when user asks for wine recommendations, wants to compare their palate to community average, or wants to discover similar wines. Recommendations require authentication; community palate average does not.
---
# WineDuck Discovery — 커뮤니티 집계 & 취향 추천

WineDuck 커뮤니티 데이터로 와인을 발견·탐색하는 스킬.
- 커뮤니티 평균 팔레트 (레이더 비교 기준선)
- 사용자 취향 기반 와인 추천 TOP N

## Trigger

다음 키워드에 반응: "추천 와인", "내 취향", "마실 만한 와인 추천", "비슷한 팔레트", "커뮤니티 평균", "다른 사람 평가", "wine recommendation", "for me", "average palate"

## API Base URL

```
https://coffeeduckbe-production.up.railway.app/api/wineduck
```

## 인증

- 커뮤니티 평균 팔레트: **인증 불필요**
- 사용자 추천: 로그인 토큰 필요. asterduck-auth 스킬로 먼저 로그인 (본인의 user_id만 조회 가능).

```
Authorization: Bearer {토큰}
```

## 기능

### 1. 커뮤니티 평균 팔레트

전체 또는 타입별로 5축 팔레트(sweetness/acidity/body/tannin/finish) 평균을 반환. 와인덕 홈/프로필의 팔레트 레이더 차트 오버레이 기준선으로 사용.

```bash
# 전체 평균
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines/aggregates/palate-avg"

# 레드 와인만
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines/aggregates/palate-avg?wine_type=red"
```

#### Query Params

| 파라미터 | 기본값 | 값 |
|---------|--------|-----|
| `wine_type` | (전체) | `red` / `rose` / `white_sparkling` |

#### 응답

```json
{
  "success": true,
  "avg_palate": {
    "sweetness": 2.1,
    "acidity": 3.4,
    "body": 3.8,
    "tannin": 3.2,
    "finish": 3.5
  },
  "sample_count": 142,
  "wine_type": "red"
}
```

- 각 축은 0~5 범위의 평균값. 표본이 없으면 `null`.
- `wine_type` 필터 미지정 시 응답에 `"wine_type": "all"`.

### 2. 사용자 취향 기반 와인 추천

사용자의 테이스팅 이력에서 취향을 학습해 안 마신 와인을 추천. 점수 = 팔레트 유사도(주 신호) + 보너스(선호 타입/국가/품종).

```bash
# 기본 8개 추천
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://coffeeduckbe-production.up.railway.app/api/wineduck/users/{user_id}/recommendations"

# 20개로 확장
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://coffeeduckbe-production.up.railway.app/api/wineduck/users/{user_id}/recommendations?limit=20"
```

> ⚠️ `user_id`는 토큰 소유자 본인이어야 함. 다른 사용자 ID로 호출하면 403.

#### Query Params

| 파라미터 | 기본값 | 값 |
|---------|--------|-----|
| `limit` | 8 | 1 ~ 20 |

#### 게이팅 (need_more)

테이스팅 노트가 `min_count_required`(기본 3) 미만이면 추천 불가:

```json
{
  "success": true,
  "wines": [],
  "need_more": true,
  "min_count_required": 3,
  "user_tasting_count": 1,
  "user_total_count": 1,
  "user_palate_count": 0
}
```

이 경우 사용자에게 "테이스팅 노트를 N개 더 남겨야 추천이 활성화돼" 안내.

#### 정상 응답

```json
{
  "success": true,
  "wines": [
    {
      "wine": {
        "id": 142,
        "canonical_name": "Barolo Marenca",
        "producer": "Giovanni Rosso",
        "wine_type": "red",
        "vintage_year": 2019,
        "country_iso_code": "IT",
        "country_name": "이탈리아",
        "region_name": "피에몬테",
        "appellation_name": "바롤로",
        "appellation_classification": "DOCG",
        "tasting_count": 12,
        "avg_rating": 4.3,
        "community_palate": {
          "sweetness": 1.5,
          "acidity": 4.2,
          "body": 4.5,
          "tannin": 4.8,
          "finish": 4.4
        }
      },
      "score": 1.82,
      "palate_similarity": 0.92,
      "reasons": ["비슷한 팔레트", "#레드", "#이탈리아", "#Nebbiolo"]
    }
  ],
  "need_more": false,
  "min_count_required": 3,
  "user_tasting_count": 18,
  "user_total_count": 18,
  "user_palate_count": 12,
  "candidate_pool_size": 87
}
```

**핵심 필드:**
- `wines[].score`: 종합 점수 (DESC 정렬됨)
- `wines[].palate_similarity`: 0~1 정규화 팔레트 유사도
- `wines[].reasons`: i18n 적용된 매치 사유 태그 — 사용자에게 보여줄 때 그대로 사용 가능
- `user_palate_count`: 평점 4점 이상 와인 수 (팔레트 학습 시그널 — 적으면 추천 정확도 낮음)
- `candidate_pool_size`: 사용자가 안 마신 + 커뮤니티 테이스팅 ≥ 1인 후보 풀 크기

## 스코어링 정책

추천 점수 계산식 (서버):

```
score = palate_similarity                      # 주 신호 (0~1)
        + 0.5 if wine.wine_type == favorite_type  # 가장 자주 평가한 타입 매치
        + 0.2 if wine.country in top_3_countries  # 자주 평가한 국가 매치
        + 0.2 if grape_token_overlap              # 자주 평가한 품종 토큰 매치
```

- **팔레트 학습 시그널**: 4점 이상 평가한 와인의 5축 평균만 사용 (좋아한 와인 기준).
- **타입/국가/품종 시그널**: 모든 평가(평점 무관) 기준 빈도 TOP1 / TOP3.
- **이미 마신 와인 제외**: 후보에서 자동 필터링.

## 대화 예시

### 추천 요청

> 사용자: "내 취향에 맞는 와인 추천해줘"

에이전트:
1. asterduck-auth로 토큰 확보 → user_id 추출
2. `GET /api/wineduck/users/{user_id}/recommendations?limit=8`
3. `need_more: true` → "테이스팅 노트가 N개 더 필요해" 안내
4. 정상 응답 → 상위 5~8개를 `canonical_name (vintage) — producer | reasons` 포맷으로 출력

### 팔레트 비교

> 사용자: "내 팔레트가 평균이랑 얼마나 다른지 보여줘"

에이전트:
1. (옵션) `GET /api/wineduck/wines/aggregates/palate-avg` → 전체 커뮤니티 평균
2. wineduck-tasting의 `/users/{user_id}/palate` 등으로 본인 평균 확보
3. 5축별 차이를 표/문장으로 비교

## 주의사항

- **본인만 추천 조회**: path `user_id`가 토큰의 `user_id`와 다르면 403.
- **빈 추천 ≠ 에러**: `need_more: true`는 정상 응답이며 게이팅 의미. UI에서 안내 메시지로 처리.
- **wine_type 필터 값**: `white_sparkling` 한 단어. `white` 단독 사용 불가 (다른 와인 API와 동일 정책).
- **점수는 절대값 아닌 상대값**: 같은 사용자 내 정렬 비교용. 다른 사용자 점수와 직접 비교 금지.
