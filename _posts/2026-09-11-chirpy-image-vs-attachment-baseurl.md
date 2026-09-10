---
title: "이미지와 첨부파일은 경로 규칙이 다르다 — blog_kate가 두 번 붙은 오류"
date: 2026-09-11 08:32:04 +0900
categories: [Jekyll]
tags: [jekyll, chirpy, baseurl, 트러블슈팅, 학습기록]
---

> 모두의연구소 AI 에이전트 과정 학습 기록입니다.
> Claude와의 대화 내용을 Claude가 초안으로 정리하고, 제가 검토·수정했습니다.
{: .prompt-info }

블로그에 이미지를 처음 넣었다. 그리고 그것 때문에 **빌드가 죽었다.**

## 오류 문구

검사 단계에서 이렇게 나왔다.

```
internal image /blog_kate/blog_kate/assets/files/ghost-message-example.png does not exist
```

`blog_kate` 가 **두 번** 붙어 있다. 이게 신호였다.

## `baseurl` 이 무엇인가

내 블로그 주소는 `사용자명.github.io` 가 아니라 `사용자명.github.io/blog_kate/` 다.
저장소 이름이 뒤에 붙는 형태라서, 사이트의 모든 경로 앞에 `/blog_kate` 를 붙여줘야 한다.
그 값을 `_config.yml` 에 적어둔 것이 `baseurl` 이다.

```yaml
baseurl: "/blog_kate"
```

그래서 파일을 가리킬 때 이 값을 앞에 붙여 쓴다 — **첨부파일은 그렇다.**

## 함정 — 테마가 이미지에만 자동으로 붙인다

첨부파일(pptx)은 전부터 이 값을 직접 붙여서 잘 통과했다.
그래서 이미지에도 똑같이 붙였는데, 그게 잘못이었다.

**Chirpy는 이미지 경로에는 `baseurl` 을 알아서 붙인다.**
거기에 내가 또 붙여서 두 번 들어간 것이다.

| 종류 | 쓰는 법 |
|---|---|
| 이미지 `![]()` | `/assets/files/파일명.png` — **붙이지 않는다** |
| 첨부파일 링크 `[]()` | {% raw %}`{{ site.baseurl }}/assets/files/파일명.pptx`{% endraw %} — **붙인다** |

같은 `assets/files/` 폴더에 있는 파일인데 쓰는 법이 다르다.
"파일 경로는 다 이렇게 쓴다"고 묶어서 외우면 틀린다.

## 고친 것

이미지 경로에서 {% raw %}`{{ site.baseurl }}`{% endraw %} 한 덩어리만 뺐다.

```markdown
![유령 메시지 화면](/assets/files/ghost-message-example.png)
```

커밋하고 push하니 빌드가 통과했고, 주소로 확인하니 글과 이미지 전부 `200` 이었다.

## 곁다리로 걸린 것 — 윈도우가 확장자를 숨긴다

이미지 파일 이름을 바꿀 때, 이름 칸에 `ghost-message-example.png` 라고 쓰면
실제 파일명이 **`ghost-message-example.png.png`** 가 된다.
윈도우가 확장자를 숨겨놓고 있어서, 화면에 보이는 이름 뒤에 확장자가 또 붙는 것이다.

이름을 바꾼 뒤에는 실제 이름을 확인하는 게 안전하다.

```bash
ls assets/files/
```

## 배운 것

- 잘 되던 규칙을 **비슷해 보이는 다른 것에 그대로 옮기지 않는다.** 이미지와 첨부파일은 다른 규칙이었다
- 오류 문구에 같은 글자가 두 번 붙어 있으면, **어딘가에서 자동으로 붙여주고 있다**는 뜻이다
- 파일명은 눈으로 보이는 것 말고 `ls` 로 확인한다
