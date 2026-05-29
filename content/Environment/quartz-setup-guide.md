---
title: Quartz로 Local Wiki 연동
parent: Environment
nav_order: 2
math: mathjax
---

# Quartz로 LLM Wiki 구축 가이드

Obsidian 스타일의 위키를 GitHub Pages(github.io)에 배포하기까지의 전체 과정을 정리한 문서.

> **환경**: Windows + Node.js 22 + Quartz v4
> **결과물**: `https://YOUR_USERNAME.github.io/` 에 배포된 Obsidian 스타일 위키

---

## 목차

1. [사전 준비](#1-사전-준비)
2. [Quartz 프로젝트 셋업](#2-quartz-프로젝트-셋업)
3. [기본 설정 (`quartz.config.ts`)](#3-기본-설정-quartzconfigts)
4. [GitHub 저장소 연결](#4-github-저장소-연결)
5. [GitHub Pages 자동 배포](#5-github-pages-자동-배포)
6. [유지보수 워크플로우](#6-유지보수-워크플로우)
7. [커스터마이징](#7-커스터마이징)
8. [PyScript로 Python 코드 실행](#8-pyscript로-python-코드-실행)
9. [트러블슈팅](#9-트러블슈팅)

---

## 1. 사전 준비

### Git 설치

1. <https://git-scm.com/download/win> 에서 다운로드
2. 설치 시 기본값 유지 (PATH 옵션은 "Git from the command line..." 선택)
3. 확인:
   ```powershell
   git --version
   ```
4. 사용자 정보 설정 (한 번만):
   ```powershell
   git config --global user.name "본인이름"
   git config --global user.email "GitHub에등록한이메일"
   ```

### Node.js 설치

Quartz는 **Node.js 22 이상**이 필요.

1. <https://nodejs.org/> 에서 LTS 버전 다운로드
2. 설치 후 확인:
   ```powershell
   node -v
   ```
   `v22.x.x` 이상이어야 함

### GitHub 저장소 생성

본인만 쓸 위키라면 깔끔한 URL을 위해 저장소 이름을 `YOUR_USERNAME.github.io`로 만들기 (계정당 1개만 가능, **User Site**).

- 예: `kim123.github.io`
- README 등 자동 생성 옵션 모두 체크 해제 (빈 저장소)
- 결과 URL: `https://kim123.github.io/`

> 일반 저장소 이름(예: `llm-wiki`)을 쓰면 URL이 `https://kim123.github.io/llm-wiki/` 형태가 됨.

---

## 2. Quartz 프로젝트 셋업

### v4 브랜치로 clone

v5는 출시 직후라 셋업 버그가 보고되고 있어 **v4 권장**. Obsidian 핵심 기능(wikilink, 그래프, 백링크, callout, LaTeX)은 v4에서 다 동작.

```powershell
cd C:\github
git clone -b v4 https://github.com/jackyzha0/quartz.git llm-wiki
cd llm-wiki
```

> 한글 경로(`바탕화면`, `문서`)는 인코딩 문제 가능성 있으므로 영문 경로 권장.

### 버전 확인

```powershell
type package.json | findstr version
```

`"version": "4.x.x"`가 나와야 함. v5가 나오면 안 됨.

### 의존성 설치

```powershell
npm i
```

2~3분 소요.

### Quartz 초기화

```powershell
npx quartz create
```

선택지:
- `How do you want to initialize Quartz?` → **Empty Quartz**
- `Choose how Quartz will resolve links` → **Treat links as shortest path**

### 로컬 빌드 검증 (중요)

GitHub 푸시 전에 로컬에서 먼저 성공 확인:

```powershell
npx quartz build --serve
```

브라우저에서 <http://localhost:8080> 접속 → 사이트가 보이면 성공 → `Ctrl+C`로 중지.

---

## 3. 기본 설정 (`quartz.config.ts`)

VS Code 또는 메모장으로 `quartz.config.ts` 열어 상단 부분 수정:

```typescript
configuration: {
  pageTitle: "LLM Wiki",
  pageTitleSuffix: "",
  enableSPA: true,
  enablePopovers: true,
  analytics: null,
  locale: "ko-KR",
  baseUrl: "YOUR_USERNAME.github.io",   // 본인 GitHub 아이디
  ignorePatterns: ["private", "templates", ".obsidian"],
  defaultDateType: "modified",
  // ...
}
```

### baseUrl 규칙

| 저장소 이름 | baseUrl 값 | 최종 URL |
|---|---|---|
| `YOUR_USERNAME.github.io` | `"YOUR_USERNAME.github.io"` | `https://YOUR_USERNAME.github.io/` |
| `llm-wiki` (또는 임의 이름) | `"YOUR_USERNAME.github.io/llm-wiki"` | `https://YOUR_USERNAME.github.io/llm-wiki/` |

**주의**: `https://`나 끝 슬래시 없이 적어야 함.

---

## 4. GitHub 저장소 연결

clone 받은 폴더엔 원본 Quartz 저장소의 git 정보가 있으므로 제거하고 새로 초기화:

```powershell
cd C:\github\llm-wiki
Remove-Item -Recurse -Force .git

git init
git branch -M v4
git remote add origin https://github.com/YOUR_USERNAME/YOUR_USERNAME.github.io.git
git add .
git commit -m "Initial Quartz v4 setup"
git push -u origin v4
```

처음 push 시 GitHub 로그인 창이 뜸 → 브라우저에서 인증.

---

## 5. GitHub Pages 자동 배포

### 워크플로우 파일 만들기

`.github/workflows/deploy.yml` 생성:

```powershell
mkdir .github\workflows -Force
notepad .github\workflows\deploy.yml
```

내용:

```yaml
name: Deploy Quartz site to GitHub Pages

on:
  push:
    branches:
      - v4

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true

jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - name: Install Dependencies
        run: npm ci
      - name: Build Quartz
        run: npx quartz build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public

  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

푸시:

```powershell
git add .github/workflows/deploy.yml
git commit -m "Add deploy workflow"
git push
```

### GitHub 설정

1. **저장소 → Settings → Branches** → Default branch를 `v4`로 변경
2. **저장소 → Settings → Pages** → Source를 **GitHub Actions**로 선택
   - ⚠️ "Deploy from a branch"가 아님!
3. **저장소 → Actions** 탭에서 빌드 진행 확인 → 초록 체크 ✅ 확인
4. `https://YOUR_USERNAME.github.io/` 접속

---

## 6. 유지보수 워크플로우

### 기본 흐름

```powershell
cd C:\github\llm-wiki

# 로컬 미리보기 (작업 중 켜두기)
npx quartz build --serve
```

`content/` 폴더에 .md 파일을 추가/수정 후:

```powershell
git add .
git commit -m "Add: 새 글 제목"
git push
```

푸시 후 1~2분 뒤 사이트 자동 업데이트.

### 단축 명령

위 세 줄을 한 번에:

```powershell
npx quartz sync -m "Add: 새 글 제목"
```

자동으로 pull → commit → push 처리.

### 참고

- 빌드 결과물 `public/` 폴더는 `.gitignore`에 있어서 푸시되지 않음
- GitHub Actions가 클라우드에서 빌드하므로 로컬 빌드 결과 올릴 필요 없음

---

## 7. 커스터마이징

### 본문 너비 넓히기

`quartz/styles/custom.scss` 상단에 추가:

```scss
:root {
  --pageWidth: 1100px;       // 기본 750px
  --sidePanelWidth: 380px;   // 기본 320px
}
```

**`base.scss`는 직접 수정하지 말 것.** 업데이트 시 충돌·덮어쓰기 위험. 모든 커스터마이징은 `custom.scss`에서.

### 자주 쓰는 변수

| 변수 | 기본값 | 의미 |
|---|---|---|
| `--pageWidth` | `750px` | 본문 너비 |
| `--sidePanelWidth` | `320px` | 좌/우 사이드바 너비 |

### 기타 스타일 예시

```scss
// custom.scss
:root {
  --pageWidth: 1100px;
  --sidePanelWidth: 360px;
}

// 본문 줄 간격
article {
  line-height: 1.75;
}

// 코드 블록
pre {
  border-radius: 8px;
  font-size: 0.9rem;
}

// 인라인 코드
code:not(pre code) {
  background: var(--lightgray);
  padding: 2px 6px;
  border-radius: 4px;
}
```

---

## 8. PyScript로 Python 코드 실행

페이지 열 때 자동으로 Python 코드를 실행하고 결과 표시. **iframe 방식**이 Quartz의 HTML sanitization을 우회해 가장 안정적.

### 구조

```
content/
  └─ concepts/
      └─ softmax.md          ← 위키 페이지 (iframe만 포함)

quartz/static/
  └─ demos/
      └─ softmax.html        ← 실제 PyScript 코드
```

### 1) 데모 HTML 만들기

`quartz/static/demos/softmax.html`:

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>Softmax Demo</title>
  <link rel="stylesheet" href="https://pyscript.net/releases/2024.1.1/core.css">
  <script type="module" src="https://pyscript.net/releases/2024.1.1/core.js"></script>
  <style>
    body {
      font-family: -apple-system, sans-serif;
      margin: 0;
      padding: 16px;
      background: transparent;
    }
    #output {
      font-family: monospace;
      background: #f5f5f5;
      padding: 12px;
      border-radius: 6px;
      white-space: pre-wrap;
    }
    @media (prefers-color-scheme: dark) {
      #output { background: #2a2a2a; color: #eee; }
    }
  </style>
</head>
<body>
  <div id="output">Python 로딩 중...</div>

  <script type="py" config='{"packages":["numpy"]}'>
import numpy as np
from pyscript import document

scores = np.array([2.0, 1.0, 0.1])
softmax = np.exp(scores) / np.exp(scores).sum()

result = f"입력: {scores}\nSoftmax: {softmax.round(3)}"
document.getElementById("output").textContent = result
  </script>
</body>
</html>
```

### 2) 위키 페이지에서 iframe 임베드

`content/concepts/softmax.md`:

```markdown
---
title: Softmax 함수
---

# Softmax 함수

$$\text{softmax}(x_i) = \frac{e^{x_i}}{\sum_j e^{x_j}}$$

## 실시간 계산

<iframe 
  src="/static/demos/softmax.html" 
  width="100%" 
  height="200" 
  style="border: 1px solid #ccc; border-radius: 6px;"
  loading="lazy">
</iframe>
```

### iframe `src` 경로 주의

| baseUrl | src 값 |
|---|---|
| `YOUR_USERNAME.github.io` (루트) | `/static/demos/softmax.html` |
| `YOUR_USERNAME.github.io/llm-wiki` (서브경로) | `/llm-wiki/static/demos/softmax.html` |

### 제약사항

- ✅ 가능: numpy, pandas, matplotlib, scipy, sympy, scikit-learn 일부
- ❌ 불가: torch, tensorflow, transformers 등 무거운 ML 라이브러리
- 첫 로드 시 Pyodide 런타임(~10MB) 다운로드로 3~5초 소요
- API 키 같은 비밀 정보는 코드에 절대 넣지 말 것 (브라우저에서 노출됨)

---

## 9. 트러블슈팅

### Could not locate Gemfile

현재 폴더 위치 잘못. `cd`로 프로젝트 폴더로 이동.

### wdm 설치 실패 (Ruby 환경)

Ruby 3.x에서 wdm 호환성 문제. Gemfile에서 wdm 줄 주석 처리. (Quartz는 Ruby 안 쓰니 해당 없음)

### Layout 'default' requested ... does not exist (Jekyll 사용 시)

`remote_theme`는 GitHub Pages에서만 동작. 로컬용으로는 `theme: just-the-docs` 사용. (Quartz는 해당 없음)

### Could not resolve "../../.quartz/plugins" (v5 사용 시)

v5의 알려진 셋업 버그. v4로 다운그레이드 권장:

```powershell
git clone -b v4 https://github.com/jackyzha0/quartz.git
```

### Node.js 20 deprecated 경고

빌드는 정상. 워크플로우 `env:` 섹션에 추가:

```yaml
env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true
```

### 404 에러

체크 순서:
1. **저장소 이름과 baseUrl 일치**하는지
2. **Actions 빌드 성공**(초록 체크)했는지
3. **Settings → Pages → Source가 "GitHub Actions"** 인지
4. **Default branch가 푸시한 브랜치(v4)** 인지

### PyScript 코드가 실행 안 됨

원인 후보:
1. Markdown 코드 블록(```` ``` ````) 안에 넣음 → 백틱 제거
2. Quartz가 `<script>` 태그를 sanitize함 → **iframe 방식**으로 우회
3. iframe `src` 경로 잘못됨 → baseUrl 확인

### `git push` 후 사이트 변경 안 됨

1. Actions 탭에서 워크플로우 실행됐는지 확인
2. 빌드 실패(빨간 X)면 로그 확인
3. 빌드 성공이면 브라우저 강력 새로고침 (`Ctrl+Shift+R`)

---

## 부록: 자주 쓰는 명령어 모음

```powershell
# 로컬 미리보기
npx quartz build --serve

# 빌드만
npx quartz build

# Git 표준 워크플로우
git add .
git commit -m "변경 내용"
git push

# Git 단축 (Quartz 내장)
npx quartz sync -m "변경 내용"

# 빌드 상태 확인
git status
git log --oneline -5
```

---

## 부록: Obsidian과 함께 쓰기

Obsidian에서 `C:\github\llm-wiki\content\` 폴더를 Vault로 열면, 평소처럼 Obsidian에서 글 쓰고 PowerShell에서 `npx quartz sync`만 실행하면 사이트 업데이트.

지원되는 Obsidian 기능:
- `[[wikilink]]` 자동 링크
- `![[image.png]]` 이미지 임베드
- `![[다른노트]]` 노트 임베드
- `> [!note]`, `> [!warning]` 등 callout 문법
- LaTeX 수식 (`$...$`, `$$...$$`)
- Mermaid 다이어그램
- 폴더 구조 기반 자동 explorer
- 백링크, 그래프 뷰

---

작성일: 2026-05-29
