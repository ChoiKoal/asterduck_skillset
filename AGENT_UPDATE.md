# AGENT_UPDATE — 스킬 사용 전 "원격 최신" 강제 절차

> 에이전트는 이 스킬셋을 쓰기 전에 **반드시 원격(main)과 대조해 최신인지 확인**하고,
> 구버전이면 최신을 받아 그 내용으로 동작한다.
> **핵심: 파일에 박힌 버전 숫자를 신뢰하지 말 것.** 그 숫자도 결국 박제되어 구버전이 된다.
> 항상 "배포 사본 VERSION vs 원격 main VERSION"을 런타임에 비교한다.

## 1. 최신 사본 확보 (쓰기 가능 위치)

배포 사본(`/workspace/.../asterduck_skillset`)이 읽기전용이면 그 자리에서 갱신 불가.
쓰기 가능한 임시 위치에 받는다:

```
git -C /tmp/asterduck_skillset pull -q 2>/dev/null \
  || git clone -q https://<GITHUB_TOKEN>@github.com/ChoiKoal/asterduck_skillset.git /tmp/asterduck_skillset
```

- `<GITHUB_TOKEN>`: 그룹 CLAUDE.md, 또는 배포 사본 `git remote -v`에 박혀 있음.
- private repo라 `raw.githubusercontent.com` 단순 curl은 401일 수 있음 → **git(토큰)으로 받는 게 안전**.

## 2. 버전 대조

```
cat VERSION                          # 배포(현재 쓰는) 사본
cat /tmp/asterduck_skillset/VERSION  # 원격 main 최신
```

- **같으면** → 최신. 배포 사본 그대로 사용.
- **다르면** → 배포본이 구버전. 3번으로.

## 3. 최신으로 동작

`/tmp/asterduck_skillset/`의 **SKILL.md · CHANGELOG.md · openapi/** 를 **배포 사본보다 우선**해 따른다.

```
cat /tmp/asterduck_skillset/CHANGELOG.md   # 최근 항목(특히 Changed/MAJOR) 반영
```

특히 **호환성 깨진 변경**(예: v0.5.0 — 음용적기 모델: 셀러 날짜 → 와인 연도속성)을 현재 동작에 반드시 반영.

## 4. (선택) 배포 사본 재배포 알림

배포 파이프라인이 수동이면, 구버전 감지 시 배포 담당(콸)에게 "스킬 재배포 필요(vX→vY)"를 알린다.

---

**요약**: 버전 숫자 박제 신뢰 ❌ → 매 사용 전 `git pull` 캐시 → `VERSION` 대조 → 다르면 최신 사본으로 동작 ✅
