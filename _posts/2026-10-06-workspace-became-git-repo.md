---
title: "입력창 위에 master 가 뜬 이유 — 작업 폴더가 통째로 git 저장소가 됐다"
date: 2026-10-06 10:10:17 +0900
categories: [Git]
tags: [git, git-init, claude-code, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

Claude Code 데스크톱 앱 입력창 위에 원래 없던 막대 두 개가 계속 떴다.

- `로컬 · claudecode · master · 워크트리`
- `claudecode master +7,192 -0 변경 사항 커밋` (× 로 닫아도 다시 뜸)

## 원인 — 작업 폴더 자체에 `.git` 이 생겼다

나는 작업 폴더(`claudecode`) 안의 **프로젝트 폴더마다** git 을 써 왔다. 작업 폴더 **자체**에는 git 이 없었다.

그런데 9월 29일 오전 11:38 에 작업 폴더 자체에 `.git` 이 생겼다. 앱은 작업 폴더만 보기 때문에 그때부터

- 작업 폴더를 git 저장소로 알아보고 브랜치 이름 `master` 를 띄웠고
- 커밋이 하나도 없으니 폴더 안 전부가 "커밋 안 한 변경"으로 잡혀 `+7,192` 가 떴다

같은 날 11:04 에 프로젝트 폴더 하나에 git 이 만들어졌고, 34분 뒤에 작업 폴더에 생겼다. 2분 뒤에는 `.gitignore` 도 생겼다. **한 단계 위 폴더에서 `git init` 이 실행된 것**으로 보인다.

## 확인한 방법 (PowerShell, 읽기만)

```powershell
# 작업 폴더의 .git 이 언제 생겼나
Get-Item -Force .git | Select-Object CreationTime, LastWriteTime

# 어느 프로젝트 폴더에 .git 이 있고 각각 언제 생겼나
Get-ChildItem -Directory | Where-Object { Test-Path (Join-Path $_.FullName '.git') } | ForEach-Object { $g = Get-Item -Force (Join-Path $_.FullName '.git'); "{0}  {1}" -f $_.Name, $g.CreationTime }
```

`.git` 은 숨김 폴더라 탐색기에 기본으로 안 보인다. **보기 → 표시 → 숨긴 항목** 을 켜거나 `Get-ChildItem -Force` 로 본다.

## 되돌린 것

작업 폴더 자체의 `.git` 과 `.gitignore` 만 내가 직접 지웠다. 커밋이 0개라 잃는 기록은 없었다.

```powershell
git -C "<작업 폴더>" rev-parse --show-toplevel
# fatal: not a git repository  ← 이제 저장소가 아니다
```

프로젝트 폴더들의 `.git` 은 그대로 남아 있는 것도 확인했다.

## 남길 것

- 앱에 `master` 가 뜨면 **작업 폴더 자체에 `.git` 이 있는지**부터 본다
- `git init` 할 때는 **지금 어느 폴더에 있는지** 확인한다. 한 칸 위에서 치면 작업 폴더 전체가 저장소가 된다
- 처음에 앱 기능 설명만 듣다가 원인을 늦게 찾았다. **폴더를 직접 확인하는 게 먼저**였다
