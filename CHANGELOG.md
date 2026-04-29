# Changelog

Aster.duck AI Skillset의 모든 주요 변경 사항을 기록합니다.

이 프로젝트는 [Semantic Versioning](https://semver.org/lang/ko/)을 따릅니다.

- **MAJOR** — 기존 스킬의 트리거/엔드포인트/응답 구조가 호환 불가능하게 바뀐 경우
- **MINOR** — 새 스킬 추가, 새 엔드포인트/필드 추가 (기존 흐름과 하위 호환)
- **PATCH** — 문서 보강, 오타 수정, 가이드 명확화

태그: `git tag v{x.y.z}` / GitHub Release로 같이 발행합니다.

---

## [0.4.0] - 2026-04-29

### Added

- **wineduck-discovery**: 신규 스킬. 커뮤니티 평균 팔레트 + 사용자 취향 기반 와인 추천
  - `GET /wineduck/wines/aggregates/palate-avg` — 5축 평균 팔레트 (인증 불필요, FE 레이더 차트 오버레이용)
  - `GET /wineduck/users/{user_id}/recommendations` — 팔레트 유사도 기반 추천 TOP N (인증 필요, 본인만)
  - 게이팅(`need_more` / `min_count_required`), 스코어링(팔레트 + 타입/국가/품종 보너스), `reasons` 태그 정책 명시
- **openapi**: `WineDuck - Discovery` 태그 + 위 두 엔드포인트 스키마 추가
- **openapi**: 0.3.0에서 cellar SKILL.md에만 반영됐던 변경을 spec에도 동기화
  - `/cellar` GET status 콤마 다중값 (string + description)
  - `/cellar` GET sort enum에 `drink_soon`, `consumed_recent` 추가
  - `CellarEntry` / `CellarCreateRequest` / `CellarUpdateRequest` 에 `drink_from` / `drink_until` 추가
  - `GET /cellar/expiring` 신규 path

---

## [0.3.0] - 2026-04-29

### Added

- **wineduck-cellar**: `drink_soon`, `consumed_recent` 정렬 옵션 명시 (BE는 진작 지원, 문서만 누락)
- **wineduck-cellar**: `status` 콤마 다중값(`in_cellar,received` 등) 지원 가이드
- **wineduck-cellar**: `drink_from` / `drink_until` 응답 · POST · PUT 필드 추가
- **wineduck-cellar**: `GET /api/cellar/expiring` 엔드포인트 추가 (홈 "곧 마실 와인" 카드용)
- **wineduck-cellar**: `purchase_price` 노출 정책 명시 — BE는 반환하나 서비스 UI는 의도적으로 숨김. 통계(`/cellar/stats.total_value`)에만 활용
- **wineduck-wine**: "기존 region/appellation ID 우선 매핑" 강제 가이드 추가 — 표기 흔들림으로 인한 중복 등록 방지

### Changed

- **wineduck-search**: 운영(Admin) DELETE 엔드포인트 섹션 제거 (일반 스킬 사용자에게 노출되어선 안 됨). admin 작업은 운영자가 직접 API 호출
- **wineduck-wine**: 매핑 규칙 4단계 절차로 재구조화 (countries → regions(+include_sub) → appellations → 신규 추가는 사용자 확인 후)

---

## [0.2.0] - 2026-04 이전

### Added

- **wineduck-tasting**: Phase 2 취향 차트 엔드포인트 반영
- **wineduck-tasting**: 대고객 스킬 톤으로 정리 (내부 팀원 레퍼런스 제거)
- **wineduck-cellar**: 보유 와인 관리 + 사진 기반 셀러 추가 스킬
- **codex/**: OpenAI Codex 스킬 추가
- **openapi/**: 통합 OpenAPI 스펙
- **coffeeduck**: 커피 / 검색 / 테이스팅 스킬셋
- **shared**: Auth 공용 스킬 분리

### Changed

- README를 AI 에이전트 한줄 설치용으로 보강
- `drink_window` 유효값 / `white_sparkling` 카테고리 경고 등 문서 보강

### Removed

- 관리자 전용 와인 수정/삭제 API 문서 (대고객 스킬에서 제거)

---

## [0.1.0] - 초기 릴리즈

### Added

- **wineduck**: auth · search · tasting · wine 4종 스킬 초기 작성
