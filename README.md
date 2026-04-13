# Aster.duck AI Skillset

[Aster.duck](https://asterduck.koalstudio.com) 플랫폼을 AI 에이전트로 활용하기 위한 스킬 모음입니다.

커피와 와인 테이스팅 노트를 AI를 통해 검색, 등록, 관리할 수 있습니다.

## Available Skillsets

| 서비스 | 설명 | 스킬 수 |
|--------|------|---------|
| [Auth](./auth/) | 공용 인증 (로그인, 회원가입, 토큰) | 1 |
| [CoffeeDuck](./coffeeduck/) | 커피 검색, 등록, 테이스팅 노트 | 3 |
| [WineDuck](./wineduck/) | 와인 검색, 등록, 테이스팅 노트 | 3 |

## Quick Start

### 전체 설치 (CoffeeDuck + WineDuck)

```bash
git clone https://github.com/ChoiKoal/asterduck_skillset.git

# 공용 인증
cp -r asterduck_skillset/auth ~/.claude/skills/asterduck-auth

# CoffeeDuck
cp -r asterduck_skillset/coffeeduck/search ~/.claude/skills/coffeeduck-search
cp -r asterduck_skillset/coffeeduck/tasting ~/.claude/skills/coffeeduck-tasting
cp -r asterduck_skillset/coffeeduck/coffee ~/.claude/skills/coffeeduck-coffee

# WineDuck
cp -r asterduck_skillset/wineduck/search ~/.claude/skills/wineduck-search
cp -r asterduck_skillset/wineduck/tasting ~/.claude/skills/wineduck-tasting
cp -r asterduck_skillset/wineduck/wine ~/.claude/skills/wineduck-wine
```

### 서비스별 설치

커피덕만 사용하려면:
```bash
cp -r asterduck_skillset/auth ~/.claude/skills/asterduck-auth
for skill in search tasting coffee; do
  cp -r asterduck_skillset/coffeeduck/$skill ~/.claude/skills/coffeeduck-$skill
done
```

와인덕만 사용하려면:
```bash
cp -r asterduck_skillset/auth ~/.claude/skills/asterduck-auth
for skill in search tasting wine; do
  cp -r asterduck_skillset/wineduck/$skill ~/.claude/skills/wineduck-$skill
done
```

## 사용 예시

### ☕ CoffeeDuck
```
"이디오피아 예가체프 커피 검색해줘"
"프리츠에서 산 이디오피아 커피 등록해줘. 워시드, 에어룸 품종."
"오늘 마신 커피 테이스팅 남길게. 산미 밝고 바디 가벼워. 블루베리 향. 4점."
```

### 🍷 WineDuck
```
"부르고뉴 피노 누아 검색해줘"
"Gevrey-Chambertin Vieilles Vignes 2023 등록해줘. 쟝테팡시오 도멘."
"어제 마신 와인 기록할게. 산미 강하고 타닌 부드러워. 체리랑 오크 향. 4점."
```

## 구조

```
asterduck_skillset/
├── auth/                  # 공용 인증 스킬
│   └── SKILL.md
├── coffeeduck/            # 커피덕 스킬셋
│   ├── README.md
│   ├── search/SKILL.md    # 커피 검색, 플레이버 노트, 품종
│   ├── tasting/SKILL.md   # 테이스팅 노트 CRUD
│   └── coffee/SKILL.md    # 커피 등록
└── wineduck/              # 와인덕 스킬셋
    ├── README.md
    ├── search/SKILL.md    # 와인 검색, 지역/아펠라시옹
    ├── tasting/SKILL.md   # 테이스팅 노트 CRUD
    └── wine/SKILL.md      # 와인 등록
```

## API

- **Base URL**: `https://coffeeduckbe-production.up.railway.app/api`
- **인증**: JWT Bearer Token (24시간 유효)
- **응답 형식**: JSON
- **다국어**: `Accept-Language` 헤더 지원 (ko, ja, en)

## 서비스

- **Web**: [asterduck.koalstudio.com](https://asterduck.koalstudio.com)
- **GitHub**: [ChoiKoal/project_coffeeduck](https://github.com/ChoiKoal/project_coffeeduck)

## License

MIT

---

Built by [Aster.duck](https://asterduck.koalstudio.com) team
