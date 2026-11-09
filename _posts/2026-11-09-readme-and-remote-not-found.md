---
title: "교안에 없는 README 와 'Repository not found' — 과제 제출과 깃허브 올리기"
date: 2026-11-09 10:06:48 +0900
categories: [Git, GitHub]
tags: [GitHub, README, SSH, 과제제출, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

## 제출물

라우팅 에이전트 과제 저장소에 들어간 것이다.

- `docs/`, `data/goldenset.json`
- `context.py`, `prompts.py`, `agent.py`, `evaluate.py`, `app.py`
- `requirements.txt`, `.gitignore`
- `PRD.md`, `REPORT.md`, `README.md`
- `results/`, 캡처 2장

REPORT.md 는 교안이 정한 7가지 항목으로 썼다. 주제와 문서, 카테고리 설계, 평가셋 설계, 측정 결과, 파이프라인 구조도, 데모 설계, 회고다. 구조도는 Mermaid 로 그렸다.

## 교안 구조에 README 가 없었다

교안이 보여 준 폴더 구조에는 README.md 가 없다. 그런데 깃허브 저장소 첫 화면에는 README.md 만 자동으로 보인다. 없으면 파일 목록만 보인다. 채점자가 저장소를 열었을 때 무엇을 봐야 할지 모르게 된다.

그래서 README.md 를 따로 만들었다. 넣은 것은 이렇다.

- 과제 소개
- 결과 요약표
- 파일 설명
- 실행 방법
- REPORT.md 링크

## 캡처를 하나만 넣었다가

README 에 처음엔 1.png 만 넣었다. 그런데 1.png 에는 질문과 판정만 있고 **답이 없다.** 이 에이전트가 무엇을 답하는지 첫 화면에서 보이지 않는다. 답이 보이는 2.png 도 같이 넣었다.

## 저장소 만들기

저장소는 github.com/new 에서 직접 만들었다. Repository name 만 넣고 README 추가는 체크하지 않았다.

## `아이디` 를 글자 그대로 넣었다

안내받은 명령은 이 모양이었다.

```
git remote add origin https://github.com/아이디/tax-routing-agent.git
```

`아이디` 는 내 깃허브 계정 이름으로 바꿔 넣으라는 자리인데, 글자 그대로 넣었다. push 하자 이렇게 나왔다.

```
remote: Repository not found.
```

`아이디` 라는 계정의 저장소를 찾았으니 없는 게 당연하다. 내 다른 저장소들은 SSH 주소(`git@github.com:계정/저장소.git`)를 쓰고 있어서, 같은 방식으로 주소를 바꿨다.

```
git -C C:\Users\ctale\Desktop\claudecode\tax-routing-agent remote set-url origin git@github.com:ctaleez51-art/tax-routing-agent.git
```

## 올라갔는지 확인

- `git status -sb` 첫 줄이 `## main...origin/main` 이고 남은 변경이 없으면 push 된 것이다.
- `.env` 와 `__pycache__` 가 올라가지 않았는지 `git ls-files` 로 확인했다.

## 정리

- 나는 교안의 폴더 구조에 없더라도 저장소 첫 화면에 필요한 README 를 만든다. 첫 화면 캡처에는 답이 보여야 한다.
- 명령어 속 `아이디` 같은 자리 표시는 내 값으로 바꿔 넣는다.
- 터미널 명령은 지금 있는 폴더에 기대지 않고 `git -C <경로>` 처럼 경로를 명령 안에 넣는다.
