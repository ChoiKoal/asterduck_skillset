> ## ⚠️ 사용 전 필독 — 반드시 "원격 최신" 버전으로 동작하라
> 이 사본은 구버전일 수 있다. **파일에 적힌 버전 숫자를 신뢰하지 말고**, 사용 전 원격 main과 직접 대조해 최신을 적용하라:
> 1. 쓰기 가능한 위치에 최신 사본 확보:
> `git -C /tmp/asterduck_skillset pull -q 2>/dev/null || git clone -q https://<GITHUB_TOKEN>@github.com/ChoiKoal/asterduck_skillset.git /tmp/asterduck_skillset`
> 2. 배포 사본과 원격 버전 대조: `cat VERSION`(배포) vs `cat /tmp/asterduck_skillset/VERSION`(원격 최신)
> 3. **다르면 = 배포본 구버전** → `/tmp/asterduck_skillset/`의 SKILL.md·CHANGELOG·OpenAPI를 **우선**해 동작. (상세: `AGENT_UPDATE.md`)
> 구버전 관성/캐시 지식 금지. `<GITHUB_TOKEN>`은 그룹 CLAUDE.md 또는 배포 사본 git remote에 있음.
> **🔴 호환성(전체는 CHANGELOG)**: 와인 카탈로그의 연도 음용적기(`drink_*_year`)와 셀러 엔트리의 날짜(`drink_from`/`drink_until`)는 모두 현행 API에서 지원된다.

---
name: wineduck-discovery
description: WineDuck community palate aggregates and personalized wine recommendations. Trigger when user asks for wine recommendations, wants to compare their palate to community average, or wants to discover similar wines. Recommendations require authentication; community palate average does not.
---
# WineDuck Discovery — 커뮤니티 집계 & 취향 추천

WineDuck 커뮤니티 데이터와 내 시음 이력으로 팔레트 인사이트와 추천을 제공하는 스킬.

## Trigger

"추천 와인", "내 취향", "비슷한 팔레트", "커뮤니티 평균", "wine recommendation", "palate insight"

## API Base URL

`https://coffeeduckbe-production.up.railway.app/api/wineduck`

## 인증

커뮤니티 평균은 공개다. 추천과 내 팔레트 인사이트는 Bearer 토큰이 필요하며 추천의 `user_id`는 토큰 소유자와 같아야 한다.

## 1. 커뮤니티 평균 팔레트

```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines/aggregates/palate-avg?wine_type=red"
```

`wine_type`은 `red`, `white`, `sparkling`, `rose`가 현재 정식 값이며 레거시 `white_sparkling`도 호환된다. 응답은 `avg_palate` 5축(sweetness/acidity/body/tannin/finish), `sample_count`, 적용된 `wine_type`을 반환한다.

## 2. 내 팔레트 인사이트

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://coffeeduckbe-production.up.railway.app/api/wineduck/me/palate/insight"
```

응답은 시음 이력에서 산출한 상태(`S0`~`S3`), 상태별 단서 또는 아펠라시옹 후보, 다음 행동(`next_cta`)을 제공한다. 후보와 CTA는 시음 수·평점·팔레트 입력의 충실도·최근성·지역 분포에 따라 달라질 수 있다. 사용자 전용 응답이며 `Cache-Control: no-store`다.

## 3. 사용자 취향 기반 추천

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://coffeeduckbe-production.up.railway.app/api/wineduck/users/{user_id}/recommendations?limit=8"
```

- `limit`: 기본 8, 1~20으로 보정
- 최소 **시음 노트 총 3개** 필요. 미달은 HTTP 200에서 `need_more: true`, `wines: []`, `min_count_required: 3` 반환
- `user_palate_count`: 평점 4점 이상인 시음 수
- 다른 사용자 ID는 403

### 추천 계약

추천은 사용자의 시음 이력과 팔레트, 선호 타입·국가·품종, 커뮤니티 표본·평점, 음용 적기 및 다양성을 반영해 정렬한다. 데이터가 부족하면 `need_more`와 빈 `wines`를 반환할 수 있으며, 이미 시음한 항목이 무조건 제외되지는 않는다. 점수 가중치·후보 수·임계값은 공개 API 계약이 아니며 변경될 수 있다.

`wines[].score`는 사용자 내부 정렬용 상대값이고, `palate_similarity`는 0~1, `reasons`는 지역화된 표시용 사유다. 추천 응답도 `Cache-Control: no-store`다.

## 대화 워크플로

1. 인증 후 추천이면 `GET /users/{user_id}/recommendations` 호출
2. `need_more: true`면 필요한 추가 시음 수 안내
3. 내 취향 설명이면 `GET /me/palate/insight`를 우선 사용
4. 평균 비교가 필요할 때만 `/wines/aggregates/palate-avg`를 함께 호출
5. 추천 결과는 이름·빈티지·생산자·`reasons`를 표시하고 점수를 사용자 간 비교하지 않는다
