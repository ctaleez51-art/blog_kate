---
title: "SSH 키부터 GitHub Pages 배포까지 - 블로그 만들며 배운 것들"
date: 2026-09-01 09:30:00 +0900
categories: [Git, GitHub]
tags: [git, github, ssh, jekyll, github-pages, 환경변수, 학습기록]
---

어제 하루 동안 GitHub에 SSH로 연결하고, 저장소를 가져오고, 이 블로그를 만들었다.
중간에 오류도 여러 번 났는데 그 과정에서 배운 게 더 많았다.

Git의 `add` / `commit` / `push` 기초는 [앞 글]({% post_url 2026-08-31-git-기초-add-commit-push %})에 따로 정리했다.
이 글은 그 외의 것들이다.

## Git은 이미 깔려 있었다

먼저 설치 여부부터 확인했다.

```bash
git --version
```

```
git version 2.55.0.windows.5
```

여기서 한 가지 오해가 풀렸다. 튜터가 "CLI로 Git을 설치해야 한다"고 했는데,
나는 그게 뭔가 특별한 설치 방식인 줄 알았다. 알고 보니 이런 뜻이었다.

- ❌ GitHub Desktop 같은 **GUI 프로그램만** 깔고 터미널에서 `git` 명령어가 안 되는 상태
- ✅ 터미널에서 `git` 명령어를 직접 칠 수 있는 상태

내 컴퓨터에 깔린 **Git for Windows**는 설치되면서 `git.exe`를 넣어주기 때문에 이미 CLI가 있는 상태였다.
"패키지로 설치"와 "CLI로 설치"가 반대되는 개념이 아니었다.

## SSH 키로 GitHub에 연결하기

### 왜 필요한가

GitHub에 코드를 올리려면 "내가 이 계정 주인이다"를 증명해야 한다.
매번 아이디·비밀번호를 치는 대신 열쇠를 등록해두는 방식이 SSH 키다.

### 공개키와 개인키

키는 **한 쌍**으로 만들어진다.

| | 역할 | 어디에 두나 |
|---|---|---|
| 개인키 `id_ed25519` | 내 신분증 | **내 컴퓨터에만.** 절대 공개 금지 |
| 공개키 `id_ed25519.pub` | 자물쇠 | GitHub에 등록 |

GitHub에 공개키(자물쇠)를 걸어두고, 내 컴퓨터의 개인키(열쇠)로 연다는 그림이다.

### 만들기

```bash
ssh-keygen -t ed25519 -C "내이메일@example.com"
```

- `-t ed25519` — 키 방식. 요즘 권장되는 방식이다
- `-C` — 주석. 나중에 "이 키가 누구 건지" 알아보려고 이메일을 적는다

실행하면 저장 위치와 **passphrase(암호)** 를 물어본다.
passphrase는 개인키 파일 자체에 거는 추가 자물쇠다.

| | passphrase 없음 | passphrase 있음 |
|---|---|---|
| 쓸 때 | 바로 됨 | 매번 암호 입력 |
| 파일이 유출되면 | 바로 뚫림 | 암호를 모르면 못 씀 |

나는 없이 만들었다. 나중에 추가하려면 키를 새로 만들 필요 없이 자물쇠만 덧붙일 수 있다.

```bash
ssh-keygen -p -f ~/.ssh/id_ed25519
```

### GitHub에 등록

공개키 내용을 복사해서 **Settings → SSH and GPG keys → New SSH key** 에 붙여넣는다.

### 연결 확인

```bash
ssh -T git@github.com
```

```
Hi ctaleez51-art! You've successfully authenticated,
but GitHub does not provide shell access.
```

"shell access를 제공하지 않는다"는 문구 때문에 처음엔 실패한 줄 알았다.
**아니다.** GitHub은 SSH 로그인 셸을 제공하지 않고 git 통신만 받기 때문에 나오는 안내다.
`Hi 사용자명!` 이 나왔으면 성공이다.

### 호스트 키 지문 검증

연결할 때 이런 걸 물어본다.

```
The authenticity of host 'github.com' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
Are you sure you want to continue connecting (yes/no)?
```

무심코 `yes`를 치기 쉬운데, 이건 **"지금 접속하려는 서버가 진짜 GitHub이 맞냐"** 를 확인하는 절차다.
GitHub이 공식 문서에 지문을 공개해두었으니 그 값과 같은지 대조하고 승인하는 게 맞다.
한 번 승인하면 `~/.ssh/known_hosts`에 기록되어 다음부터는 안 물어본다.

## 저장소 가져오기

```bash
git clone git@github.com:사용자명/저장소명.git
```

여기서 주소 방식이 두 가지라는 걸 알았다.

| | clone | push |
|---|---|---|
| HTTPS `https://github.com/...` | 됨 | 매번 로그인 정보 필요 |
| SSH `git@github.com:...` | 됨 | 등록한 키로 자동 |

내 저장소라면 SSH가 편하다. 이미 HTTPS로 받아온 경우엔 주소만 바꿔주면 된다.

```bash
git remote set-url origin git@github.com:사용자명/저장소명.git
```

`git remote -v` 로 현재 어떤 주소를 쓰는지 확인할 수 있다.

## GitHub Pages 블로그 만들기

### Jekyll이 뭔가

내가 쓴 Markdown 파일(`.md`)을 웹사이트용 HTML로 변환해주는 도구다.
GitHub Pages가 이걸 지원해서, 저장소에 `.md` 파일만 올리면 알아서 블로그가 된다.

### 테마 선택

이 블로그는 **Chirpy** 테마를 쓴다. 기본 테마(minima)와 비교하면 이렇다.

| | minima | Chirpy |
|---|---|---|
| 파일 수 | 5개 정도 | 20개 넘음 |
| 카테고리·태그 페이지 | ❌ | ✅ |
| 목차(TOC) | ❌ | ✅ |
| 검색 | ❌ | ✅ |
| 다크모드 | ❌ | ✅ |
| 배포 방식 | 브랜치에서 바로 | GitHub Actions로 빌드 |

기록이 쌓였을 때 찾기 쉬운 쪽을 골랐다.
대신 설치 과정이 복잡했고, 실제로 여기서 오류가 났다.

### 빌드가 실패했다

push하고 나니 Actions 탭에 빨간 X가 떴다.

```
HttpError: Not Found
Get Pages site failed. Please verify that the repository has Pages enabled
and configured to build using GitHub Actions
```

원인을 이해하려면 등장인물 둘을 알아야 한다.

- **GitHub Pages** — 저장소 파일을 웹사이트로 보여주는 기능. **기본적으로 꺼져 있다**
- **GitHub Actions** — push하면 자동으로 일하는 로봇. "Markdown을 웹사이트로 조립해서 Pages에 올려라"를 맡는다

시간 순서가 문제였다.

| 순서 | 일어난 일 |
|---|---|
| 1 | push → 로봇 출동 |
| 2 | 로봇이 Pages를 찾음 → **꺼져 있음** → 실패 ❌ |
| 3 | Settings에서 Pages를 켬 |
| 4 | 근데 로봇은 이미 돌아갔고 자동으로 다시 오지 않음 |

로봇에게 "가게에 물건을 진열해라"라고 시켰는데 가게 문이 잠겨 있어서 그냥 돌아온 셈이다.

### 해결

워크플로 파일(`.github/workflows/pages-deploy.yml`)에 옵션 한 줄을 추가했다.

```yaml
- name: Setup Pages
  id: pages
  uses: actions/configure-pages@v6
  with:
    enablement: true
```

`enablement`는 "활성화"라는 뜻이다.
**"문이 잠겨 있으면 네가 직접 열고 들어가라"** 는 지시를 추가한 것이다.
이걸 push하니 로봇이 다시 출동해서 성공했다.

오류 메시지 끝에 `consider exploring the 'enablement' parameter`라고 힌트가 적혀 있었다.
**오류 메시지를 끝까지 읽는 게 중요하다**는 걸 배웠다.

### 주소가 두 개다

빌드가 성공했는데도 블로그가 안 보여서 한참 헤맸다. 주소를 잘못 보고 있었다.

| 주소 | 보이는 것 |
|---|---|
| `github.com/사용자명/저장소명` | **파일 목록** (저장소) |
| `사용자명.github.io/저장소명/` | **블로그** (완성된 사이트) |

`github.com`과 `github.io`는 다른 곳이다.

### baseurl 설정

저장소 이름이 `사용자명.github.io`가 아니면 블로그가 하위 경로에 생긴다.
그래서 `_config.yml`에 이걸 적어줘야 링크가 깨지지 않는다.

```yaml
baseurl: "/blog_kate"
```

## 개발 도구 설치하며 배운 것

### 환경변수와 PATH

Python 패키지 관리 도구인 uv를 설치했는데, 설치 직후에 `uv --version`이 안 먹혔다.
여기서 **환경변수**를 알게 됐다.

환경변수는 컴퓨터가 프로그램들에게 알려주는 **설정 메모**다. `이름 = 값` 형태다.

```
USERPROFILE = C:\Users\내계정
TEMP        = C:\Users\내계정\AppData\Local\Temp
PATH        = C:\Windows\system32; C:\Program Files\nodejs; ...
```

이 중 **PATH**가 특별하다. 값이 하나가 아니라 **폴더 주소 목록**이다.

터미널에 `node`라고 치면 컴퓨터가 이 목록을 **위에서부터 순서대로** 뒤져서 `node.exe`를 찾는다.
찾으면 실행하고, 끝까지 없으면 "그런 명령어 없다"고 한다.
덕분에 긴 경로를 매번 치지 않아도 된다.

### 왜 터미널을 다시 열어야 하나

설치 후 이런 메시지가 나왔다.

```
Path environment variable modified; restart your shell to use the new value.
```

핵심은 **터미널이 시작할 때 PATH를 복사해서 들고 있다**는 점이다.

| 시점 | 원본 PATH | 이미 열린 터미널의 사본 |
|---|---|---|
| 터미널을 열었을 때 | uv 없음 | uv 없음 |
| uv 설치 후 | **uv 추가됨** ✅ | uv 없음 ❌ (옛날 사본) |

원본은 갱신됐지만 이미 열려 있던 터미널은 모른다.
닫고 다시 열면 새 사본을 받아오니까 그때부터 작동한다.

### 버전이 사람마다 다른 이유

다른 사람들과 버전을 비교해보니 다 달랐다. 이유가 몇 가지 있었다.

1. **설치 시점이 다르다** — Node.js는 2주마다 새 버전이 나온다
2. **설치 방법이 다르다** — 공식 사이트 / winget / nvm 이 각각 다른 버전을 준다
3. **Node.js는 두 갈래로 나온다**
   - **짝수** (20, 22, 24) = **LTS**. 안정 버전, 오래 지원. 학습·실무용 권장
   - 홀수 (21, 23, 25) = Current. 최신 기능 실험용

내 환경은 이렇다.

```
node  v24.15.0    ← 짝수 = LTS 라인
npm   11.12.1
uv    0.12.7
git   2.55.0.windows.5
```

버전이 다르다는 것만으로 문제가 되진 않는다.
다만 메이저 숫자가 크게 다르면 라이브러리 호환 문제가 생길 수 있고,
여러 명이 같은 프로젝트를 할 때 "내 컴퓨터에서는 되는데" 문제의 원인이 된다.
그래서 실무에서는 프로젝트마다 버전을 고정한다.

## 아직 헷갈리는 것

- Jekyll이 파일들을 어떤 순서로 읽어서 사이트를 만드는지 아직 흐릿하다.
  `_config.yml`, `_posts`, `_tabs` 말고 나머지 파일들이 각각 뭘 하는지 모르겠다.
- `.github/workflows/` 안의 YAML 문법이 낯설다. 오류가 나면 어디를 봐야 할지 감이 안 온다.

## 다음에 볼 것

- `git branch`, `git diff`, `.gitignore`
- Jekyll 폴더 구조 하나씩 뜯어보기
