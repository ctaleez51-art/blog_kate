---
title: "실습 두 개를 다 돌리는 파이썬 버전 고르기"
date: 2026-10-13 09:59:00 +0900
categories: [AI]
tags: [uv, 파이썬, MCP, colab-mcp, Qdrant, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

도구 설계 노드의 실습을 처음부터 다시 하면서, 가장 먼저 막힌 곳은 코드가 아니라 **파이썬 버전**이었다.
노드에는 파이썬 버전이 어디에도 적혀 있지 않다. 그런데 버전에 따라 실습이 돌기도 하고 안 돌기도 했다.

## 버전마다 무엇이 막혔나

| 버전 | 결과 | 이유 |
|---|---|---|
| 3.14 (윈도우 기본) | 5번 노드의 Qdrant 서버 설치 실패 | 필요한 패키지 하나에 3.14용 완성본이 없어서 직접 만들려다 실패 |
| 3.12 (uv가 받아 둔 것) | 2번 노드의 colab-mcp를 못 돌림 | colab-mcp가 3.13 이상을 요구 |
| 3.13 | 둘 다 됨 | |

### 3.14 — 완성본이 없으면 직접 만들려고 한다

`uvx mcp-server-qdrant --help`를 실행했더니 긴 오류가 났다. 핵심 줄은 두 개였다.

```
error: linker `link.exe` not found
```

```
hint: `pydantic-core` (v2.33.2) was included because `mcp-server-qdrant` (v0.8.1) depends on `pydantic` (v2.11.10) which depends on `pydantic-core`
```

- `mcp-server-qdrant 0.8.1` → `pydantic 2.11.10` → `pydantic-core 2.33.2` 순서로 딸려 들어온다.
- `pydantic-core 2.33.2`에는 파이썬 3.14용 완성본(바로 설치하는 파일)이 없다.
- 완성본이 없으면 소스에서 직접 만들려고 한다. 그러려면 Visual Studio C++ 빌드 도구(`link.exe`)가 필요한데 내 컴퓨터에는 없다.

### 3.12 — 프로그램이 정한 최소 버전

colab-mcp 폴더의 `pyproject.toml`에 이렇게 적혀 있다.

```
requires-python = ">=3.13"
```

3.12로는 2번 노드를 할 수 없다.

### 3.13 — 두 조건을 모두 만족

```powershell
uvx --python 3.13 mcp-server-qdrant --help
```

이건 패키지 71개를 설치하고 바로 실행됐다. colab-mcp도 3.13 이상이면 된다.

## 정한 것

- 노드의 모든 명령에 `--python 3.13`을 넣는다.
  - `uv run --python 3.13 --with ... python 파일명`
  - `uvx --python 3.13 mcp-server-qdrant`
- 3.14는 지우지 않는다. `python` 명령이 3.14에 연결되어 있어서, 지우면 `python`으로 직접 돌리던 다른 작업이 멈출 수 있다. uv가 받아 둔 3.12·3.13은 `python` 명령에 연결되어 있지 않다.

## 배운 것

- 한 실습만 보고 버전을 고르면 다른 실습에서 막힌다. **실습 전체의 조건을 같이 놓고** 골라야 한다.
- `uv run --directory 폴더`로 프로젝트를 돌리면 uv가 그 폴더 안에 `.venv`(그 프로젝트 전용 환경)를 만든다. 여러 프로젝트에서 쓸 프로그램은 어디에 둘지를 먼저 정해야 한다.
