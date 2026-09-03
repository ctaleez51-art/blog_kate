---
title: "git init부터 배포까지 - 저장소를 처음부터 만들어보고 알게 된 것들"
date: 2026-09-03 09:25:10 +0900
categories: [Git, GitHub]
tags: [git, github-pages, 배포, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

지금까지 만든 저장소는 전부 **GitHub에서 먼저 만들고 `git clone` 으로 받아온 것**이었다.
이번엔 내 컴퓨터에 이미 만들어둔 게임 폴더가 있었다.
순서가 반대라 `git init` 부터 시작했다.

그 과정에서 걸린 것들과, 배포를 확인하다 알게 된 것들을 정리한다.

## 1. 새 저장소를 처음부터 만드는 순서

```bash
git init -b main
git add index.html
git commit -m "픽셀 러너 게임 추가"
git remote add origin git@github.com:ctaleez51-art/pixel_runner.git
git push -u origin main
```

한 줄씩 뜯어보면 이렇다.

- `git init -b main` — 이 폴더를 git 저장소로 만든다.
  `-b main` 은 **기본 브랜치 이름을 지정**하는 것이다. 안 붙이면 다른 이름이 될 수 있다
- `git add` — `git status` 에서 `??` 이던 파일이 `A` 로 바뀐다
- `git remote add origin ...` — 원격 저장소 주소에 `origin` 이라는 별명을 붙인다.
  `origin` 은 **관례적인 이름일 뿐 특별한 뜻은 없다.** 다른 이름을 붙여도 된다
- `git push -u origin main` — `-u` 는 `--set-upstream`.
  한 번 붙여두면 **앞으로는 `git push` 만 쳐도** `origin main` 으로 간다

## 2. GitHub에서 저장소를 만들 때 체크박스를 건드리면 안 된다

`gh`(GitHub 명령줄 도구)가 안 깔려 있어서 저장소는 웹에서 직접 만들었다.

저장소 만드는 화면에 **README / .gitignore / license** 를 같이 만들어주는 체크박스가 있다.
편해 보이지만, 이 경우엔 **켜면 안 된다.**

켜면 GitHub 쪽에 커밋이 하나 생긴다.
내 컴퓨터에도 이미 커밋이 있으니, 양쪽에 **조상이 다른 커밋**이 하나씩 있게 된다.
git은 이걸 이어붙일 수 없어서 push가 `rejected` 된다.

로컬에 이미 커밋이 있다면 저장소는 **완전히 비어 있는 상태로** 만든다.

## 3. push가 온전히 끝났는지 확인하기

```bash
git log origin/main..HEAD --oneline
```

**비어 있으면** "로컬에는 있는데 원격에는 없는 커밋"이 하나도 없다는 뜻이다.
push 전에는 무엇이 올라갈지 보는 용도, push 후에는 다 올라갔는지 보는 용도로 쓴다.

## 4. Pages 배포 확인은 API가 아니라 주소로

배포됐는지 확인하려고 이 API를 불렀다.

```bash
curl -s "https://api.github.com/repos/ctaleez51-art/pixel_runner/pages"
```

**404가 왔다.** 그런데 사이트는 멀쩡히 열렸다.

`pages` API 는 **로그인이 필요하다.** 로그인 없이 부르면 배포가 잘 돼 있어도 404가 온다.
이미 배포가 끝난 다른 저장소로 대조해봤더니 거기도 똑같이 404였다.
**404를 "배포 안 됨"으로 읽으면 틀린다.**

→ Pages 확인은 **주소에 직접 접속해서 응답 코드를 본다.**

```bash
curl -s -o /dev/null -w "HTTP %{http_code}\n" "https://ctaleez51-art.github.io/pixel_runner/"
```

참고로 `actions/runs` API 는 로그인 없이 보인다. 빌드 성공 여부는 그걸로 계속 확인할 수 있다.

## 5. 저장소 옆 `Deployments` 의 숫자는 배포 횟수가 아니다

배포를 두 번 했는데 숫자가 계속 `1` 이었다.

옆의 숫자는 **환경(environment) 개수**다. 배포 횟수가 아니다.
API로 확인해보니 environments 1개, deployments 2건이었다.

Pages를 켜면 `github-pages` 라는 환경이 자동으로 하나 생기고,
이후 push마다 **같은 환경에 새 배포를 덮어쓴다.** 목적지가 하나니까 계속 `1` 이다.

- 초록 체크 = 최근 배포 성공
- 빨간 X = 빌드 실패

**배포 상태를 가장 빨리 볼 수 있는 자리가 여기다.**
그리고 `README.md` 만 추가해도 Pages는 다시 빌드된다. `main` 에 커밋이 들어가면 다시 돈다.

## 6. 남의 저장소를 받아서 테스트하기

짝의 게임을 테스트하려고 저장소를 클론했다.

- 남의 **공개** 저장소를 클론할 땐 `https://` 가 간단하다.
  SSH는 "쓰기 권한이 있는 저장소"에 필요한 것이다
- 클론한 폴더에서 뭘 고쳐도 원본에 영향이 없다. 쓰기 권한이 없어서 push 자체가 안 된다
- 짝이 코드를 고친 뒤 최신 버전을 받으려면 `git pull`

**로컬 서버가 필요한가?** `fetch` / `type="module"` / 외부 CDN 이 **없으면**
로컬 서버 없이 더블클릭으로 열린다.
있으면 `file://` 로 열 때 브라우저가 막아서 화면이 하얗게 뜬다. 그때 서버가 필요하다.

짝 코드는 `index.html` 하나로 완결이라 더블클릭으로 열렸다.

> 참고: PowerShell의 `Invoke-Item`(별명 `ii`)은 **탐색기에서 더블클릭한 것과 완전히 같다.**
> `.html` 의 기본 동작이 "기본 브라우저로 열기"라서 브라우저가 열리는 것뿐이다.
> 터미널에서 안 나가려고 쓰는 것이지, 안 써도 된다.
{: .prompt-tip }

## 7. 클론하면 파일 크기가 커진다 - CRLF

GitHub 원본이 8,749바이트인데 클론한 파일은 **9,030바이트**였다. 281바이트 늘었다.

줄바꿈을 리눅스·맥은 `LF` 한 글자로, 윈도우는 `CRLF` 두 글자로 쓴다.
git은 **저장소에는 `LF` 로 저장하고, 윈도우로 꺼낼 때 `CRLF` 로 바꿔준다.**
그래서 줄 수만큼 바이트가 늘어난다. 내용이 달라진 게 아니다.

커밋할 때 나오는 이 문구가 그것이다. **오류가 아니라 알림이다.**

```
warning: in the working copy of 'README.md',
LF will be replaced by CRLF the next time Git touches it
```

## 8. PRD 와 README 는 뭐가 다른가

짝은 저장소에 `PRD.md` 를 뒀고 나는 `README.md` 를 뒀는데, 내용이 비슷해 보였다.

| | PRD | README |
|---|---|---|
| 언제 쓰나 | **만들기 전** | 만든 뒤 (계속 갱신) |
| 누구에게 | 만드는 사람·결정하는 사람 | 저장소를 여는 사람 |
| 답하는 것 | 무엇을 왜 만들 것인가 | 이게 뭐고 어떻게 쓰나 |
| 다 만든 뒤 | 역할이 끝남 | 계속 살아 있음 |

PRD = **P**roduct **R**equirements **D**ocument.

내 README에는 "요청받은 조건과 구현" 표가 있는데,
**왼쪽 칸(요청)이 PRD, 오른쪽 칸(구현)이 README**에 해당한다.
과제가 작아서 두 문서가 겹쳐 보인 것이다.

저장소에 흔히 두는 문서는 `README` / `LICENSE` / `.gitignore` / `CHANGELOG` /
`CONTRIBUTING` / `docs/` / `SECURITY` / `CLAUDE.md`·`AGENTS.md` 정도다.
**PRD는 이 목록에 잘 안 들어간다.** 보통 저장소가 아니라 노션·컨플루언스에 있다.

만들기 전에 쓰는 문서는 이름이 여러 가지다 — 설계 문서, 기술 스펙, RFC,
**ADR**(Architecture Decision Record, 왜 A 대신 B를 골랐는지 남기는 기록).

## 곁가지 - 저장소가 달라 보였던 이유

짝 저장소와 내 저장소의 화면이 달라 보여서 비교해봤는데, 대부분 저장소 차이가 아니었다.

- 가운데 `Ask Copilot about this repository` 박스는 **README가 없어서 생긴 빈자리**다.
  박스 안에 "이 저장소에는 아직 README가 없다"고 써 있다
- `Deployments` 섹션이 없는 것은 Pages를 안 켰기 때문이다
- **`Pin`(내 저장소) vs `Report repository`(남의 저장소) 는 보는 사람 차이다.** 저장소 차이가 아니다
- `Contributors` 에 `claude` 가 잡히는 건 커밋 메시지의 `Co-Authored-By` 때문이다. 내 저장소도 같다

## 정리

- `git init -b main` → `add` → `commit` → `remote add` → `push -u`
- 로컬에 커밋이 있으면 GitHub 저장소는 **완전히 비워서** 만든다
- `pages` API는 로그인이 필요하다. 배포 확인은 **주소 응답 코드**로 한다
- `Deployments` 숫자는 환경 개수다. 배포 횟수가 아니다
- CRLF 경고는 오류가 아니다. 윈도우 줄바꿈으로 바꿔준다는 알림이다
