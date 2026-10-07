---
title: "챗봇에게 폴더 권한을 하나씩 주기"
date: 2026-10-07 10:28:17 +0900
categories: [AI]
tags: [ClaudeAgentSDK, 권한, Python, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

## 과제

`claude-agent-sdk` 로 가장 단순한 챗봇을 만들고, 챗봇에게 폴더를 만들고 지우게 시키면서 막힐 때마다 코드를 최소한으로 고치는 실습이다. VS Code 로 코드가 어떻게 달라지는지 보면서 했다.

처음엔 한 번에 끝까지 돌린 결과만 받아 봤는데, 그러면 내가 배운 게 없었다. 폴더를 지우고 처음부터 다시, 한 단계씩 내가 직접 고쳤다.

## 처음 코드

설정이 하나도 없다. 입력을 받아 보내고, 답의 글만 출력한다.

```python
import asyncio
from claude_agent_sdk import query, AssistantMessage, TextBlock


async def main():
    while True:
        prompt = input("나: ")
        async for message in query(prompt=prompt):
            if isinstance(message, AssistantMessage):
                for block in message.content:
                    if isinstance(block, TextBlock):
                        print("Claude:", block.text)


asyncio.run(main())
```

매번 `query()` 를 새로 부르니 앞 대화를 기억하지 않는다.

## 1. 폴더 만들기 — 실패

"현재 폴더 안에 sub1 이라는 하위 폴더를 만들어 줘"라고 했다. 챗봇은 PowerShell 로 `New-Item` 을 두 번 시도했고, 둘 다 막혔다.

원인은 두 겹이었다.

- 명령을 실행하려면 사람 허락이 필요한데, 이 챗봇에는 허락할 화면이 없다. 그래서 자동으로 거절된다
- Claude Code 의 PowerShell 도구가 작업 폴더 안의 `New-Item` 도 "허용된 폴더 밖"이라며 막는다. 허락을 줘도 이 검사에 걸렸다

**Bash 와 PowerShell.** 둘 다 명령어를 입력하는 창(셸)이다. PowerShell 은 윈도우 것이고 폴더 만들기가 `New-Item`, Bash 는 리눅스(우분투) 것이고 `mkdir` 이다. Claude Code 에는 둘 다 명령 실행 도구로 들어 있다.

**고친 것.** 잘못 막는 PowerShell 도구는 끄고, Bash 의 `mkdir` 만 허락 없이 실행하게 했다.

- 맨 위 import 줄 끝에 `, ClaudeAgentOptions` 추가
- `query(prompt=prompt)` 의 `prompt=prompt` 뒤에 설정 추가

```python
query(prompt=prompt, options=ClaudeAgentOptions(
    allowed_tools=["Bash(mkdir:*)"], disallowed_tools=["PowerShell"]))
```

| 설정 | 뜻 |
|---|---|
| `allowed_tools` | 묻지 않고 써도 되는 도구·명령 |
| `disallowed_tools` | 아예 못 쓰게 막는 도구 |

다시 시키자 `sub1` 이 생겼다.

이 답을 찾기 전에 안 된 것도 있다. `permission_mode="acceptEdits"`(파일 수정 자동 허락)는 명령 실행이 "파일 수정"에 들지 않아 안 먹었다. `PowerShell(New-Item:*)` 허락은 승인은 통과했지만 경로 검사에 걸렸다. `Bash(mkdir:*)` 만 허락하면 챗봇이 계속 PowerShell 을 골랐다. 원인은 SDK 가 보내는 메시지를 전부 출력해 보며 찾았다. `permission_denied` 메시지에 막힌 이유가 그대로 적혀 있다.

## 2. 폴더 지우기 — 실패

"sub1 폴더를 삭제해 줘". 챗봇은 `rmdir`(빈 폴더 지우기)로 지우려다 승인 요구에 막혔다. 허락 목록에 `mkdir` 만 있었기 때문이다.

**고친 것.** `"Bash(mkdir:*)"` 뒤에 `, "Bash(rmdir:*)"` 추가. 다시 시키자 지워졌다.

`rm` 을 넣을 줄 알았는데 챗봇은 `rmdir` 를 썼다. 무엇을 허락할지는 챗봇이 실제로 쓴 명령을 보고 정한다.

## 3. 삭제는 무조건 승인받기

허락 목록에 `rmdir` 가 있으면 묻지 않고 지운다. 지우기 전에는 꼭 내게 묻게 하고 싶었다. 메시지를 넣을 곳은 **승인 함수**(`can_use_tool`)다. 챗봇이 허락 목록에 없는 도구를 쓰려 할 때 SDK 가 이 함수를 부른다.

순서대로 고쳤다.

1. 허락 목록에서 `, "Bash(rmdir:*)"` 를 다시 뺀다. 목록에 있으면 승인 함수까지 안 온다
2. import 줄 끝에 `, PermissionResultAllow, PermissionResultDeny` 추가
3. `async def main():` 바로 위에 승인 함수를 넣는다

```python
async def ask_approval(tool_name, tool_input, context):
    answer = input(f"[승인 요청] {tool_name} {tool_input} 실행할까요? (y/n): ")
    if answer == "y":
        return PermissionResultAllow()
    return PermissionResultDeny(message="사용자가 거부했습니다.")
```

4. 설정 끝에 `, can_use_tool=ask_approval` 을 붙여 연결한다. 함수를 만들기만 해서는 SDK 가 모른다

시험해 봤다.

```
나: sub1 폴더를 만들어 줘
Claude: TEST 폴더 안에 sub1 폴더를 만들었습니다.
나: sub1 폴더를 삭제해 줘
[승인 요청] Bash {'command': 'rmdir "C:/.../TEST/sub1" && ls -la "C:/.../TEST"', ...} 실행할까요? (y/n): y
Claude: TEST\sub1 폴더를 삭제했습니다.
```

만들기는 묻지 않고, 지우기는 물었다. 지우기와 확인(`ls`)이 `&&` 로 한 줄에 묶여 있어서 승인 요청도 한 번만 떴다.

시스템 프롬프트에 "삭제 전에 물어봐"라고 쓰는 방법도 있다. 하지만 그건 모델이 따르느냐에 달렸다. 승인 함수는 모델이 무시해도 프로그램이 막는다.

## 4. 파일·폴더 작업을 전부 미리 막으려면

도구를 하나도 주지 않으면 된다. `ClaudeAgentOptions(` 바로 뒤에 `tools=[], ` 를 넣었다.

```
나: sub1 폴더를 만들어 줘
Claude: 지금 이 대화에는 명령을 실행할 도구가 연결되어 있지 않아서 폴더를 직접 만들 수 없습니다.
```

허락 목록이나 승인 함수가 있어도, 쓸 도구가 없으니 실행할 방법 자체가 없다. 주 실습 코드에 `tools=[]` 가 들어 있던 이유가 이것이었다.

## 대화를 기억 못 해서 생긴 일

중간에 챗봇이 "진행할까요?"라고 말로 물은 적이 있다. "응"이라고 답해 봐야 소용없다. 매번 `query()` 를 새로 부르니 "응"만 새 질문으로 간다. 거부(`n`)를 보냈을 땐 "Your message was just n"이라고 답했다.

이 구조에서는 챗봇이 말로 묻는 승인은 이어지지 않는다. 승인을 프로그램 안에서 받아야 하는 이유가 하나 더 생겼다.

## 정리

| 단계 | 바꾼 설정 |
|---|---|
| 만들기 | `allowed_tools=["Bash(mkdir:*)"]`, `disallowed_tools=["PowerShell"]` |
| 지우기 | `"Bash(rmdir:*)"` 추가 |
| 지우기 전 승인 | `rmdir` 빼고 `can_use_tool=ask_approval` |
| 전부 막기 | `tools=[]` |

## 남길 것

- 막히면 메시지를 전부 출력해 본다. 막힌 이유가 거기 적혀 있다
- 허락은 챗봇이 실제로 쓴 명령에 맞춘다(`rm` 이 아니라 `rmdir`)
- 허락 목록에 있으면 승인 함수까지 안 온다. 물어야 하는 건 목록에서 뺀다
- 반드시 막아야 하는 건 지시문이 아니라 프로그램에서 막는다
