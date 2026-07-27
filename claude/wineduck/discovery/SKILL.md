> ## ⚠️ 사용 전 필독 — 반드시 "원격 최신" 버전으로 동작하라
> 이 사본은 구버전일 수 있다. **파일에 적힌 버전 숫자를 신뢰하지 말고**, 사용 전 원격 main과 직접 대조해 최신을 적용하라:
> 1. 쓰기 가능한 위치에 최신 사본 확보:
> `git -C /tmp/asterduck_skillset pull -q 2>/dev/null || git clone -q https://<GITHUB_TOKEN>@github.com/ChoiKoal/asterduck_skillset.git /tmp/asterduck_skillset`
> 2. 배포 사본과 원격 버전 대조: `cat VERSION`(배포) vs `cat /tmp/asterduck_skillset/VERSION`(원격 최신)
> 3. **다르면 = 배포본 구버전** → `/tmp/asterduck_skillset/`의 SKILL.md·CHANGELOG·OpenAPI를 **우선**해 동작. (상세: `AGENT_UPDATE.md`)
> 구버전 관성/캐시 지식 금지. `<GITHUB_TOKEN>`은 그룹 CLAUDE.md 또는 배포 사본 git remote에 있음.
> **🔴 호환성(전체는 CHANGELOG)**: 음용적기 = 와인 카탈로그 연도속성 `drink_from_year`/`peak_year`/`drink_until_year`. 셀러 등록 시 날짜(`drink_from`/`drink_until`) **폐기**.

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

시음 수에 따라 `S0`(0), `S1`(1~2), `S2`(3~5), `S3`(6+) 상태를 반환한다. S1은 최고 평점 기록의 단서를, S2/S3은 상위 3개 아펠라시옹 취향 후보를 제공한다. 후보는 최소 2개의 서로 다른 와인이 필요하며 긍정 평가, 평균 평점, 팔레트 완성도/유사도, 90일 반감기 최신성을 조합한다. `next_cta`는 quick tasting, 유사 탐색, 또는 conquest board로 연결된다. 응답은 사용자 전용이며 `Cache-Control: no-store`다.

## 3. 사용자 취향 기반 추천

```bash
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://coffeeduckbe-production.up.railway.app/api/wineduck/users/{user_id}/recommendations?limit=8"
```

- `limit`: 기본 8, 1~20으로 보정
- 최소 **시음 노트 총 3개** 필요. 미달은 HTTP 200에서 `need_more: true`, `wines: []`, `min_count_required: 3` 반환
- `user_palate_count`: 평점 4점 이상인 시음 수
- 다른 사용자 ID는 403

### 현재 추천 정책

선호 팔레트·타입·국가·품종은 우선 평점 4점 이상 기록으로 학습하고, 긍정 표본이 없으면 타입/국가/품종만 전체 시음으로 fallback한다.

후보는 커뮤니티 시음이 1개 이상인 최대 250개다. 커뮤니티 시음 2개 이상이면서 평균 2.5 미만인 와인과, 사용자가 해당 와인에 준 최고 평점이 2점 이하인 와인은 제외한다. **이미 마신 와인이 보편적으로 제외되는 것은 아니다** — 2점 초과로 평가한 기존 와인은 다시 추천될 수 있다.

실제 점수 요인:
- 측정 가능한 5축의 RMS 거리와 차원 커버리지로 계산한 팔레트 유사도; 사용자 팔레트가 있으면 유사도 0.18 미만 제외
- 커뮤니티 표본 신뢰도(`min(1, tasting_count/4)`)를 반영한 유사도 항
- 커뮤니티 평점 품질의 작은 보정
- 현재 연도가 음용 적기 안이면 `+0.05`
- 선호 타입 `+0.10`, 상위 국가 `+0.12`, 품종 토큰 겹침 `+0.20`
- 생산자당 우선 2개 제한(결과가 부족하면 단계적으로 완화)

`wines[].score`는 사용자 내부 정렬용 상대값이고, `palate_similarity`는 0~1, `reasons`는 지역화된 표시용 사유다. 추천 응답도 `Cache-Control: no-store`다.

## 대화 워크플로

1. 인증 후 추천이면 `GET /users/{user_id}/recommendations` 호출
2. `need_more: true`면 필요한 추가 시음 수 안내
3. 내 취향 설명이면 `GET /me/palate/insight`를 우선 사용
4. 평균 비교가 필요할 때만 `/wines/aggregates/palate-avg`를 함께 호출
5. 추천 결과는 이름·빈티지·생산자·`reasons`를 표시하고 점수를 사용자 간 비교하지 않는다
