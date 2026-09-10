---
title: "Build site 성공이 발행 성공이 아니다 — 빌드가 죽었는지 확인하는 순서"
date: 2026-09-11 08:32:03 +0900
categories: [Jekyll]
tags: [github-actions, jekyll, 트러블슈팅, 블로그, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

글을 push했는데 사이트에 새 글이 안 보였다.
처음엔 전에 겪은 **서비스 워커 캐시** 문제인 줄 알았다. 아니었다. **빌드가 실패해 있었다.**

## 캐시를 의심하기 전에 볼 것

전에 캐시 때문에 옛 글이 계속 보인 적이 있어서, 이번에도 `Update` 버튼부터 찾았다.
그런데 그 버튼은 **서버에 새 내용이 올라가 있을 때** 뜨는 것이다.
빌드가 죽어서 새 내용이 아예 안 만들어졌으면, 뜰 이유가 없다.

의심하는 순서를 거꾸로 잡았던 것이다. **캐시는 맨 마지막에 의심할 것이다.**

## 확인 순서 4단계

### (1) push는 됐는가

로컬과 원격의 커밋 해시를 비교한다.

```bash
git fetch origin
git rev-parse --short HEAD
git rev-parse --short origin/main
```

두 값이 같으면 올라간 건 맞다. 여기까진 이상이 없었다.

### (2) 빌드는 성공했는가

GitHub Actions 결과는 로그인 없이도 볼 수 있다.

```bash
curl -s "https://api.github.com/repos/ctaleez51-art/blog_kate/actions/runs?per_page=3"
```

```
"head_sha": "47cce99..."
"conclusion": "failure"
```

**실패였다.** 바로 앞 커밋의 결과는 `success` 였다. 그래서 사이트가 직전 상태로 남아 있었던 것이다.

### (3) 어느 단계에서 죽었는가

같은 API에서 그 실행의 `jobs` 를 보면 단계별로 나온다.

```bash
curl -s "https://api.github.com/repos/ctaleez51-art/blog_kate/actions/runs/<실행번호>/jobs"
```

```
Build site              success
Test site               failure      ← 여기
Upload site artifact    skipped
```

`Test site` 는 `htmlproofer` 라는 도구가 **만들어진 페이지의 내부 링크와 이미지가 실제로 있는지**
검사하는 단계다. 사이트를 만드는 건 성공했는데, 검사에서 걸린 것이다.

### (4) 오류 본문 읽기

여기서 막혔다. API로 보이는 것은 이 한 줄뿐이다.

```
Process completed with exit code 1
```

무엇이 잘못됐는지는 안 나온다.
**실제 오류 문구는 GitHub 웹 화면에서 `Test site` 를 클릭해 펼쳐야** 보였다.

## 빌드가 실패하면 사이트는 직전 상태로 살아 있다

이게 헷갈렸던 진짜 이유다.

| 단계 | 결과 |
|---|---|
| Build site | 성공 |
| Test site | **실패** |
| Upload site artifact | 건너뜀 |
| deploy | 건너뜀 |

배포 단계가 통째로 건너뛰어지므로 **사이트는 어제 상태 그대로 멀쩡히 떠 있다.**
오류 화면도 안 뜨고, 옛 글도 잘 보인다. 겉으로는 아무 일도 없는 것처럼 보인다.
"새 글만 없는" 상태라서, 캐시 문제와 증상이 똑같다.

## 정리

- 새 글이 안 보이면 **캐시가 아니라 빌드부터** 본다
- `Build site` 성공은 발행 성공이 아니다. 그 뒤에 **검사 단계가 하나 더** 있다
- 빌드가 실패하면 배포가 건너뛰어져 사이트는 직전 상태로 남는다. 그래서 조용하다
- 성공·실패와 죽은 단계까지는 API로 보이지만, **오류 본문은 웹 화면에서 펼쳐야** 보인다
