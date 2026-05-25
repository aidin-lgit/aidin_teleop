# AIDIN Teleop Documentation Site

[![Pages](https://img.shields.io/badge/docs-online-brightgreen)](https://aidin-lgit.github.io/aidin_teleop/)

**Rainbow Robotics RBY1 + AIDIN Robot Hand** 휴머노이드 원격제어 및 Physical AI 학습·추론 파이프라인의 공식 한국어 매뉴얼.

> 🌐 **라이브 사이트**: <https://aidin-lgit.github.io/aidin_teleop/>

## 스택

- [Hugo](https://gohugo.io/) (정적 사이트 생성기)
- [Hugo Relearn Theme](https://github.com/McShelby/hugo-theme-relearn) (포크하여 커스터마이즈)
- 한국어 단일, Zen Light/Dark + Noto Sans KR

## 로컬 개발

```bash
cd docs
hugo server -p 3131 --cleanDestinationDir
```

→ <http://localhost:3131/aidin_teleop/>

## 저장소 구조

```
.
├── docs/                       ← 매뉴얼 사이트
│   ├── content/                ← 마크다운 콘텐츠 (이곳을 편집)
│   │   ├── introduction/
│   │   ├── architecture/
│   │   ├── hardware-setup/
│   │   ├── software-install/
│   │   ├── teleoperation/
│   │   ├── data-collection/
│   │   ├── training/
│   │   ├── inference/
│   │   ├── tutorial/
│   │   ├── troubleshooting/
│   │   └── reference/
│   ├── assets/images/          ← 로고·favicon (Hugo Pipes 처리)
│   ├── static/images/          ← 본문 이미지 (그대로 서빙)
│   ├── layouts/partials/       ← 테마 오버라이드 (auto-logo, custom-header)
│   └── config/                 ← Hugo 설정 (_default / github)
├── layouts/, assets/, i18n/    ← Relearn 테마 본체 (포크된 상태)
└── .github/workflows/          ← GitHub Pages 자동 배포
```

## 콘텐츠 편집

새 페이지 추가:

1. 해당 챕터 디렉토리에 `<slug>.md` 생성
2. 프론트매터 작성:

   ```toml
   +++
   title = "페이지 제목"
   weight = 1
   +++
   ```

3. 이미지는 `docs/static/images/` 에 두고 `![](/images/<file>)` 로 참조
4. 사이즈 조정: `![](/images/foo.png?width=400)`

## 자동 배포

`main` 브랜치 push 시 [.github/workflows/docs-build-deployment.yaml](.github/workflows/docs-build-deployment.yaml) 가 트리거되어:

1. Hugo 가 `docs/` 를 빌드
2. `actions/deploy-pages` 가 GitHub Pages 로 배포

## Related Repository

본 매뉴얼이 다루는 텔레옵 스택 소스 코드: <https://github.com/aidin-lgit/aidin_rby1_teleop>

## Acknowledgements

본 사이트는 [Hugo Relearn Theme](https://github.com/McShelby/hugo-theme-relearn) (MIT) 을 포크하여 사용합니다. 테마 원본 라이선스는 [LICENSE](LICENSE) 참조.
