> ## ⚠️ 사용 전 필독 — 반드시 "원격 최신" 버전으로 동작하라
> 이 사본은 구버전일 수 있다. **파일에 적힌 버전 숫자를 신뢰하지 말고**, 사용 전 원격 main과 직접 대조해 최신을 적용하라:
> 1. 쓰기 가능한 위치에 최신 사본 확보:
> `git -C /tmp/asterduck_skillset pull -q 2>/dev/null || git clone -q https://<GITHUB_TOKEN>@github.com/ChoiKoal/asterduck_skillset.git /tmp/asterduck_skillset`
> 2. 배포 사본과 원격 버전 대조: `cat VERSION`(배포) vs `cat /tmp/asterduck_skillset/VERSION`(원격 최신)
> 3. **다르면 = 배포본 구버전** → `/tmp/asterduck_skillset/`의 SKILL.md·CHANGELOG·OpenAPI를 **우선**해 동작. (상세: `AGENT_UPDATE.md`)
> 구버전 관성/캐시 지식 금지. `<GITHUB_TOKEN>`은 그룹 CLAUDE.md 또는 배포 사본 git remote에 있음.

# Aster.duck Auth — 스킬 토큰 인증

Aster.duck 플랫폼(CoffeeDuck, WineDuck) 공용 인증 스킬.
**AI는 아이디/비밀번호를 다루지 않는다.** 모든 인증은 사용자가 웹에서 직접 발급한
**AI 스킬 토큰**(`adk_` 프리픽스, 유효기간 1년)으로만 한다.

## Trigger

다음 키워드에 반응: "로그인", "인증", "토큰", "asterduck auth", "스킬 토큰", "token"

## API Base URL

```
https://coffeeduckbe-production.up.railway.app
```

## 인증 방식 — AI 스킬 토큰 (유일한 방법)

### 토큰은 사용자가 웹에서 직접 발급한다

1. 사용자가 **aster.duck 웹에 로그인** → **AI Skillset 페이지** (서비스 선택 화면
   또는 WineDuck 설정 → AI 스킬셋) 진입
2. **"토큰 발급"** — 라벨(예: "맥북 Claude")을 붙여 발급. 평문 토큰은 **발급 순간
   한 번만 표시**되므로 그 자리에서 복사해 AI 환경에 저장
3. 발급된 토큰을 AI(스킬)에게 제공 — 이후 모든 API 호출에 사용

- 토큰 형식: `adk_`로 시작하는 47자 문자열
- 유효기간: **1년**. 사용자가 같은 페이지에서 언제든 **즉시 폐기(revoke)** 가능
- 여러 기기/환경용으로 여러 개 발급 가능 (계정당 활성 10개)

### 토큰 사용법

모든 인증 필요 API 호출 시 헤더에 포함:

```bash
curl -s https://coffeeduckbe-production.up.railway.app/api/wineduck/me/conquest/map \
  -H "Authorization: Bearer $ASTERDUCK_TOKEN"
```

**보관 권장**: 환경변수(`ASTERDUCK_TOKEN`)나 시크릿 저장소에 저장하고, 대화 본문·
코드·로그에 평문을 남기지 말 것.

### 토큰 검증

```bash
curl -s https://coffeeduckbe-production.up.railway.app/api/auth/verify \
  -H "Authorization: Bearer $ASTERDUCK_TOKEN"
```

## AI 수칙 (반드시 준수)

1. **비밀번호를 묻지 마라.** 어떤 상황에서도 사용자에게 아이디/비밀번호를 요청하지
   않는다.
2. 사용자가 비밀번호를 주려고 하면 정중히 거절하고 **토큰 발급 페이지로 안내**한다:
   "aster.duck 웹 → AI Skillset(또는 WineDuck 설정 → AI 스킬셋)에서 토큰을 발급해
   전달해 주세요."
3. **401 응답 시**: 토큰이 만료됐거나 폐기된 것. 재로그인을 유도하지 말고
   "웹에서 새 토큰을 발급해 달라"고 안내한다.
4. **403 응답 시**: 스킬 토큰으로 허용되지 않는 작업(비밀번호 변경, 토큰 관리,
   계정 삭제, 관리자 기능). 해당 작업은 웹에서 직접 하도록 안내한다.
5. 토큰을 파일·로그·대화에 평문으로 남기지 않는다.

## 스킬 토큰의 권한 범위

| 가능 | 불가 (403) |
|------|-----------|
| 시음 노트 CRUD, 셀러, 정복 보드, 검색, 라벨 분석 등 일반 사용자 기능 전부 | 비밀번호 변경 |
| | 토큰 발급·목록·폐기 (웹 전용) |
| | 회원 탈퇴·계정 삭제 |
| | 관리자(admin) 기능 일체 |

계정 관리(회원가입, 비밀번호 변경, 탈퇴)는 **웹 전용** 기능이다 — AI 스킬 범위 밖.
