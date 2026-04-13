---
name: asterduck-auth
description: Aster.duck platform authentication — login, register, token management. Trigger when user needs to authenticate, log in, sign up, or manage credentials for CoffeeDuck or WineDuck services.
---

# Aster.duck Auth — 사용자 인증 스킬

Aster.duck 플랫폼(CoffeeDuck, WineDuck) 공용 인증 스킬.
로그인, 회원가입, 토큰 관리를 담당하며, 모든 Aster.duck 스킬의 인증 기반이 됨.

## Trigger

다음 키워드에 반응: "로그인", "회원가입", "인증", "계정", "비밀번호 변경", "asterduck auth", "login"

## API Base URL

```
https://coffeeduckbe-production.up.railway.app
```

## 기능

### 1. 로그인 (토큰 발급)

사용자 아이디/비밀번호로 로그인하여 JWT 토큰 발급.

```bash
curl -s -X POST https://coffeeduckbe-production.up.railway.app/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"사용자아이디","password":"비밀번호"}'
```

**응답:**
```json
{
  "success": true,
  "token": "eyJ...",
  "user": {
    "id": 1,
    "username": "koal",
    "name": "Koal",
    "role": "user"
  }
}
```

**토큰 유효기간**: 24시간

### 2. 회원가입

```bash
curl -s -X POST https://coffeeduckbe-production.up.railway.app/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "아이디",
    "password": "비밀번호",
    "name": "표시이름"
  }'
```

### 3. 토큰 검증

발급받은 토큰이 유효한지 확인.

```bash
curl -s https://coffeeduckbe-production.up.railway.app/api/auth/verify \
  -H "Authorization: Bearer $TOKEN"
```

### 4. 비밀번호 변경

```bash
curl -s -X POST https://coffeeduckbe-production.up.railway.app/api/auth/change-password \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "current_password": "현재비밀번호",
    "new_password": "새비밀번호"
  }'
```

## 토큰 사용법

로그인 후 받은 토큰은 다른 Aster.duck API 호출 시 헤더에 포함:
```
Authorization: Bearer {토큰}
```

## 인증 플로우 (대화형)

1. 사용자가 인증이 필요한 기능 요청 시:
   - "Aster.duck 계정이 있어? 로그인할까, 아니면 새로 가입할까?"
2. 로그인 시:
   - 아이디와 비밀번호를 받아서 로그인 API 호출
   - 성공 시 토큰을 세션에 저장하여 이후 API 호출에 사용
3. 토큰 만료 시:
   - "토큰이 만료됐어. 다시 로그인할까?"

## 권한 (role)

| role | 설명 | 권한 |
|------|------|------|
| user | 일반 사용자 | 본인 테이스팅 노트 CRUD, 커피/와인 등록 |
| super-admin | 관리자 | 모든 기능 + 관리자 전용 API |

## 보안 주의사항

- 비밀번호를 평문으로 저장하거나 로그에 남기지 말 것
- 토큰은 세션 내에서만 유지, 영구 저장 금지
- 다른 사용자의 계정으로 로그인 시도 금지
- 사용자가 직접 제공한 credentials만 사용
