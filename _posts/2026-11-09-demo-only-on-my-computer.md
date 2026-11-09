---
title: "데모가 내 컴퓨터에서만 열린다 — localhost 주소와 채점자가 보는 방법"
date: 2026-11-09 10:08:31 +0900
categories: [AI]
tags: [Streamlit, localhost, 데모, 과제제출, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

## localhost 는 "이 컴퓨터"

종합소득세 딥리서처 데모는 Streamlit 이고, 주소가 `http://localhost:8501` 이다. localhost 는 "이 컴퓨터"라는 뜻이다. 채점자가 이 주소를 열면 채점자 자기 컴퓨터를 찾는다. 그래서 채점자는 이 주소로 내 데모에 들어올 수 없다.

## 채점자가 직접 보려면

채점자 자기 컴퓨터에서 차례로 이렇게 한다.

```bash
git clone https://github.com/ctaleez51-art/tax-deep-researcher.git
```

```bash
cd tax-deep-researcher
```

```bash
pip install -r requirements.txt
```

```bash
streamlit run app.py
```

## 키가 없어도 열리는 것

- "저장된 실행" 보기는 키 없이 열린다.
- 키(`OpenAIModel`)는 새 질문을 실행할 때만 읽는다. 새 질문을 돌리려면 채점자 본인의 OpenAI 키를 `.env` 에 넣어야 한다.

## 실행하지 않아도 보이는 것

깃허브 화면에서 바로 읽히는 것도 있다.

- README 의 캡처
- `REPORT.md`
- `output/reports/` 의 보고서 md

## 정리

- 나는 데모 주소가 localhost 면 남에게 링크로 줄 수 없다는 것부터 확인한다.
- 대신 남이 자기 컴퓨터에서 여는 방법과, 실행하지 않아도 볼 수 있는 것을 같이 적어 둔다.
