---
title: "첫 커밋 전에 확인한 것 — 키, 내 이름이 든 경로, 깨진 커밋 명령"
date: 2026-11-09 10:08:31 +0900
categories: [Git]
tags: [Git, 보안, PowerShell, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

## 저장소부터 없었다

종합소득세 딥리서처 폴더에는 깃 저장소가 없었다. `git init -b main` 으로 만들고 로컬 커밋을 했다. 푸시와 깃허브 저장소 만들기는 아직 하지 않았다.

## 키가 들어가지 않았는지

- `.env` 는 `.gitignore` 에 있어서 커밋에서 빠졌다.
- `git grep --cached` 로 OpenAI 키 모양(`sk-`)을 찾아봤다. 나온 것은 README 의 `sk-...` 예시뿐이었다.

## 내 사용자 이름이 든 경로

README 에 내 컴퓨터의 사용자 폴더 경로가 그대로 들어 있었다. 경로에 내 사용자 이름이 들어 있다. 실행 방법에는 저장소 폴더만 있으면 되니 `cd tax-deep-researcher` 로 바꿨다.

## here-string 으로 넘긴 커밋 메시지가 깨졌다

PowerShell 에서 `git commit -F -` 에 here-string 을 넘겼다. 메시지 전체가 파일 이름(pathspec)으로 읽혀 커밋이 실패했다.

메시지를 파일로 저장한 뒤 그 파일을 넘겨서 커밋했다.

```
git commit -F 파일경로
```

## `git grep` 은 옵션을 앞에

`--cached` 같은 옵션을 검색어 뒤에 쓰면 이렇게 나온다.

```
option '--cached' must come before non-option arguments
```

`git grep --cached 검색어` 처럼 옵션을 검색어보다 앞에 써야 한다.

## 정리

- 나는 첫 커밋 전에 `.env` 가 빠졌는지, 키 모양 글자가 없는지 `git grep --cached` 로 확인한다.
- 문서 속 경로에 내 사용자 이름이 들어 있으면 뺀다.
- PowerShell 에서 여러 줄 커밋 메시지는 파일로 저장해서 `-F 파일경로` 로 넘긴다.
