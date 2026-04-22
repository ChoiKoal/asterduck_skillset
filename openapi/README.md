# Aster.duck OpenAPI Spec

OpenAPI 3.1 스펙으로 정의된 Aster.duck API입니다.
GPT, Gemini, Copilot 등 모든 AI 에이전트에서 활용할 수 있습니다.

## 파일

- `asterduck-api.yaml` — 전체 API 스펙 (Auth + CoffeeDuck + WineDuck)

## 활용 방법

### ChatGPT (GPTs Custom Actions)

1. [GPTs 생성 페이지](https://chat.openai.com/gpts/editor)에서 새 GPT 생성
2. **Configure** → **Actions** → **Create new action**
3. **Schema** 필드에 `asterduck-api.yaml` 내용 붙여넣기
4. **Authentication** → `API Key` → Header Name: `Authorization`, Value: `Bearer {your_token}`
5. Instructions에 원하는 시나리오 추가:
   ```
   You are a coffee and wine tasting assistant.
   Help users search for coffee/wine, record tasting notes, and manage their collection.
   Always search before registering to avoid duplicates.
   ```

### Google Gemini (Gems)

1. [Gemini Gems](https://gemini.google.com/gems)에서 새 Gem 생성
2. Instructions에 API 사용법 설명 추가
3. OpenAPI 스펙의 endpoint URL을 참조하여 function calling 설정
4. 인증이 필요한 API는 사용자에게 로그인 토큰 요청

### Cursor / Copilot

1. 프로젝트에 `asterduck-api.yaml` 포함
2. AI가 자동으로 API 구조를 인식하여 코드 생성에 활용

### Swagger UI로 미리보기

```bash
# Docker
docker run -p 8080:8080 -e SWAGGER_JSON=/api/asterduck-api.yaml -v $(pwd):/api swaggerapi/swagger-ui

# 또는 온라인: https://editor.swagger.io 에 yaml 붙여넣기
```

## API 요약

| 카테고리 | 엔드포인트 수 | 인증 |
|---------|-------------|------|
| Auth | 4 | 일부 |
| CoffeeDuck Search | 7 | 불필요 |
| CoffeeDuck Tasting | 6 | 필요 |
| CoffeeDuck Coffee | 1 | 필요 |
| WineDuck Search | 7 | 불필요 |
| WineDuck Tasting | 5 | 필요 |
| WineDuck Wine | 1 | 필요 |
| WineDuck Cellar | 7 | 필요 |
| **합계** | **38** | |

## CoffeeDuck vs WineDuck 차이점

| 항목 | CoffeeDuck | WineDuck |
|------|-----------|----------|
| 팔레트 범위 | 0-100 | 1-5 |
| 팔레트 축 | acidity, body, sweetness, bitterness, finish | sweetness, acidity, body, tannin, finish |
| 향미 입력 | flavor_note_ids (마스터 테이블 ID) | aromas (custom label or taxonomy node_id) |
| 추가 필드 | roasting_level, brew_method | aroma_intensity, color, pairing_food, repurchase |

## Base URL

```
https://coffeeduckbe-production.up.railway.app/api
```

## 서비스

- **Web**: [asterduck.koalstudio.com](https://asterduck.koalstudio.com)
- **GitHub**: [ChoiKoal/project_coffeeduck](https://github.com/ChoiKoal/project_coffeeduck)
