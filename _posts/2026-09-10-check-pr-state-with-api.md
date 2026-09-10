---
title: "PR 상태를 화면 대신 명령어로 확인하기"
date: 2026-09-10 09:11:33 +0900
categories: [GitHub]
tags: [풀리퀘스트, api, curl, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

내 PR이 승인됐는지 아직인지를 화면에서 읽으려다 몇 번 잘못 읽었다.
목록 화면의 숫자와 표시가 뜻하는 게 생각과 달랐다.

값으로 확인하는 방법이 있다. 공개 저장소는 **로그인 없이** GitHub API로 볼 수 있다.

## 명령어

```bash
curl -s "https://api.github.com/repos/소유자/저장소이름/pulls/1" | grep -E '"(state|merged|merged_at|commits)"'
```

맨 뒤의 `1` 이 PR 번호다.

## 결과

```
"state": "open"
"merged": false
"merged_at": null
"commits": 2
```

## 보는 값

| 값 | 뜻 |
|---|---|
| `state` | `open` 이면 열려 있음, `closed` 면 닫힘 |
| `merged` | `true` 면 병합됨 |
| `merged_at` | 병합된 시각. 안 됐으면 `null` |
| `commits` | 이 PR에 붙어 있는 커밋 수 |

`state` 와 `merged` 를 같이 봐야 한다. **`closed` 인데 `merged: false` 인 경우가 있다.**
승인 없이 그냥 닫은 것이다. `closed` 만 보고 "합쳐졌구나"로 읽으면 틀린다.

## 이번에 확인한 것

`open` / `merged: false` / `commits: 2`.

- 아직 승인 전이다
- 오후에 고쳐서 다시 푸시한 커밋이 이 PR에 들어가 있다(그래서 2개다)
- 그러니 PR을 새로 보낼 필요가 없다

## 화면과 뭐가 다른가

같은 것을 화면에서도 볼 수 있다. 다만 화면은 **표시를 해석해야** 한다.
빨간 X가 무엇의 X인지, 옆의 숫자가 무엇을 세는 숫자인지를 알아야 읽을 수 있다.

`"merged": false` 는 해석할 게 없다. 그냥 그 값이다.

**주의할 것 하나.** 로그인 없이 쓰는 GitHub API는 시간당 호출 횟수 제한이 있다.
확인은 한 번만 하고, 될 때까지 반복해서 부르지 않는다.
