---
title: "토익 어휘 게임 만들어 배포하기 - clone부터 GitHub Pages까지"
date: 2026-09-02 09:58:00 +0900
categories: [Git, GitHub]
tags: [git, github-pages, javascript, localstorage, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

수업 퀘스트로 게임을 만들었다. 토익 어휘 짝맞추기 게임이다.

- 저장소: <https://github.com/ctaleez51-art/voca_game>
- 게임: <https://ctaleez51-art.github.io/voca_game/>

만들면서 배운 것을 순서대로 정리한다.

## 1. `git clone` 이 폴더를 만들어준다

폴더를 미리 만들 필요가 없었다. `clone` 이 세 가지를 한 번에 한다.

1. 저장소 이름으로 **폴더 생성**
2. 그 안에 내용 내려받기
3. `origin`(원격 주소) 연결

그래서 clone은 **상위 폴더에서** 실행한다.
이미 만든 폴더 안에서 clone하면 그 안에 또 폴더가 생긴다.

```bash
cd C:\Users\ctale\Desktop\claudecode
git clone git@github.com:ctaleez51-art/voca_game.git
```

주소를 `https://` 대신 `git@github.com:` 형태로 쓰면 push할 때 로그인을 안 묻는다.
SSH 키가 이미 등록돼 있어서 그렇다.

**빈 저장소를 clone하면** 이런 경고가 뜬다. 정상이다.

```
warning: You appear to have cloned an empty repository
```

## 2. GitHub 먼저 vs 내 컴퓨터 먼저

둘 다 되는데 단계 수가 다르다.

**GitHub 먼저 (이번에 한 방식)**

```
저장소 생성 → git clone → 작업 → add/commit → push
```

**내 컴퓨터 먼저**

```
폴더 생성 → git init → 작업 → add/commit
→ GitHub에 빈 저장소 생성 → git remote add origin <주소>
→ git push -u origin main
```

어느 쪽이든 **GitHub에 저장소는 만들어야 한다.**
`git push` 가 없는 저장소를 알아서 만들어주지는 않는다.

차이는 뒷정리다. clone은 `origin` 연결과 브랜치 추적을 자동으로 잡아준다.
새로 시작할 땐 clone이 단계가 적고, 이미 만들어둔 폴더를 올릴 땐 두 번째 방식을 쓴다.

## 3. push 했다고 웹사이트가 되는 건 아니다

이게 헷갈렸던 부분이다.

**push = 파일을 GitHub 창고에 넣는 것.** 이것만으로는 웹 주소로 안 열린다.

웹사이트로 보이게 하려면 **GitHub Pages를 켜야** 한다.

```
Settings → Pages → Source: Deploy from a branch
→ Branch: main / (root) → Save
```

한 번만 켜두면 그 뒤로는 **push할 때마다 자동으로 웹에 반영**된다. 1~2분 걸린다.

저장소 주소와 실행 주소는 다르다.

| | 주소 | 보이는 것 |
|---|---|---|
| 저장소 | `github.com/ctaleez51-art/voca_game` | 파일 목록 (코드) |
| 실행 | `ctaleez51-art.github.io/voca_game/` | 실제 게임 |

저장소에서 `index.html` 을 눌러도 게임은 안 돌아간다. 코드 내용만 보여준다.

## 4. HTTP 응답 코드로 배포 확인하기

배포가 됐는지 확인할 때 쓴 숫자들.

- **200** — 정상. 파일을 찾아서 보냈다
- **404** — 파일을 못 찾았다
- **500** — 서버 쪽 오류

```bash
curl -s -o /dev/null -w "%{http_code}\n" "https://.../styles.css"
```

배포 직후에 `index.html` 은 200인데 `styles.css` 가 404였다.
→ 아직 **옛 버전(파일 하나짜리)이 배포돼 있어서** 그 파일이 웹에 없던 것이다.
배포가 끝나니 셋 다 200이 됐다.

## 5. HTML / CSS / JS 를 파일로 나누기

처음엔 `index.html` 하나에 전부 넣었다. 773줄이었다.
평가기준이 "html, css, js가 적절하게 만들어졌는가"라 세 파일로 나눴다.

```
index.html    59줄
styles.css   153줄
script.js    560줄
```

HTML에서 이렇게 불러온다.

```html
<link rel="stylesheet" href="styles.css">
<script src="script.js"></script>
```

**나눈 뒤에 확인이 안 되는 문제가 있었다.**
파일을 브라우저로 직접 열면(`file://`) 미리보기 도구가 HTML만 복사해 띄우는 탓에
`styles.css` 와 `script.js` 를 못 찾았다. 화면이 깨져 보였다.

→ 로컬에 임시 웹서버를 띄워서 확인했다.

```bash
python -m http.server 8765
```

GitHub Pages도 같은 방식(웹서버)이라 **여기서 되면 배포해도 된다.**

## 6. 모바일에서 좌우 줄이 어긋났던 이유

단어와 뜻을 각각 **독립된 세로 목록** 두 개로 쌓았다.
PC에서는 멀쩡했는데 좁은 화면에서 문제가 생겼다.

긴 숙어가 두 줄로 접히면서 그 칸만 높이가 커졌고, 좌우 줄이 안 맞았다.

→ 두 열을 **하나의 표(grid)** 로 묶었다.
한 줄에 `[단어][뜻]` 두 칸씩 넣으면 같은 줄끼리 높이가 자동으로 맞춰진다.

```css
.board { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
```

PC에서는 글자가 짧아 높이가 같으니 안 보이던 문제였다.
**화면 크기를 바꿔서 확인해봐야 알 수 있다.**

## 7. 브라우저에 저장하기 - 단어장

게임에 나온 단어를 모아두는 단어장을 만들었다. 저장은 `localStorage` 를 쓴다.

```js
localStorage.setItem(KEY, JSON.stringify(data));
JSON.parse(localStorage.getItem(KEY));
```

성질을 알아둬야 한다.

- 창을 닫아도, 컴퓨터를 껐다 켜도 **남는다**
- **다른 기기·다른 브라우저에서는 안 보인다**
- 시크릿 모드는 창을 닫으면 사라진다
- 기기를 넘나들게 하려면 서버와 로그인이 필요하다

시크릿 모드 등에서 막혀 있을 수 있어서 `try / catch` 로 감쌌다.

## 8. 만들면서 고쳐나간 것들

처음 만든 것과 최종본이 꽤 다르다. 해보니까 알게 된 것들이다.

**단어가 8개뿐이라 같은 문제가 반복됐다.**
60초 안에 15개를 맞추니 바닥이 났다. → 레벨당 30개로 늘림.

**"틀린 횟수"가 문제 개수가 아니었다.**
16/20을 맞췄는데 틀린 횟수가 3으로 나왔다. 잘못 누른 횟수를 세고 있었기 때문이다.
다시 풀 수 있게 만들어놨으니 결국 맞췄으면 맞춘 건데, 누른 횟수를 셀 이유가 없었다.
→ **틀린 문제 = 20 − 맞춘 문제** 로 통일.

**정확도와 점수가 같은 숫자였다.**
20문제 5점씩이니 16개 맞추면 80점, 정확도도 80%. → 정확도 삭제.

**결과 화면 칸을 억지로 채우려 했다.**
걸린 시간을 넣었다가 뺐다. 필요해서가 아니라 칸이 비어서 채우려던 것이었다.
→ 맞춘 문제 / 틀린 문제 두 칸만 남기고,
**누르면 해당 단어 목록이 펼쳐지게** 바꿈.

## 9. 짝맞추기의 두 가지 방식

- **카드 뒤집기** — 기억력 게임이 된다. 어휘 실력보다 위치 외우기가 된다
- **양쪽 연결** — 왼쪽 단어, 오른쪽 뜻을 보고 짝을 고른다

제한시간 안에 몇 개를 맞추는 게임이라 **양쪽 연결**을 골랐다.

**뜻 순서를 섞을 때 함정이 있었다.**
5개를 무작위로 섞으면 가끔 단어와 **같은 순서**로 나온다.
그러면 짝이 한눈에 보여서 게임이 안 된다.
→ 섞은 결과가 원래 순서와 같으면 **다시 섞도록** 했다.

## 정리

- `git clone` 이 폴더 생성 + 내려받기 + `origin` 연결을 한 번에 한다
- **push는 창고에 넣는 것이고, Pages를 켜야 웹사이트가 된다**
- 파일을 나눈 뒤에는 `file://` 로 못 열어본다. 로컬 웹서버를 띄운다
- 모바일은 화면 크기를 바꿔봐야 문제가 보인다
- 만들어놓고 써봐야 설계가 틀린 게 보인다
