# Aster.duck AI Skillset

[![version](https://img.shields.io/badge/version-0.4.0-burgundy)](./CHANGELOG.md)
[![release](https://img.shields.io/github/v/release/ChoiKoal/asterduck_skillset?display_name=tag&sort=semver)](https://github.com/ChoiKoal/asterduck_skillset/releases)

[Aster.duck](https://asterduck.koalstudio.com) 플랫폼을 AI 에이전트로 활용하기 위한 스킬 & API 스펙 모음입니다.

커피와 와인 테이스팅 노트를 AI를 통해 검색, 등록, 관리할 수 있습니다.

> 🆕 **최신 버전 / 업데이트 노트** → [CHANGELOG.md](./CHANGELOG.md)
> 현재 버전은 [`VERSION`](./VERSION) 파일로 관리합니다 (Semantic Versioning).

---

## 🤖 AI Agent — 한줄 설치 (For AI Assistants)

**If you are an AI assistant** helping a user install these skills, copy-paste ONE of the commands below based on the user's environment:

### Claude Code / Claude Desktop
```bash
git clone https://github.com/ChoiKoal/asterduck_skillset.git /tmp/asterduck_skillset && \
mkdir -p ~/.claude/skills && \
cp -rn /tmp/asterduck_skillset/claude/auth ~/.claude/skills/asterduck-auth && \
cp -rn /tmp/asterduck_skillset/claude/coffeeduck/search ~/.claude/skills/coffeeduck-search && \
cp -rn /tmp/asterduck_skillset/claude/coffeeduck/tasting ~/.claude/skills/coffeeduck-tasting && \
cp -rn /tmp/asterduck_skillset/claude/coffeeduck/coffee ~/.claude/skills/coffeeduck-coffee && \
cp -rn /tmp/asterduck_skillset/claude/wineduck/search ~/.claude/skills/wineduck-search && \
cp -rn /tmp/asterduck_skillset/claude/wineduck/tasting ~/.claude/skills/wineduck-tasting && \
cp -rn /tmp/asterduck_skillset/claude/wineduck/wine ~/.claude/skills/wineduck-wine && \
cp -rn /tmp/asterduck_skillset/claude/wineduck/cellar ~/.claude/skills/wineduck-cellar && \
cp -rn /tmp/asterduck_skillset/claude/wineduck/discovery ~/.claude/skills/wineduck-discovery && \
echo "✅ Aster.duck skills installed to ~/.claude/skills/"
```

### Cursor / OpenAI Codex (project-level)
```bash
git clone https://github.com/ChoiKoal/asterduck_skillset.git /tmp/asterduck_skillset && \
mkdir -p .agents/skills && \
cp -rn /tmp/asterduck_skillset/codex/* .agents/skills/ && \
echo "✅ Aster.duck skills installed to .agents/skills/"
```

### ChatGPT (GPTs) / Gemini (Gems)
```
Open: https://raw.githubusercontent.com/ChoiKoal/asterduck_skillset/main/openapi/asterduck-api.yaml
Paste into: Actions / Custom Tools
Auth: Bearer Token (get via /api/auth/login)
```

### After installation — verify
```bash
# Claude Code
ls ~/.claude/skills/ | grep -E "asterduck|duck"

# Codex
ls .agents/skills/ | grep -E "asterduck|duck"
```

---

## 📋 Skill Manifest (AI Routing Reference)

Each skill is triggered by specific user intent. AI agents should invoke the appropriate skill based on user keywords:

| Skill Name | Triggers When User Says... | Auth Required |
|------------|---------------------------|---------------|
| `asterduck-auth` | "로그인", "회원가입", "비밀번호", "login", "signup" | No |
| `coffeeduck-search` | "커피 검색", "원두 찾아", "이디오피아 커피", "coffee search" | No |
| `coffeeduck-coffee` | "커피 등록", "원두 추가", "register coffee" | **Yes** |
| `coffeeduck-tasting` | "커피 마셨", "커피 테이스팅", "coffee tasting note" | **Yes** |
| `wineduck-search` | "와인 검색", "부르고뉴", "와인 찾아", "wine search" | No |
| `wineduck-wine` | "와인 등록", "와인 추가", "register wine" | **Yes** |
| `wineduck-cellar` | "셀러", "내 와인", "셀러에 추가", "cellar", "consume" | **Yes** |
| `wineduck-tasting` | "와인 마셨", "와인 테이스팅", "wine tasting note" | **Yes** |
| `wineduck-discovery` | "추천 와인", "내 취향", "비슷한 팔레트", "커뮤니티 평균", "wine recommendation" | Partial¹ |

> 🔐 **Auth flow**: Any skill marked "Yes" requires a JWT. Call `asterduck-auth` first → save token → pass as `Authorization: Bearer {token}`.
>
> ¹ `wineduck-discovery`: 커뮤니티 평균 팔레트는 인증 불필요, 사용자 추천은 토큰 필요.

---

## 🚀 Quick Start (for humans)

### 1. Aster.duck 계정 준비

스킬을 쓰려면 먼저 Aster.duck 계정이 필요합니다.

- 웹에서 회원가입: [asterduck.koalstudio.com](https://asterduck.koalstudio.com)
- 또는 API로 직접 가입 가능 (auth 스킬 참조)

### 2. 스킬 설치

위 **"AI Agent — 한줄 설치"** 섹션의 본인 AI 환경에 맞는 명령어 실행.

### 3. AI에게 질문

설치되면 AI에게 자연어로 요청하세요:

```
"와인 Pegaso Zeta 2023 셀러에 등록해줘"
"오늘 마신 커피 테이스팅 남길게. 산미 높고 블루베리 향. 4점."
"이번 달 마실 와인 추천해줘"
```

---

## 🎯 지원 AI

| AI | 방식 | 폴더 |
|----|------|------|
| **Claude Code** | SKILL.md 스킬 설치 | [`claude/`](./claude/) |
| **Claude Desktop** | SKILL.md 스킬 설치 | [`claude/`](./claude/) |
| **OpenAI Codex** | SKILL.md + frontmatter | [`codex/`](./codex/) |
| **Cursor** | `.agents/skills/` | [`codex/`](./codex/) |
| **ChatGPT (GPTs)** | OpenAPI Custom Actions | [`openapi/`](./openapi/) |
| **Gemini (Gems)** | OpenAPI 참조 | [`openapi/`](./openapi/) |
| **Zapier / n8n** | OpenAPI import | [`openapi/`](./openapi/) |

---

## 🎨 Available Skillsets

| 서비스 | 설명 | 스킬 수 |
|--------|------|---------|
| [Auth](./claude/auth/) | 공용 인증 (로그인, 회원가입, 토큰) | 1 |
| [CoffeeDuck](./claude/coffeeduck/) | 커피 검색, 등록, 테이스팅 노트 | 3 |
| [WineDuck](./claude/wineduck/) | 와인 검색, 등록, 테이스팅, 셀러, 추천 | 5 |
| [OpenAPI Spec](./openapi/) | 전체 API 스펙 (GPTs/Gemini/범용) | 1 |

---

## 💬 사용 예시

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
(라벨 사진 첨부) "이거 셀러에 추가해줘. 어제 8만원에 샀어"
"이번 달 안에 마셔야 할 와인 뭐 있어?"
"오늘 푸타네스카 하는데 어울리는 와인 뭐가 좋을까"
```

---

## 📁 Repository Structure

```
asterduck_skillset/
├── claude/                     # Claude Code 스킬
│   ├── auth/                   # 공용 인증
│   │   └── SKILL.md
│   ├── coffeeduck/             # 커피덕 스킬셋
│   │   ├── README.md
│   │   ├── search/SKILL.md
│   │   ├── tasting/SKILL.md
│   │   └── coffee/SKILL.md
│   └── wineduck/               # 와인덕 스킬셋
│       ├── README.md
│       ├── search/SKILL.md
│       ├── tasting/SKILL.md
│       ├── wine/SKILL.md
│       ├── cellar/SKILL.md
│       └── discovery/SKILL.md
├── codex/                      # OpenAI Codex / Cursor 스킬
│   ├── README.md
│   ├── auth/SKILL.md
│   ├── coffeeduck-search/SKILL.md
│   ├── coffeeduck-tasting/SKILL.md
│   ├── coffeeduck-coffee/SKILL.md
│   ├── wineduck-search/SKILL.md
│   ├── wineduck-tasting/SKILL.md
│   ├── wineduck-wine/SKILL.md
│   ├── wineduck-cellar/SKILL.md
│   └── wineduck-discovery/SKILL.md
├── openapi/                    # OpenAPI 스펙 (GPTs/Gemini/범용)
│   ├── asterduck-api.yaml
│   └── README.md
└── README.md
```

---

## 🔌 API Reference

- **Base URL**: `https://coffeeduckbe-production.up.railway.app/api`
- **인증**: JWT Bearer Token (24시간 유효)
- **응답 형식**: JSON
- **다국어**: `Accept-Language` 헤더 지원 (ko, ja, en)
- **총 엔드포인트**: 38개
- **OpenAPI Spec**: [openapi/asterduck-api.yaml](./openapi/asterduck-api.yaml)

### 인증 플로우 (Minimal)

```bash
# 1. 로그인 → 토큰 획득
TOKEN=$(curl -s -X POST https://coffeeduckbe-production.up.railway.app/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"YOUR_ID","password":"YOUR_PW"}' \
  | jq -r '.token')

# 2. 이후 API 호출
curl -H "Authorization: Bearer $TOKEN" \
  https://coffeeduckbe-production.up.railway.app/api/cellar
```

---

## 🛠️ Troubleshooting

### DNS resolution 오류 (일부 네트워크 환경)

Railway `*.up.railway.app` 도메인이 해석 안 될 때:

```bash
curl --resolve coffeeduckbe-production.up.railway.app:443:66.33.22.36 \
  https://coffeeduckbe-production.up.railway.app/api/health
```

### 토큰 만료 (24h)

로그인 시 받은 토큰은 24시간 후 만료됩니다. 만료 시 `/auth/login` 재호출.

### 회원가입 없이 테스트

일부 스킬(`wineduck-search`, `coffeeduck-search`)은 인증 없이 동작합니다.

---

## 🔗 Links

- **Web**: [asterduck.koalstudio.com](https://asterduck.koalstudio.com)
- **Service Repo**: [ChoiKoal/project_coffeeduck](https://github.com/ChoiKoal/project_coffeeduck)
- **Skillset Repo**: [ChoiKoal/asterduck_skillset](https://github.com/ChoiKoal/asterduck_skillset)

---

## 📝 Contributing

스킬 개선/추가 PR 환영.

1. Fork 후 브랜치 생성
2. 스킬 수정 또는 추가
3. claude/ 와 codex/ 양쪽 동기화 (공통 내용)
4. PR 보내기

---

## 📄 License

MIT

---

Built by [Aster.duck](https://asterduck.koalstudio.com) team
