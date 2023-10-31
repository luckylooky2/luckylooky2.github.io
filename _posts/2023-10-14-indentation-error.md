---
title: "[Github 블로그] Github Action 에러; did not find expected key while parsing a block mapping (Psych::SyntaxError)"
date: 2023-10-14
published: true
toc: true
toc_sticky: true
last_modified_at: 2023-10-23

categories:
  - Github-블로그
tags:
  - Error handling
---

### 1. 문제 발생

Github 블로그를 `minimal-mistakes`를 이용하여 생성하고, `_config.yml` 파일에서 블로그 세팅을 하였다.

프로필을 변경하고 commit, push를 하였는데 `Github Action`에서 다음과 같은 에러가 발생하였다.

```
github-pages 228 | Error:  (/github/workspace/./_config.yml): did not find expected key while parsing a block mapping at line 16 column 1

`parse': (/github/workspace/./_config.yml): did not find expected key while parsing a block mapping at line 16 column 1 (Psych::SyntaxError)
```

### 2. 원인

`_config.yml` 파일을 파싱하다가 생기는 문제라는 것을 추측할 수 있다.

다만, 16번째 줄을 확인하라는 메시지는 정확하지 않았다. 알고보니 파싱 에러가 발생한 위치는 65번 째 줄이었다.

원인은 `author`-`links`-`label`의 indentation(들여쓰기)이, 파일을 수정하는 과정에서 실수로 건드린 것 때문이었다.

`.yml` 파일에서는

- indentation의 위치가 같아야지만, 같은 레벨의 데이터라고 인식한다.
- 단, indentation의 개수는 상관이 없다고 한다. 일반적으로 2칸 또는 4칸을 사용한다.

`links` 블록 아래 `label`, `icon`, `url`은 같은 레벨의 데이터로써, 같은 indentation 위치에 있어야 했지만 그렇지 않아서 에러가 발생하였다.

### 3. 해결

```yml
# before
author:
  name: "First Lastname"
  avatar: "/assets/images/bio-photo.jpg"
  bio: "My awesome biography constrained to a sentence or two goes here."
  links:
 - label: "Email"
      icon: "fas fa-fw fa-envelope-square"
      url: mailto:example@example.com
    - label: "GitHub"
      icon: "fab fa-fw fa-github"
      url: "https://github.com/example"

# after
author:
  name: "First Lastname"
  avatar: "/assets/images/bio-photo.jpg"
  bio: "My awesome biography constrained to a sentence or two goes here."
  links:
    - label: "Email" # fix indentation
      icon: "fas fa-fw fa-envelope-square"
      url: mailto:example@example.com
    - label: "GitHub"
      icon: "fab fa-fw fa-github"
      url: "https://github.com/example"
```

indentation 문제는 위처럼 디버깅도 쉽지 않고 잘 눈에도 띄지 않기 때문에, 수정할 때 항상 주의가 필요할 것 같다!
