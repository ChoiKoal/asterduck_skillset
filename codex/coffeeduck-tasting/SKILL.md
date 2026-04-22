---
name: coffeeduck-tasting
description: Manage coffee tasting notes on CoffeeDuck — create, read, update, delete tasting records. Trigger when user wants to record a coffee tasting, view tasting history, or manage tasting notes. Requires authentication.
---

# CoffeeDuck Tasting — 커피 테이스팅 노트 관리

CoffeeDuck 플랫폼에서 커피 테이스팅 노트를 대화형으로 등록하고 조회하는 스킬.
인증 필요 (asterduck-auth 스킬과 함께 사용).

## Trigger

다음 키워드에 반응: "커피 테이스팅", "커피 노트", "커피 기록", "커피 평가", "커피 마셨", "coffeeduck tasting"

## API Base URL

```
https://coffeeduckbe-production.up.railway.app/api/coffeeduck/tasting
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
커피: Ethiopia Yirgacheffe
날짜: 2026-04-13
산도: 80
바디: 60
당도: 50
쓴맛: 30
여운: 75
로스팅: medium
추출법: pour over
총점: 4.0
한줄평: 블루베리와 재스민 향이 인상적인 깔끔한 워시드
향: blueberry, jasmine, citrus
```

#### 입력 방식 B: 자연어

사용자가 자유롭게 말하면 AI가 파싱:

> "오늘 프리츠 이디오피아 이르가체프 마셨는데, 산미 엄청 밝고 바디는 가벼워. 블루베리랑 꽃향 나고 깔끔해. 4점 줄게"

#### 자연어 파싱 가이드

| 표현 | 필드 | 값 (0-100) |
|------|------|------------|
| "산미 밝다/강하다/높다" | acidity | 70~90 |
| "산미 적당하다" | acidity | 40~60 |
| "산미 약하다/낮다" | acidity | 10~30 |
| "바디 풀/무겁다" | body | 70~90 |
| "바디 미디엄" | body | 40~60 |
| "바디 라이트/가볍다" | body | 10~30 |
| "달다/달콤하다" | sweetness | 60~80 |
| "쓰다/쓴맛 강하다" | bitterness | 70~90 |
| "여운 길다" | finish | 70~90 |
| "여운 짧다" | finish | 10~30 |
| "N점" | rating | N (0.0~5.0) |

### 등록 프로세스

1. **커피 특정** — coffeeduck-search로 커피 검색하여 coffee_id 확보
2. **데이터 파싱** — 구조화 or 자연어에서 테이스팅 데이터 추출
3. **확인** — "이렇게 등록할게, 맞아?" 파싱 결과를 사용자에게 보여주고 확인
4. **API 호출** — 확인 후 등록
5. **결과 보고** — 등록 성공/실패 알림

### 등록 API

```bash
curl -s -X POST https://coffeeduckbe-production.up.railway.app/api/coffeeduck/tasting/coffees/{coffee_id}/notes \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tasted_at": "2026-04-13",
    "acidity": 80,
    "body": 60,
    "sweetness": 50,
    "bitterness": 30,
    "finish": 75,
    "roasting_level": "medium",
    "brew_method": "pour over",
    "rating": 4.0,
    "one_liner": "블루베리와 재스민 향이 인상적인 깔끔한 워시드",
    "flavor_note_ids": [1, 15, 8]
  }'
```

**응답:**
```json
{
  "success": true,
  "message": "테이스팅 노트가 저장되었습니다.",
  "tasting_note_id": 15
}
```

### 2. 테이스팅 노트 조회

#### 특정 커피의 테이스팅 목록 (인증 필요)
```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffeeduck/tasting/coffees/{coffee_id}/notes?page=1&per_page=20" \
  -H "Authorization: Bearer $TOKEN"
```

#### 단일 테이스팅 상세 (인증 필요)
```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffeeduck/tasting/notes/{note_id}" \
  -H "Authorization: Bearer $TOKEN"
```

#### 커피 커뮤니티 통계 (공개)
```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffeeduck/tasting/coffees/{coffee_id}/stats"
```

**응답:**
```json
{
  "success": true,
  "stats": {
    "total_tastings": 12,
    "avg_rating": 4.1,
    "avg_palate": {
      "avg_acidity": 72.5,
      "avg_body": 55.0,
      "avg_sweetness": 48.3,
      "avg_bitterness": 35.0,
      "avg_finish": 65.8
    }
  }
}
```

#### 내 테이스팅 통계 (인증 필요, 본인만)
```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffeeduck/tasting/users/{user_id}/stats" \
  -H "Authorization: Bearer $TOKEN"
```

### 3. 테이스팅 노트 수정 (인증 필요, 본인만)

```bash
curl -s -X PUT "https://coffeeduckbe-production.up.railway.app/api/coffeeduck/tasting/notes/{note_id}" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"rating": 4.5, "one_liner": "수정된 한줄평"}'
```

### 4. 테이스팅 노트 삭제 (인증 필요, 본인만)

```bash
curl -s -X DELETE "https://coffeeduckbe-production.up.railway.app/api/coffeeduck/tasting/notes/{note_id}" \
  -H "Authorization: Bearer $TOKEN"
```

## 필드 레퍼런스

| 필드 | API key | 범위 | 필수 |
|------|---------|------|------|
| 시음 날짜 | tasted_at | YYYY-MM-DD (기본: 오늘) | ❌ |
| 산도 | acidity | 0-100 | ❌ |
| 바디 | body | 0-100 | ❌ |
| 당도 | sweetness | 0-100 | ❌ |
| 쓴맛 | bitterness | 0-100 | ❌ |
| 여운 | finish | 0-100 | ❌ |
| 로스팅 레벨 | roasting_level | light / medium_light / medium / medium_dark / dark | ❌ |
| 추출 방법 | brew_method | 텍스트 (pour over, espresso, aeropress 등) | ❌ |
| 총점 | rating | 0.0-5.0 (소수점 가능) | ✅ |
| 한줄평 | one_liner | 텍스트 (280자 이하) | ❌ |
| 플레이버 노트 | flavor_note_ids | 숫자 배열 (flavor_notes 마스터 테이블의 ID) | ❌ |

## 플레이버 노트 연동

테이스팅 노트에 향미를 기록하려면 `flavor_note_ids`에 플레이버 노트 ID 배열을 전달.
ID는 coffeeduck-search 스킬의 `/api/flavor-notes` 에서 조회.

```json
// 예: blueberry(id:1), jasmine(id:15), citrus(id:8)
"flavor_note_ids": [1, 15, 8]
```

## 와인덕과의 차이점

| 항목 | CoffeeDuck | WineDuck |
|------|-----------|----------|
| 팔레트 범위 | 0-100 | 1-5 |
| 팔레트 축 | acidity, body, sweetness, **bitterness**, finish | sweetness, acidity, body, **tannin**, finish |
| 향미 입력 | flavor_note_ids (마스터 테이블 ID) | aromas (custom label or taxonomy node_id) |
| 추가 필드 | roasting_level, brew_method | aroma_intensity, color, pairing_food, repurchase |

## 주의사항

- **사용자가 말한 정보만 등록** — 추측하여 값을 채우지 말 것
- 빠진 필드는 null (API에서 선택사항으로 처리). 단, **rating은 필수**
- **등록 전 반드시 파싱 결과 확인** — 잘못된 값 방지
- 팔레트 값은 **0-100** 범위 (와인덕의 1-5와 다름!)
- roasting_level은 반드시 정해진 5개 값 중 하나여야 함
