---
name: coffeeduck-coffee
description: Register new coffees on CoffeeDuck — duplicate check, origin/processing/variety mapping. Trigger when user wants to add a new coffee to the platform. Requires authentication.
---

# CoffeeDuck Coffee — 커피 등록 & 관리

CoffeeDuck 플랫폼에 새로운 커피를 등록하는 스킬.
중복 검색, 산지/가공법/품종 매핑을 포함.
인증 필요 (asterduck-auth 스킬과 함께 사용).

## Trigger

다음 키워드에 반응: "커피 등록", "커피 추가", "새 커피", "coffee register", "coffeeduck coffee", "커피 올려"

## API Base URL

```
https://coffeeduckbe-production.up.railway.app/api
```

## 인증

커피 등록은 로그인 토큰 필요. asterduck-auth 스킬로 먼저 로그인.

```
Authorization: Bearer {토큰}
```

## 등록 프로세스

### Step 1: 중복 확인 (필수!)

등록 전 **반드시** coffeeduck-search 스킬로 기존 커피 검색.

```bash
# 이름으로 검색
curl -s "https://coffeeduckbe-production.up.railway.app/api/coffee/search-by-name?name=Ethiopia%20Yirgacheffe"
```

- **정확 매칭 있음** → "이미 등록되어 있습니다" 알림
- **유사 결과만 있음** → "비슷한 커피가 있는데, 새로 등록할까?" 확인
- **검색 결과 없음** → Step 2 진행

### Step 2: 커피 정보 수집

사용자에게 필요한 정보 확인:

| 필드 | 필수 | 설명 |
|------|------|------|
| name | ✅ | 커피 이름 |
| roastery | ✅ | 로스터리/공급처 |
| coffee_type | ❌ | `single_origin` (기본) / `blend` |
| origin.country | ⚠️ | 산지 국가 (싱글 오리진 시 필수) |
| origin.region | ❌ | 산지 지역 |
| origin.farm_name | ❌ | 농장명 |
| origin.producer | ❌ | 생산자 |
| origin.altitude_meters | ❌ | 재배 고도 (m) |
| processing | ❌ | 가공법 (Washed, Natural, Honey 등) |
| fermentation | ❌ | 발효 방식 |
| harvest_year | ❌ | 수확 연도 |
| variety | ❌ | 품종 (쉼표 구분 또는 배열) |
| flavor_notes | ❌ | 대표 향미 (쉼표 구분 또는 배열) |
| detail | ❌ | 상세 메모 |

### Step 3: 등록 API 호출

```bash
curl -s -X POST https://coffeeduckbe-production.up.railway.app/api/coffee \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Ethiopia Yirgacheffe Konga",
    "roastery": "Fritz Coffee",
    "coffee_type": "single_origin",
    "origin": {
      "country": "Ethiopia",
      "region": "Yirgacheffe",
      "farm_name": "Konga",
      "producer": null,
      "altitude_meters": 1900
    },
    "processing": "Washed",
    "fermentation": null,
    "harvest_year": 2025,
    "variety": ["Heirloom"],
    "flavor_notes": ["blueberry", "jasmine", "citrus"],
    "detail": ""
  }'
```

**응답:**
```json
{
  "success": true,
  "message": "커피 정보가 성공적으로 저장되었습니다.",
  "coffee_id": 42,
  "origin_id": 35,
  "processing_id": 12
}
```

## 커피명 작성 규칙

커피 이름은 간결하고 식별 가능하게:

### 기본 형식

`[산지국가] [산지지역] [농장/워싱스테이션]`

✅ 올바른 예시:
- `Ethiopia Yirgacheffe Konga`
- `Kenya Nyeri Karatina AA`
- `Colombia Huila El Paraiso`
- `Guatemala Antigua La Flor`

### 특수 케이스

- **블렌드**: 브랜드명 또는 블렌드명 사용
  - `Fritz House Blend`
  - `Center Coffee Signature`
- **스페셜티 프로세싱**: 가공법 포함 가능
  - `Ethiopia Sidamo Natural`
  - `Costa Rica Honey SL28`

### 절대 하지 말 것

- ❌ 로스터리를 이름에 포함 (roastery 필드에 별도 저장)
- ❌ 수확 연도를 이름에 포함 (harvest_year 필드 별도)

## 가공법 (Processing) 참고

일반적으로 사용되는 가공법:

| 가공법 | 설명 |
|--------|------|
| Washed | 수세식 — 깔끔하고 밝은 산미 |
| Natural | 내추럴 — 과일향, 풍부한 바디 |
| Honey | 허니 — 워시드와 내추럴 중간 |
| Anaerobic | 혐기성 발효 |
| Carbonic Maceration | 탄산침용 |
| Wet Hulled | 습식 탈곡 (인도네시아) |

## 대화형 등록 예시

사용자가 자연어로 커피 등록을 요청하면:

> "프리츠에서 산 이디오피아 예가체프 콩가 등록해줘. 워시드, 1900m, 품종은 에어룸. 블루베리랑 재스민 향이야"

→ 처리 순서:
1. `coffee/search-by-name?name=Ethiopia Yirgacheffe Konga` 로 중복 확인
2. name: "Ethiopia Yirgacheffe Konga"
3. roastery: "Fritz Coffee"
4. origin: {country: "Ethiopia", region: "Yirgacheffe", farm_name: "Konga", altitude_meters: 1900}
5. processing: "Washed"
6. variety: ["Heirloom"]
7. flavor_notes: ["blueberry", "jasmine"]
8. 사용자에게 확인 → 등록

## 주의사항

- **중복 확인 없이 등록하지 말 것** — 같은 커피가 여러 개 생기면 데이터 오염
- name + roastery가 같으면 동일 커피로 판단
- **사용자가 제공한 정보만 등록** — 추측하여 값을 채우지 말 것
- origin.country는 싱글 오리진 커피에서 필수
- variety와 flavor_notes는 문자열(쉼표 구분) 또는 배열 모두 지원
