# Aster.duck AI Skillset

[Aster.duck](https://asterduck.koalstudio.com) 플랫폼을 AI 에이전트로 활용하기 위한 스킬 모음입니다.

## Available Skillsets

| 서비스 | 설명 | 스킬 수 |
|--------|------|---------|
| [Auth](./auth/) | 공용 인증 (로그인, 회원가입, 토큰) | 1 |
| [CoffeeDuck](./coffeeduck/) | 커피 검색, 등록, 테이스팅 노트 | 3 |
| [WineDuck](./wineduck/) | 와인 검색, 등록, 테이스팅 노트 | 3 |

## Quick Start

```bash
git clone https://github.com/ChoiKoal/asterduck_skillset.git
```

각 서비스 디렉토리의 README를 참고하세요.

## 구조

```
asterduck_skillset/
├── auth/                  # 공용 인증 스킬
│   └── SKILL.md
├── coffeeduck/            # 커피덕 스킬셋
│   ├── README.md
│   ├── search/SKILL.md
│   ├── tasting/SKILL.md
│   └── coffee/SKILL.md
└── wineduck/              # 와인덕 스킬셋
    ├── README.md
    ├── search/SKILL.md
    ├── tasting/SKILL.md
    └── wine/SKILL.md
```

## License

MIT

---

Built by [Aster.duck](https://asterduck.koalstudio.com) team
