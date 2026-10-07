---
title: "SDK가 claude.exe를 못 찾았다"
date: 2026-10-07 10:28:09 +0900
categories: [AI]
tags: [ClaudeAgentSDK, PATH, PowerShell, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

## 12번 모두 실패

실습 코드를 저장하고 PowerShell 에서 실행했다. 화면엔 "저장했습니다" 한 줄만 나왔는데, 결과 파일을 열어 보니 12번 모두 같은 오류였다.

```
CLINotFoundError: Claude Code not found. Install the native claude.exe ...
```

코드가 오류를 잡아 파일에 적고 다음으로 넘어가게 되어 있어서, 화면만 보면 성공한 것처럼 보였다.

## SDK 는 Claude Code 를 거쳐서 부른다

Claude Agent SDK 는 Claude 를 직접 부르지 않는다. 컴퓨터에 있는 Claude Code 프로그램(`claude.exe`)을 실행해서 그걸 통해 부른다. 그래서 `claude.exe` 가 없으면 아무것도 못 한다.

## 이미 있는데 왜 없다고 하나

나는 Claude Code 를 이미 데스크톱 앱으로 쓰고 있었다. 그런데 앱 안의 Claude Code 는 앱 안에서만 쓰인다. PowerShell 에서 `claude` 라는 명령으로 부를 수 없고, SDK 도 못 찾는다.

| 어디의 Claude Code | PowerShell 에서 `claude` |
|---|---|
| 데스크톱 앱 안 | 안 됨 |
| 우분투에 설치한 것 | 우분투 안에서만 |
| SDK 가 찾는 것 | 윈도우 PowerShell 에서 부를 수 있는 `claude.exe` |

## 설치하기 전에 물었어야 할 것

오류 메시지가 설치 명령을 알려 줘서 그대로 설치했다.

```powershell
irm https://claude.ai/install.ps1 | iex
```

나중에 보니 다른 길도 있었다. 오류 메시지에 이렇게도 적혀 있었다.

```
provide the path to a claude.exe via ClaudeAgentOptions:
  ClaudeAgentOptions(cli_path='C:\\path\\to\\claude.exe')
```

이미 컴퓨터 어딘가에 `claude.exe` 가 있다면 위치만 알려 주면 된다. "없다"는 오류가 나면 **새로 설치하기 전에, 이미 있는 걸로 해결되는지 먼저** 본다.

## 설치했는데 `claude` 가 안 된다

설치는 됐다. 위치는 `C:\Users\ctale\.local\bin\claude.exe`. 그런데 노란 경고가 같이 나왔다.

```
Native installation exists but C:\Users\ctale\.local\bin is not in your PATH.
```

PATH 는 "명령어를 찾을 폴더 목록"이다. PowerShell 은 이 목록에 있는 폴더에서만 `claude` 를 찾는다. 그래서 `Get-Command claude` 는 계속 실패했다.

신기하게도 SDK 는 코드를 안 고쳐도 이 위치의 `claude.exe` 를 찾아서 실행했다. PowerShell 에서 직접 `claude` 를 쓰려면 PATH 에 폴더를 넣는다.

```powershell
[Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path", "User") + ";C:\Users\ctale\.local\bin", "User")
```

이미 열려 있는 창에는 반영되지 않는다. PowerShell 창을 새로 열어야 한다.

## 그다음에 만난 것 둘

**멈춤.** 다시 돌렸더니 4번은 성공하고 5번째에서 멈췄다. SDK 가 Claude Code 와 연결을 시작하는 단계(`initialize`)에서 응답을 기다리다가 끝나지 않았다. Ctrl+C 로 끊었고, 원인은 기록에 남지 않았다. 다시 돌리니 12번 모두 성공했다.

**노란 경고.** 실행할 때마다 이 줄이 나왔다.

```
claude.ai connectors are disabled because ANTHROPIC_API_KEY or another auth source is set ...
```

"disabled" 라서 뭔가 꺼진 줄 알았는데, 꺼진 건 claude.ai 에 연결해 둔 외부 서비스(커넥터)다. 이 실험은 도구도 커넥터도 안 쓰니 상관없다. 오류가 아니라 안내였다.

## 남길 것

- 화면에 "완료"가 떠도 결과 파일을 열어 본다
- SDK 는 `claude.exe` 를 거쳐 돈다. 앱 안의 Claude Code 는 바깥에서 안 보인다
- "없다"는 오류엔 설치보다 먼저 "이미 있는가, 위치만 알려 주면 되는가"를 본다
- 설치 직후 PATH 경고가 나오면, 그 폴더를 PATH 에 넣고 창을 새로 연다
