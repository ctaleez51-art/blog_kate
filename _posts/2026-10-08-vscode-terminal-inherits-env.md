---
title: "VS Code 터미널을 새로 열어도 환경 변수가 없었다"
date: 2026-10-08 11:03:08 +0900
categories: [AI]
tags: [환경변수, setx, PowerShell, VSCode, Gemini, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

## 증상

Gemini 경로로 실습 코드를 돌리려고 API 키를 환경 변수로 저장했다.

```powershell
setx GOOGLE_API_KEY "키"
```

"성공: 지정한 값을 저장했습니다"가 나왔다. 그런데 확인하면 계속 `False` 였다.

```powershell
[bool]$env:GOOGLE_API_KEY
```

## setx 는 저장만 한다

`setx` 는 값을 컴퓨터의 사용자 설정에 **저장만** 한다. 프로그램은 켜질 때 설정을 한 번 읽어 오고, 그 뒤에 저장된 값은 모른다. 그래서 이미 열려 있던 터미널에는 반영되지 않고 새로 여는 터미널부터 적용된다.

여기까지는 9월 28일에 겪은 것과 같다. 처음엔 같은 터미널에서 확인해서 `False` 였다.

## 새 터미널도 False 였다

그래서 VS Code 안에서 터미널을 새로 열었다. 또 `False` 였다.

**VS Code 안의 터미널은 새로 열어도 VS Code 가 켜질 때 읽은 설정을 물려받는다.** 터미널을 여는 주인이 VS Code 이기 때문이다. 키를 저장하기 전에 켜 둔 VS Code 라서, 그 안에서 몇 번을 새로 열어도 키가 없었다. Claude 앱 안의 터미널도 마찬가지다.

| 어디서 연 터미널 | 새 값을 아는가 |
|---|---|
| `setx` 를 친 그 터미널 | 모른다 |
| 그 VS Code 안에서 새로 연 터미널 | 모른다 (VS Code 가 켜질 때 읽은 값) |
| VS Code 를 완전히 껐다 켠 뒤의 터미널 | 안다 |

## 해결: 둘 중 하나

1. VS Code 창을 모두 닫았다가 다시 켠다
2. 저장된 값을 지금 터미널로 불러온다

나는 2번으로 VS Code 를 끄지 않고 해결했다. 키 값은 화면에 나오지 않는다.

```powershell
$env:GOOGLE_API_KEY = [Environment]::GetEnvironmentVariable('GOOGLE_API_KEY','User'); [bool]$env:GOOGLE_API_KEY
```

저장이 됐는지만 확인할 때는 이렇게 본다.

```powershell
[bool][Environment]::GetEnvironmentVariable('GOOGLE_API_KEY','User')
```

- 앞의 `$env:` 는 **지금 이 터미널**이 알고 있는 값
- 뒤의 `GetEnvironmentVariable(..., 'User')` 는 **컴퓨터에 저장된** 값

둘이 다르면 저장은 됐는데 터미널이 아직 모르는 상태다.

## .env 파일은 읽는 줄이 있어야 읽힌다

수업에서는 `.env` 파일에 키를 적었다. 그런데 `.env` 는 파일만 있다고 읽히지 않는다. 코드에 `load_dotenv()` 같은 **읽는 줄**이 있어야 한다.

`prompt_lab.py` 에는 그 줄이 없어서 환경 변수로 넣었다. `load_dotenv()` 줄은 항상 넣어 둬도 된다. Claude 나 Ollama 로 실행할 때는 영향이 없다.

## 내 앱에서는

실제 앱에서는 사용자가 키를 등록하지 않는다. 키는 만든 사람의 서버(예: Vercel 환경 변수)에 두고, 브라우저는 서버에 질문만 보낸다. 오늘 한 것은 내 컴퓨터에서 실습 코드를 돌리기 위한 방법이다.

## 덤: 터미널 입력 줄 한 번에 지우기

Esc (또는 Ctrl+C). 붙여 넣은 내용에 줄바꿈이 있으면 바로 실행돼 버리니 조심한다.

## 남길 것

- `setx` 뒤에는 터미널만이 아니라 **터미널을 연 프로그램**까지 새로 켜야 한다
- 저장된 값과 지금 터미널의 값을 따로 확인한다
- `.env` 를 쓰려면 코드에 읽는 줄이 있는지 먼저 본다
