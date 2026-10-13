---
title: "opencode가 안 켜질 때, 그리고 모델 바꾸기"
date: 2026-10-13 09:59:02 +0900
categories: [AI]
tags: [opencode, PowerShell, API키, OpenAI, Gemini, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

검색 도구 실습을 하려고 opencode를 켜는 데서 두 번 막혔다.

## opencode.ps1이 막힐 때

PowerShell에서 `opencode`를 쳤더니 이런 오류가 났다.

```
opencode : 이 시스템에서 스크립트를 실행할 수 없으므로 C:\Users\ctale\AppData\Roaming\npm\opencode.ps1 파일을 로드할 수 없습니다. 자세한 내용은 about_Execution_Policies(https://go.microsoft.com/fwlink/?LinkID=135170)를 참조하십시오.
```

- PowerShell의 보안 설정(실행 정책)이 `.ps1` 파일을 못 돌리게 막은 것이다.
- npm은 같은 프로그램을 `.cmd` 파일로도 만들어 둔다. 보안 설정을 바꾸지 않고 이걸로 켤 수 있다.

```powershell
opencode.cmd
```

## opencode가 쓰는 AI 바꾸기

켜 보니 opencode가 쓰는 AI가 Claude Sonnet 4.6이었다. 실습 폴더의 `opencode.json`에는 AI를 정하는 칸이 없어서 opencode 자체 설정을 따른 것이다.
Claude Code도 Claude라서, 다른 AI로 비교하려고 바꿨다.

1. 입력칸에 `/models` → AI 목록 창이 뜬다. 목록에 Claude만 있으면 다른 회사는 연결이 안 된 것이다.
2. **그 창이 떠 있는 상태에서** `ctrl`+`a`(Connect provider). 창을 닫으면 안 된다.
3. 회사를 고른다.

| 회사 | 연결 방법 | 돈 |
|---|---|---|
| 구글(제미나이) | https://aistudio.google.com/apikey 에서 키를 받아 넣는다 | 무료로 쓸 수 있는 양이 있다 |
| OpenAI | "ChatGPT Pro/Plus (browser)": 구독이 있으면 로그인으로 / "Manually enter API Key": https://platform.openai.com/api-keys 에서 받은 키로 | API 키는 충전해야 쓸 수 있다 |

- OpenAI 키 페이지에 들어가니 키가 바로 만들어졌다. 키는 만들 때 한 번만 보인다.
- OpenAI로 연결해 GPT-6 Sol(high)을 골랐더니 느렸다. "high"는 오래 생각하는 설정이다. GPT-6 Sol Fast(default)로 다시 했다.
- 쓴 돈은 opencode 화면 오른쪽 아래에 찍힌다. 이번 실습에서 약 $0.13이었다.

## 배운 것

- 다른 회사 AI를 붙이려면 그 회사의 키(또는 구독 로그인)가 필요하고, 돈이 드는지 먼저 확인해야 한다.
- 무료로 해 보려면 제미나이 키가 낫다.
