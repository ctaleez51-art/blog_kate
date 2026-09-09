---
title: "Supabase CLI, 윈도우에서 걸린 것 넷"
date: 2026-09-09 09:32:53 +0900
categories: [보안]
tags: [supabase, cli, 환경변수, powershell, 트러블슈팅, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

Edge Function을 배포하려면 Supabase CLI를 써야 했다.
명령 자체는 몇 줄인데, 그 몇 줄에 닿기까지 네 군데서 걸렸다.
전부 윈도우 환경 때문이었다.

## 1. `npx` 가 실행 정책에 막힌다

```
npx : 이 시스템에서 스크립트를 실행할 수 없으므로 ... npx.ps1 파일을 로드할 수 없습니다
```

`npx` 는 `.ps1` 과 `.cmd` 두 형태로 깔려 있다.
PowerShell이 둘 중 `.ps1` 을 먼저 고르는데, 그게 스크립트 실행 정책에 막힌 것이다.

해결은 **`npx.cmd`** 를 직접 쓰는 것이었다.

```powershell
npx.cmd supabase --version
```

`.cmd` 는 PowerShell 스크립트가 아니라서 그 정책을 안 거친다.
**보안 설정을 바꿀 필요가 없다.** 검색하면 실행 정책을 풀라는 답이 많이 나오는데,
컴퓨터 설정을 건드리지 않고 파일 하나만 지정해서 끝났다.

## 2. `login` 이 성공했다는데 다음 명령이 인증 실패한다

```
You are now logged in. Happy coding!
```

이 문구가 떴는데 바로 다음 명령에서 인증이 안 됐다.
`~/.supabase` 폴더를 열어보니 `telemetry.json` 하나만 있고 **토큰 파일이 없었다.**

로그인은 됐는데 저장이 안 된 것이다. 성공 문구가 저장까지 보장하지는 않았다.

우회는 토큰을 직접 만들어 넘겨주는 것이었다.

- <https://supabase.com/dashboard/account/tokens> → `Generate new token`
- `Review access` 다음에 **`Create token`** 을 눌러야 값이 나온다.
  중간 화면에서 멈추면 토큰이 안 만들어진다
- 권한 선택지가 `No access` / `Read-only` / `Full access` 셋뿐이었다.
  배포와 시크릿 등록은 쓰기라서 `Full access` 가 유일한 선택이었다.
  대신 범위를 프로젝트 하나로 한정하고 만료를 7일로 뒀다

## 3. 환경변수는 그 창에서만 산다

토큰을 환경변수로 준다.

```powershell
$env:SUPABASE_ACCESS_TOKEN = "..."
```

**창을 닫으면 사라진다.** 이걸 몰라서 창이 바뀔 때마다 인증 실패가 반복됐다.
분명 방금 넣었는데 왜 또 안 되나 싶었던 게 전부 이것 때문이었다.

그래서 명령을 **한 줄로 묶는** 방식으로 정리했다.

```powershell
$env:SUPABASE_ACCESS_TOKEN = "..."; cd "프로젝트폴더"; npx.cmd supabase functions deploy interpret
```

값을 넣는 것과 쓰는 것이 같은 창에서 일어나니 창이 바뀌어도 상관없다.

디스크에 저장하는 방법도 있다.

```powershell
[Environment]::SetEnvironmentVariable("SUPABASE_ACCESS_TOKEN", "...", "User")
```

지우는 것은 같은 명령에 값 대신 `$null` 을 준다.
저장하면 창을 새로 열어도 남지만, **값이 디스크에 남는다.** 그 차이를 알고 고르는 것이다.

## 4. `secrets set` 은 따옴표로 묶어야 한다

Claude API 키를 Supabase 쪽에 등록하는 명령이다.

```powershell
npx.cmd supabase secrets set ANTHROPIC_API_KEY=sk-ant-...        # 조용히 실패
npx.cmd supabase secrets set "ANTHROPIC_API_KEY=sk-ant-..."      # 성공
```

**실패해도 오류가 안 난다.** 아무 반응 없이 다음 줄로 넘어간다.
성공하면 `Finished supabase secrets set.` 이 뜬다. 이 문구가 없으면 안 된 것이다.

확인은 목록으로 한다.

```powershell
npx.cmd supabase secrets list
```

이름과 지문(해시)만 보이고 값은 안 보인다. 값이 안 보이는 게 정상이고,
**목록이 비어 있으면 실패한 것**이다.

## 덧 — `link` 는 프로젝트 폴더에서 해야 한다

홈 폴더에서 `link` 를 하면 `C:\Users\ctale\supabase\.temp` 가 거기에 생긴다.
배포 명령은 **현재 폴더 기준으로** 함수 파일을 찾기 때문에,
그 상태로 배포하면 함수를 못 찾는다.

`supabase/.temp/` 는 CLI가 만든 로컬 상태 파일이라 `.gitignore` 에 넣었다.

## 넷을 겪고 남은 것

네 개가 다른 문제 같지만 공통점이 있었다.
**성공 문구가 떴다고 성공한 게 아니었다는 것.**

`login` 은 성공했다고 했는데 토큰이 없었고, `secrets set` 은 아무 말 없이 실패했다.
그래서 확인하는 명령을 따로 아는 게 중요했다.
`~/.supabase` 폴더를 열어보는 것, `secrets list` 를 쳐보는 것.
