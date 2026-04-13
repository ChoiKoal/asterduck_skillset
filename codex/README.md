# Aster.duck Codex Skills

OpenAI Codex용 Aster.duck 스킬셋입니다.

## 설치

### 프로젝트 레벨 (권장)

```bash
git clone https://github.com/ChoiKoal/asterduck_skillset.git

# 프로젝트 루트에 .agents/skills 디렉토리 생성
mkdir -p .agents/skills

# 원하는 스킬 복사
cp -r asterduck_skillset/codex/auth .agents/skills/asterduck-auth
cp -r asterduck_skillset/codex/coffeeduck-search .agents/skills/
cp -r asterduck_skillset/codex/coffeeduck-tasting .agents/skills/
cp -r asterduck_skillset/codex/coffeeduck-coffee .agents/skills/
cp -r asterduck_skillset/codex/wineduck-search .agents/skills/
cp -r asterduck_skillset/codex/wineduck-tasting .agents/skills/
cp -r asterduck_skillset/codex/wineduck-wine .agents/skills/
```

### 사용자 레벨 (모든 프로젝트에서 사용)

```bash
mkdir -p ~/.agents/skills

cp -r asterduck_skillset/codex/auth ~/.agents/skills/asterduck-auth
# ... (위와 동일하게 복사)
```

## 스킬 목록

| 스킬 | 설명 | 인증 |
|------|------|------|
| `auth` | 공용 인증 (로그인, 회원가입, 토큰) | - |
| `coffeeduck-search` | 커피 검색, 플레이버 노트, 품종 | 불필요 |
| `coffeeduck-tasting` | 커피 테이스팅 노트 CRUD | 필요 |
| `coffeeduck-coffee` | 커피 등록, 중복 검사 | 필요 |
| `wineduck-search` | 와인 검색, 지역/아펠라시옹 탐색 | 불필요 |
| `wineduck-tasting` | 와인 테이스팅 노트 CRUD | 필요 |
| `wineduck-wine` | 와인 등록, 명명 규칙 | 필요 |

## Claude Code 스킬과의 차이

- **포맷**: YAML frontmatter (`name`, `description`) 추가
- **설치 경로**: `.agents/skills/` (Claude는 `.claude/skills/`)
- **내용**: API 엔드포인트, 파라미터 등 동일

## 호출 방법

```
# 명시적 호출 (Codex CLI)
$ coffeeduck-search "이디오피아 예가체프 검색해줘"

# 암묵적 호출 — description 매칭으로 자동 트리거
"커피 검색해줘" → coffeeduck-search 자동 선택
```
