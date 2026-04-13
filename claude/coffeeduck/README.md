# CoffeeDuck AI Skills

> AI를 통해 [CoffeeDuck](https://asterduck.koalstudio.com) 커피 테이스팅 플랫폼을 활용하세요.

CoffeeDuck은 커피 덕후를 위한 테이스팅 노트 앱입니다. 이 스킬셋을 사용하면 AI 에이전트를 통해 커피 검색, 등록, 테이스팅 노트 작성을 대화형으로 할 수 있습니다.

## Skills

| 스킬 | 설명 | 인증 |
|------|------|------|
| [search](./search/SKILL.md) | 커피 검색, 플레이버 노트, 품종 탐색 | 불필요 |
| [tasting](./tasting/SKILL.md) | 테이스팅 노트 등록/조회/수정/삭제 | 필요 |
| [coffee](./coffee/SKILL.md) | 커피 등록, 중복 검사, 산지/가공법 매핑 | 필요 |

> 인증은 공용 스킬 [`auth`](../auth/SKILL.md)를 사용합니다.

## 설치

### Claude Code (`.claude/skills/` 디렉토리)

```bash
# 레포 클론
git clone https://github.com/ChoiKoal/asterduck_skillset.git

# 공용 인증 스킬 + CoffeeDuck 스킬 설치
cp -r asterduck_skillset/claude/auth ~/.claude/skills/asterduck-auth
cp -r asterduck_skillset/claude/coffeeduck/search ~/.claude/skills/coffeeduck-search
cp -r asterduck_skillset/claude/coffeeduck/tasting ~/.claude/skills/coffeeduck-tasting
cp -r asterduck_skillset/claude/coffeeduck/coffee ~/.claude/skills/coffeeduck-coffee
```

## 사용 예시

### 커피 검색
> "이디오피아 예가체프 커피 검색해줘"
> "게이샤 품종 커피 있어?"
> "블루베리 향 나는 커피 찾아줘"

### 테이스팅 노트 등록 (자연어)
> "오늘 프리츠 이디오피아 마셨는데, 산미 밝고 바디 가벼워. 블루베리랑 꽃향. 4점 줄게"

### 테이스팅 노트 등록 (구조화)
```
커피: Ethiopia Yirgacheffe
산도: 80  바디: 60  당도: 50  쓴맛: 30  여운: 75
로스팅: medium  추출: pour over
총점: 4.0
향: blueberry, jasmine, citrus
한줄평: 블루베리 폭탄, 깔끔한 워시드
```

### 커피 등록
> "프리츠에서 산 이디오피아 예가체프 콩가 등록해줘. 워시드, 1900m, 에어룸 품종."

## 스킬 간 연동

```
auth (공용) ────────────────┐
  │                          │
  │ (토큰 발급)               │
  ▼                          ▼
search ──► coffee ──► tasting
  │          │          │
  │ (검색)    │ (등록)    │ (기록)
  ▼          ▼          ▼
  CoffeeDuck Platform API
```

## API

- **Base URL**: `https://coffeeduckbe-production.up.railway.app/api`
- **인증**: JWT Bearer Token (24시간 유효)
- **응답 형식**: JSON
- **다국어**: `Accept-Language` 헤더 지원 (ko, ja, en)

## License

MIT

---

Built by [Aster.duck](https://asterduck.koalstudio.com) team
