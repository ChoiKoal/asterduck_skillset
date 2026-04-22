# WineDuck AI Skills

> AI를 통해 [WineDuck](https://asterduck.koalstudio.com) 와인 테이스팅 플랫폼을 활용하세요.

WineDuck은 와인 덕후를 위한 테이스팅 노트 앱입니다. 이 스킬셋을 사용하면 AI 에이전트를 통해 와인 검색, 등록, 테이스팅 노트 작성을 대화형으로 할 수 있습니다.

## Skills

| 스킬 | 설명 | 인증 |
|------|------|------|
| [search](./search/SKILL.md) | 와인 검색, 지역/아펠라시옹 탐색 | 불필요 |
| [tasting](./tasting/SKILL.md) | 테이스팅 노트 등록/조회/수정/삭제 | 필요 |
| [wine](./wine/SKILL.md) | 와인 등록, 중복 검사, 명명 규칙 | 필요 |
| [cellar](./cellar/SKILL.md) | 셀러 목록/통계/음용 임박, 사진으로 셀러 추가, 소비 기록 | 필요 |

> 인증은 공용 스킬 [`auth`](../auth/SKILL.md)를 사용합니다.

## 설치

### Claude Code (`.claude/skills/` 디렉토리)

```bash
# 레포 클론
git clone https://github.com/ChoiKoal/asterduck_skillset.git

# 공용 인증 스킬 + WineDuck 스킬 설치
cp -r asterduck_skillset/claude/auth ~/.claude/skills/asterduck-auth
cp -r asterduck_skillset/claude/wineduck/search ~/.claude/skills/wineduck-search
cp -r asterduck_skillset/claude/wineduck/tasting ~/.claude/skills/wineduck-tasting
cp -r asterduck_skillset/claude/wineduck/wine ~/.claude/skills/wineduck-wine
cp -r asterduck_skillset/claude/wineduck/cellar ~/.claude/skills/wineduck-cellar
```

## 사용 예시

### 와인 검색
> "부르고뉴 피노 누아 검색해줘"
> "바롤로 2019년산 있어?"
> "프랑스에 어떤 와인 지역이 있어?"

### 테이스팅 노트 등록 (자연어)
> "어제 Gevrey-Chambertin 마셨는데, 산미 강하고 타닌 부드럽고 바디 미디엄. 체리랑 오크 향. 4점. 스테이크랑 먹었어"

### 와인 등록
> "Pouilly-Fuissé 1er Cru 2021 등록해줘. Roger Lassarat 도멘이야. 샤르도네."

### 셀러 관리
> (라벨 사진 첨부) "이거 셀러에 추가해줘. 어제 8만원에 샀어"
> "내 셀러에 보유 중인 레드 와인 보여줘"
> "이번 달 안에 마셔야 할 와인 뭐 있어?"
> "#23 Wachau Riesling 다 마셨어"

## 스킬 간 연동

```
auth (공용) ────────────────────────┐
  │                                  │
  │ (토큰 발급)                       │
  ▼                                  ▼
search ──► wine ──► tasting ──► cellar
  │         │         │            │
  │ (검색)   │ (등록)   │ (기록)      │ (보유/소비)
  ▼         ▼         ▼            ▼
  WineDuck Platform API
```

## API

- **Base URL**: `https://coffeeduckbe-production.up.railway.app/api`
- **인증**: JWT Bearer Token (24시간 유효)
- **응답 형식**: JSON

## License

MIT

---

Built by [Aster.duck](https://asterduck.koalstudio.com) team
