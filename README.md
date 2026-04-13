# Aster.duck AI Skillset

[Aster.duck](https://asterduck.koalstudio.com) 플랫폼을 AI 에이전트로 활용하기 위한 스킬 & API 스펙 모음입니다.

커피와 와인 테이스팅 노트를 AI를 통해 검색, 등록, 관리할 수 있습니다.

## 지원 AI

| AI | 방식 | 폴더 |
|----|------|------|
| **Claude Code** | SKILL.md 스킬 설치 | [`claude/`](./claude/) |
| **ChatGPT (GPTs)** | OpenAPI Custom Actions | [`openapi/`](./openapi/) |
| **Gemini (Gems)** | OpenAPI 참조 | [`openapi/`](./openapi/) |
| **Cursor / Copilot** | OpenAPI 자동 인식 | [`openapi/`](./openapi/) |

## Available Skillsets

| 서비스 | 설명 | 스킬 수 |
|--------|------|---------|
| [Auth](./claude/auth/) | 공용 인증 (로그인, 회원가입, 토큰) | 1 |
| [CoffeeDuck](./claude/coffeeduck/) | 커피 검색, 등록, 테이스팅 노트 | 3 |
| [WineDuck](./claude/wineduck/) | 와인 검색, 등록, 테이스팅 노트 | 3 |
| [OpenAPI](./openapi/) | 전체 API 스펙 (GPTs/Gemini/범용) | 1 |

## Quick Start

### Claude Code — 스킬 설치

```bash
git clone https://github.com/ChoiKoal/asterduck_skillset.git

# 공용 인증
cp -r asterduck_skillset/claude/auth ~/.claude/skills/asterduck-auth

# CoffeeDuck
cp -r asterduck_skillset/claude/coffeeduck/search ~/.claude/skills/coffeeduck-search
cp -r asterduck_skillset/claude/coffeeduck/tasting ~/.claude/skills/coffeeduck-tasting
cp -r asterduck_skillset/claude/coffeeduck/coffee ~/.claude/skills/coffeeduck-coffee

# WineDuck
cp -r asterduck_skillset/claude/wineduck/search ~/.claude/skills/wineduck-search
cp -r asterduck_skillset/claude/wineduck/tasting ~/.claude/skills/wineduck-tasting
cp -r asterduck_skillset/claude/wineduck/wine ~/.claude/skills/wineduck-wine
```

### ChatGPT (GPTs) — Custom Actions

1. GPTs 편집 → Actions → Create new action
2. `openapi/asterduck-api.yaml` 내용 붙여넣기
3. Authentication: API Key, Header `Authorization: Bearer {token}`
4. 상세 가이드: [`openapi/README.md`](./openapi/README.md)

### Gemini / 기타 AI

`openapi/asterduck-api.yaml` 스펙을 참조하여 API 호출 설정.
상세 가이드: [`openapi/README.md`](./openapi/README.md)

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
├── claude/                # Claude Code 스킬
│   ├── auth/              # 공용 인증
│   │   └── SKILL.md
│   ├── coffeeduck/        # 커피덕 스킬셋
│   │   ├── README.md
│   │   ├── search/SKILL.md
│   │   ├── tasting/SKILL.md
│   │   └── coffee/SKILL.md
│   └── wineduck/          # 와인덕 스킬셋
│       ├── README.md
│       ├── search/SKILL.md
│       ├── tasting/SKILL.md
│       └── wine/SKILL.md
├── openapi/               # OpenAPI 스펙 (GPTs/Gemini/범용)
│   ├── asterduck-api.yaml
│   └── README.md
└── README.md
```

## API

- **Base URL**: `https://coffeeduckbe-production.up.railway.app/api`
- **인증**: JWT Bearer Token (24시간 유효)
- **응답 형식**: JSON
- **다국어**: `Accept-Language` 헤더 지원 (ko, ja, en)
- **총 엔드포인트**: 31개

## 서비스

- **Web**: [asterduck.koalstudio.com](https://asterduck.koalstudio.com)
- **GitHub**: [ChoiKoal/project_coffeeduck](https://github.com/ChoiKoal/project_coffeeduck)

## License

MIT

---

Built by [Aster.duck](https://asterduck.koalstudio.com) team
