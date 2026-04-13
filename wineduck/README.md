# WineDuck AI Skills

> AI를 통해 [WineDuck](https://asterduck.koalstudio.com) 와인 테이스팅 플랫폼을 활용하세요.

WineDuck은 와인 덕후를 위한 테이스팅 노트 앱입니다. 이 스킬셋을 사용하면 AI 에이전트를 통해 와인 검색, 등록, 테이스팅 노트 작성을 대화형으로 할 수 있습니다.

## Skills

| 스킬 | 설명 | 인증 |
|------|------|------|
| [auth](./auth/SKILL.md) | 로그인, 회원가입, 토큰 관리 | - |
| [search](./search/SKILL.md) | 와인 검색, 지역/아펠라시옹 탐색 | 불필요 |
| [tasting](./tasting/SKILL.md) | 테이스팅 노트 등록/조회/수정/삭제 | 필요 |
| [wine](./wine/SKILL.md) | 와인 등록/수정/삭제, 중복 검사, 명명 규칙 | 필요 |

## 설치

### Claude Code (`.claude/skills/` 디렉토리)

```bash
# 레포 클론
git clone https://github.com/ChoiKoal/asterduck_skillset.git

# 원하는 스킬을 .claude/skills/ 에 복사
cp -r asterduck_skillset/wineduck/auth ~/.claude/skills/wineduck-auth
cp -r asterduck_skillset/wineduck/search ~/.claude/skills/wineduck-search
cp -r asterduck_skillset/wineduck/tasting ~/.claude/skills/wineduck-tasting
cp -r asterduck_skillset/wineduck/wine ~/.claude/skills/wineduck-wine
```

### 전체 설치 (한 번에)

```bash
git clone https://github.com/ChoiKoal/asterduck_skillset.git
for skill in auth search tasting wine; do
  cp -r asterduck_skillset/wineduck/$skill ~/.claude/skills/wineduck-$skill
done
```

## 사용 예시

### 와인 검색
> "부르고뉴 피노 누아 검색해줘"
> "바롤로 2019년산 있어?"
> "프랑스에 어떤 와인 지역이 있어?"

### 테이스팅 노트 등록 (자연어)
> "어제 Gevrey-Chambertin 마셨는데, 산미 강하고 타닌 부드럽고 바디 미디엄. 체리랑 오크 향. 4점. 스테이크랑 먹었어"

### 테이스팅 노트 등록 (구조화)
```
와인: Gevrey-Chambertin 2023
당도: 1  산도: 4  바디: 3  타닌: 3  여운: 4
총점: 4.0  재구매: ㅇ
향: 체리, 오크, 바닐라
한줄평: 우아한 피노누아
```

### 와인 등록
> "Pouilly-Fuissé 1er Cru 2021 등록해줘. Roger Lassarat 도멘이야. 샤르도네."

## 스킬 간 연동

```
auth ──────────────────────────────┐
  │                                 │
  │ (토큰 발급)                      │
  ▼                                 ▼
search ──► wine ──► tasting
  │         │         │
  │ (검색)   │ (등록)   │ (기록)
  ▼         ▼         ▼
  WineDuck Platform API
```

- **auth** → tasting, wine의 인증 기반
- **search** → wine 등록 전 중복 확인, tasting 시 와인 탐색
- **wine** → 새 와인 등록 (search로 중복 확인 후)
- **tasting** → 테이스팅 노트 작성 (search로 와인 찾고)

## API

모든 스킬은 WineDuck 공개 REST API를 사용합니다.

- **Base URL**: `https://coffeeduckbe-production.up.railway.app/api`
- **인증**: JWT Bearer Token (24시간 유효)
- **응답 형식**: JSON

## 서비스

- **WineDuck Web**: [asterduck.koalstudio.com](https://asterduck.koalstudio.com)
- **GitHub**: [ChoiKoal/project_coffeeduck](https://github.com/ChoiKoal/project_coffeeduck)

## License

MIT

---

Built by [Aster.duck](https://asterduck.koalstudio.com) team
