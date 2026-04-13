# WineDuck Wine — 와인 등록 & 관리

WineDuck 플랫폼에 새로운 와인을 등록하고 관리하는 스킬.
중복 검색, 와인명 규칙 가이드, 아펠라시옹 매핑을 포함.
인증 필요 (wineduck-auth 스킬과 함께 사용).

## Trigger

다음 키워드에 반응: "와인 등록", "와인 추가", "새 와인", "wine register", "wineduck wine", "와인 올려", "와인 만들어"

## API Base URL

```
https://coffeeduckbe-production.up.railway.app/api/wineduck
```

## 인증

와인 등록/수정/삭제는 로그인 토큰 필요. wineduck-auth 스킬로 먼저 로그인.

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
| wine_type | ✅ | red / rose / white_sparkling |
| producer | ❌ | 생산자 (도멘/네고시앙/와이너리) |
| vintage_year | ❌ | 빈티지 연도 (NV는 null) |
| grapes_text | ❌ | 품종 (쉼표 구분) |
| country_id | ✅ | 국가 ID |
| region_id | ✅ | 지역 ID |
| appellation_id | ❌ | 아펠라시옹 ID |

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
    "country_id": 1,
    "region_id": 5,
    "appellation_id": 9
  }'
```

**응답:**
```json
{
  "success": true,
  "message": "와인이 등록되었습니다.",
  "wine_id": 42
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

## 지역/아펠라시옹 ID 조회

등록 시 country_id, region_id를 정확히 매핑해야 탐색 페이지에서 검색 가능.

### 국가 조회
```bash
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/countries"
```

### 지역 조회
```bash
# 프랑스의 지역
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/countries/1/regions"
```

### 아펠라시옹 조회
```bash
# 코트 드 뉘의 아펠라시옹
curl -s "https://coffeeduckbe-production.up.railway.app/api/wineduck/regions/5/appellations"
```

## wine_type 값

| 값 | 설명 |
|----|------|
| `red` | 레드 와인 |
| `rose` | 로제 와인 |
| `white_sparkling` | 화이트 와인, 스파클링 와인 (⚠️ "white" 단독 사용 불가!) |

## 와인 수정

```bash
curl -s -X PUT "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines/{wine_id}" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"producer": "수정된 생산자명"}'
```

## 와인 삭제

```bash
curl -s -X DELETE "https://coffeeduckbe-production.up.railway.app/api/wineduck/wines/{wine_id}" \
  -H "Authorization: Bearer $TOKEN"
```

## 대화형 등록 예시

사용자가 자연어로 와인 등록을 요청하면:

> "Gevrey-Chambertin Vieilles Vignes 2023 등록해줘. 쟝테팡시오 도멘이야. 피노 누아."

→ 처리 순서:
1. `wines/search?name=Gevrey-Chambertin&vintage=2023` 로 중복 확인
2. Type A 판단 → canonical_name: "Gevrey-Chambertin Vieilles Vignes"
3. producer: "Domaine Geantet-Pansiot"
4. country_id=1(FR), region_id=5(Côte de Nuits), appellation_id=9(Gevrey-Chambertin)
5. 사용자에게 확인 → 등록

## 주의사항

- **중복 확인 없이 등록하지 말 것** — 같은 와인이 여러 개 생기면 데이터 오염
- producer + 아펠라시옹 + 빈티지가 같으면 이름이 약간 달라도 동일 와인으로 판단
- country_id, region_id는 필수 — 없으면 탐색 페이지에서 검색 불가
