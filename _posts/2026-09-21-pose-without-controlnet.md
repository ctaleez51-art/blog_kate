---
title: "ControlNet 이 없는 모델에 포즈를 넣는 법 — ReferenceLatent"
date: 2026-09-21 09:07:44 +0900
categories: [AI]
tags: [이미지생성, ComfyUI, ControlNet, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

과제는 참조 사진의 자세를 가져와 다른 인물을 그리는 것이었다.
방법으로는 ControlNet 으로 포즈를 조건으로 주라고 안내돼 있었다.
그런데 쓰기로 한 모델인 **FLUX.2-klein 4B 에는 전용 ControlNet 이 없었다.**

## 검색으로 나온 건 전부 다른 모델용이었다

| 파일 | 대상 |
|---|---|
| `Flux.2-Klein-9B-MatchingPose` | klein **9B** |
| `refcontrol-FLUX.2-klein-9B-reference-pose-lora` | klein **9B** |
| `alibaba-pai/FLUX.2-dev-Fun-Controlnet-Union` | FLUX.2-**dev** |

이름에 `klein` 이 들어 있어서 처음엔 되는 줄 알았다. 자세히 보니 전부 9B 용이다.
같은 계열이라도 크기가 다르면 안 맞는다.

여기서 선택지는 둘이었다. **모델을 9B 로 바꾸든가, ControlNet 없이 해결하든가.**

## 빠져나갈 구멍은 모델 구조에 있었다

klein 은 텍스트로 그리는 것과 이미지를 참고해 그리는 것을 **한 모델에 합쳐** 놓았다.
즉 참조 이미지를 받는 입구가 원래부터 있다. ControlNet 이 하려던 일도 결국
"이 그림을 참고해라"인데, 그 통로가 이미 뚫려 있는 셈이다.

그래서 ComfyUI 기본 노드인 **`ReferenceLatent`** 로 뼈대 그림을 꽂았다.

```
LoadImage → VAEEncode → ReferenceLatent(conditioning, latent) → KSampler.positive
```

`ReferenceLatent` 는 프롬프트로 만든 조건에 **"이 그림도 참고해라"를 덧붙이는** 노드다.
ComfyUI 에 기본 내장이라 따로 설치할 게 없다. 설치가 필요 없다는 건
Colab 처럼 매번 환경을 새로 만드는 곳에서 특히 크다.

## ControlNet 을 아예 안 쓴 건 아니다

포즈 뼈대를 뽑는 건 `comfyui_controlnet_aux` 의 `OpenposePreprocessor` 를 썼다.
**ControlNet 본체는 안 쓰고 전처리기만 가져다 쓴 것**이다.

`comfyui_controlnet_aux` 는 이름이 ControlNet 이지만 안에 든 건 두 종류다.

- **전처리기** — 사진에서 포즈·윤곽선·깊이 같은 걸 뽑아내는 도구. 모델과 상관없이 쓸 수 있다
- **ControlNet 모델** — 뽑아낸 걸 조건으로 받아 그림을 만드는 쪽. 이건 모델마다 전용이 필요하다

앞쪽만 필요했고, 앞쪽은 어느 모델에나 쓸 수 있었다.

## 남길 것

- 도구가 없다고 할 때, **그 도구가 하려던 일**을 다시 정의해보면 다른 길이 보인다
- 패키지 이름에 매이지 않는다. 안에 뭐가 들었는지 갈라 보면 일부만 쓸 수 있다
- 대신 이 방식에는 대가가 있다. 색으로 좌우를 구분하는 OpenPose 규약이 전달되지 않는데,
  그건 다음 글에서 실제로 문제가 됐다
