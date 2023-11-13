---
title: "[React][Tailwindcss] CSS 유틸리티 클래스와 관심사 분리; className, SoC"
date: 2023-11-13
published: true
toc: true
toc_sticky: true
last_modified_at: 2023-11-13

categories:
  - React
tags:
  - tailwindcss
  - CSS
  - seperation of concerns
---

_"이 글은 [CSS Utility Classes and "Separation of Concerns"](https://adamwathan.me/css-utility-classes-and-separation-of-concerns/) 글을 번역하여 해석한 글입니다. 해석하는 과정에서 주관적인 견해가 포함되었을 수 있습니다."_

## 본문

지난 수 년동안 나의 CSS를 작성하는 방법이 "의미론적(semantic) CSS"에서 "기능적(functional) CSS"로 변화하였습니다. 기능적인 CSS를 사용하는 것에 어떻게 도달했고, 어떤 통찰력과 깨달음이 있었는지를 공유하려고 합니다.

- 의미론적(semantic) CSS

  - 클래스나 선택자의 이름을 HTML의 의미나 구조에 따라 지정하는 방식입니다.
  - HTML 요소가 스타일에 어떤 의미를 지니는지 파익하기 쉽고, 가독성이 좋을 수 있습니다.

- 기능적(functional) CSS

  - 작고 단일한 클래스들을 사용하여 스타일을 지정하는 방식입니다.
  - 클래스 이름은 해당 클래스가 하는 기능을 설명하거나, 특정한 스타일을 가리킵니다.
  - HTML 구조와 독립적으로 디자인을 적용할 수 있으며, 작은 단위의 클래스를 조합하여 디자인을 구현할 수 있습니다.

### 1. 의미론적(semantic) CSS

CSS를 잘 배우는 방법 중 하나는 "관심사를 분리"하는 것이라고 익히 들어왔을 것입니다. 다시 말해서, HTML을 작성할 때는 컨텐츠에 구애받지 않는 클래스에 대한 정보만 포함되어 있어야 하고, 스타일에 관한 정보는 CSS 파일에 있어야 한다는 것입니다.

> 관심사 분리(separation of concerns, SoC)는 컴퓨터 프로그램을 구별된 부분으로 분리시키는 디자인 원칙으로, 각 부문은 개개의 관심사를 해결한다.

아래와 같은 방식으로 HTML을 작성하는 것은, 스타일에 관한 정보를 CSS로 분리시키지 않고 HTML에서 직접적으로 다루고 있기 때문에 "관심사를 분리"하는 규칙을 어긴 것입니다.

```html
<style>
  .text-center {
    text-align: center;
  }
</style>

<p class="text-center">Hello there!</p>
```

반면에 아래와 같이 클래스 이름을 스타일과 관련없이 짓는다면, HTML 자체를 수정하지 않고도 스타일을 수정할 수 있습니다. 즉, 관심사를 분리할 수 있습니다. "관심사을 분리"하면 **스타일시트를 교체하는 것만으로도 사이트를 완전히 다시 디자인할 수 있다**는 장점이 있습니다. 또한, HTML과 CSS의 관계를 유연하게 만들어 유지보수를 쉽게 합니다.

```html
<style>
  .greeting {
    text-align: center;
  }
</style>

<p class="greeting">Hello there!</p>
```

하지만 의미론적(semantic) CSS를 적용하여 관심사를 분리하여도, HTML과 CSS 사이에는 분명한 결합이 존재합니다. 아래 CSS 코드에서 `.profile` 클래스 하위의 `img`, `p` 태그는 HTML 구조를 여전히 반영하고 있습니다. 다시 말해, HTML은 CSS에 관심이 없지만 **CSS는 HTML 구조에 매우 관심이 많았습니다.** 의미론적(semantic) CSS를 적용하는 방법이 어쩌면 관심사를 분리하지 않았을 수도 있습니다.

```css
.profile {
  background-color: white;
}

.profile img {
  display: block;
  width: 100%;
  height: auto;
}

.profile div p {
  font-size: 1rem;
}
```

```html
<div class="profile">
  <img src="https://cdn.example.com/chanhyle" alt="profile" />
  <div>
    <h2>chanhyle</h2>
    <p>hello world!</p>
  </div>
</div>
```

### 2. CSS에서 구조를 분리

CSS 파일에서 HTML 구조를 반영하는 문제점을 해결하기 위해, 선택자가 HTML 태그를 직접 가리키도록 클래스를 추가해 보았습니다. Block Element Modifier(BEM) 라고도 부릅니다.

```css
.profile {
  background-color: white;
}

.profile_image {
  display: block;
  width: 100%;
  height: auto;
}

.profile_body {
  font-size: 1rem;
}
```

```html
<div class="profile">
  <img
    class="profile_image"
    src="https://cdn.example.com/chanhyle"
    alt="profile"
  />
  <div>
    <h2>chanhyle</h2>
    <p class="profile_body">hello world!</p>
  </div>
</div>
```

HTML은 이미 스타일에 관심이 없었고, CSS 또한 HTML 구조로부터 자유로워졌기 때문에 관심사가 분리되었다고 할 수 있습니다. 하지만 이 방법에서는 또 다른 문제가 있습니다. 바로 재사용의 문제입니다.

예를 들어, `profile` 클래스와 매우 비슷한 스타일이 비슷한 `article` 클래스를 구현해야 하는 경우입니다. 방법은 크게 두 가지가 있을 것입니다.

첫 번째는 그대로 복사하는 방법입니다. 여전히 관심사는 분리되었지만 재사용이 전혀 되지 않았습니다. 만약 스타일을 일괄적으로 변경해야 하는 경우, 유지보수성이 떨어집니다.

```css
.article {
  background-color: white;
}

.article_image {
  display: block;
  width: 100%;
  height: auto;
}

.article_body {
  font-size: 1rem;
}
```

```html
<div class="article">
  <img
    class="article_image"
    src="https://cdn.example.com/article1"
    alt="article"
  />
  <div>
    <h2>This is first post.</h2>
    <p class="article_body">hello world!</p>
  </div>
</div>
```

두 번째는 추상화하는 방법입니다. `profile` 과 `article` 은 의미론적 관점에서는 공통점이 없지만, 스타일 관점에서는 공통점이 존재합니다. 따라서 두 클래스를 `media-card` 라는 컨텐츠에 구애받지 않는 클래스(content-agnostic component) 로 추상화하여 재사용성을 확보할 수 있습니다.

```css
.media-card {
  background-color: white;
}

.media-card_image {
  display: block;
  width: 100%;
  height: auto;
}

.media-card_body {
  font-size: 1rem;
}
```

재사용성 문제는 해결했지만, 이 방법은 관심사가 다시 결합될 수 있는 문제점이 존재합니다. 두 클래스의 스타일이 미래에도 완전히 같다면 다행이겠지만, 둘 중 하나를 변경해야 한다면 어떻게 될까요? 결국에는 HTML에 새로운 클래스를 추가해야 할 것입니다. 즉, **CSS 파일 안에서 해결하지 못하기 때문에 관심사가 다시 결합되었다고 볼 수 있습니다.**

```html
<div class="media-card">
  <img
    class="media-card_image"
    src="https://cdn.example.com/chanhyle"
    alt="media-card"
  />
  <div>
    <h2>chanhyle</h2>
    <p class="media-card_body profile_body">hello world!</p>
  </div>
</div>
```

> 결국, "관심사의 분리"만 핵심이 아니다!

HTML과 CSS의 관계를 관심사 분리의 대상으로 본다면, 흑백논리일 수 밖에 없습니다. "관심사가 분리되었다면 좋고, 분리되지 않았다면 나쁘다"는 것은 올바른 방법이 아닙니다. 대신 **_의존 방향_**이라는 측면에서 생각해봅시다.

1. HTML에 의존하는 CSS

- HTML <- CSS
- 스타일을 수정해야 할 때, HTML은 고정되어있고 CSS를 수정해야 합니다.
- HTML에서는 CSS의 참조 지점(또는 훅)을 설정합니다.
- `profile` 처럼 컨텐츠에 기반해 클래스 이름을 짓는 방법입니다.
- 관심사는 분리(restylable)되었으나, 재사용성(reusable)이 떨어집니다.

2. CSS에 의존하는 HTML

- HTML -> CSS
- 스타일을 수정해야 할 때, CSS는 고정되어있고 HTML을 수정해야 합니다.
- 미리 정의된 CSS 클래스를 결합하여 사용합니다.
- `media-card` 처럼 컨텐츠에 구애받지 않는 클래스 이름을 짓는 방법입니다.
- 재용성(reusable)을 확보했으나, 관심사를 분리(restyleable)하지 못합니다.

둘 다 본질적으로 잘못된 것은 아닙니다. 단지 상황에 따라 어떤 측면이 더 중요한지에 따라 결정해야 하는 것입니다. 즉, 관심사가 분리(restyleable)된 HTML과 재사용(reusable) 가능한 CSS 중 어떤 것이 더 가치있는가의 문제입니다.

Nicolas Gallagher의 [About HTML semantics and front-end Architecture](https://nicolasgallagher.com/about-html-semantics-front-end-architecture/)를 읽고 재사용 가능한 CSS가 내 프로젝트에 적합하다는 것을 확신하게 되었습니다.

### 3. 컨텐츠에 구애받지 않는 클래스

이 시점에서 목표는 컨텐츠를 기반으로 클래스를 만드는 것(restylable)을 피하고, 대신 **가능한 한 재사용 가능한 방식으로 클래스를 만드는 것(reusable)**이었습니다.

위에서 살펴본 `profile`, `article`의 예시에서 볼 수 있듯이 <U>클래스의 기능이 많을수록, 클래스가 구체적일수록 재사용하기가 더 어렵습니다.</U> 클래스의 기능이 많다는 뜻은 여러 곳에서 사용될 수 있음을 뜻합니다. 이 때 해당 클래스가 구체적인 이름을 가지고 있다면 재사용하기가 어렵습니다.

예를 들어, `profile` 클래스에 두 가지 버튼, _좋아요_ 버튼과 _닫기_ 버튼을 추가해보겠습니다. 두 버튼이 같은 스타일을 사용한다면, 재사용성을 위해 클래스 이름도 같아야 합니다. 하지만 클래스 이름이 `profile-like-button` 처럼 구체적이기 때문에(다시 말해, 컨텐츠를 기반으로 클래스를 만들었기 때문에) _닫기_ 버튼에 클래스를 재사용할 수 없는 상황이 발생합니다.

```html
<div class="profile">
  <img
    class="profile_image"
    src="https://cdn.example.com/chanhyle"
    alt="profile"
  />
  <div>
    <h2>chanhyle</h2>
    <p class="profile_body">hello world!</p>
  </div>
  <!-- profile_like 컨테이너 추가 -->
  <div class="profile_like">
    <button class="profile_like-button">Like</button>
  </div>
  <!-- 클래스 이름이 구체적이기 때문에, 재사용할 수 없음 -->
  <button class="profile_like-button">Close</button>
</div>
```

따라서 위에서 예시로 든 `profile`, `article` 클래스를 `media-card` 클래스로 추상화한 것처럼, 버튼을 재사용하기 위해 추상화(e.g. `btn`)가 필요합니다. 이러한 방식을 사용한다면 클래스를 더 많이 재사용할 수 있으며 새로운 CSS 클래스를 작성할 필요가 없게 됩니다. 비슷한 방식으로 좋아요 기능을 `article` 클래스에서도 재사용하려면, `profile_like` 클래스를 `like` 클래스로 추상화할 수 있을 것입니다. 결과적으로는 아래와 같을 것이고 재사용성을 극대화할 수 있습니다.

```html
<div class="profile">
  <img
    class="profile_image"
    src="https://cdn.example.com/chanhyle"
    alt="profile"
  />
  <div>
    <h2>chanhyle</h2>
    <p class="profile_body">hello world!</p>
  </div>
  <!-- article 클래스에서 재사용하기 위해 추상화 -->
  <div class="like">
    <!-- 아래 닫기 버튼에서 재사용하기 위해 추상화 -->
    <button class="btn">Like</button>
  </div>
  <button class="btn">Close</button>
</div>
```

```html
<div class="article">
  <img
    class="article_image"
    src="https://cdn.example.com/article1"
    alt="article"
  />
  <div>
    <h2>This is first post.</h2>
    <p class="article_body">hello world!</p>
  </div>
  <div class="like">
    <button class="btn">Like</button>
  </div>
  <button class="btn">Close</button>
</div>
```

### 4. 컨텐츠에 구애받지 않는 클래스 + 유틸리티 클래스

항상 컨텐츠에 구애받지 않는(추상화된) 클래스의 이름을 생각해내려고 노력하는 것은 지치는 일입니다. 또한 추상화된 클래스를 수정해야 할 때, 추상화의 의미가 사라질 수 있습니다.

예를 들어, `like` 클래스를 왼쪽으로 정렬하고 싶은 경우 새로운 `like-left` 클래스를 만들 수 있습니다. 이런 경우 `profile_like`와 `article_like`를 따로 생성하는(즉, 재사용성을 고려하지 않은) 이전의 복제 문제로 되돌아가는 것과 다름 없습니다. 그렇다면 이 문제를 복제가 아닌 구성의 방법으로 어떻게 해결할 수 있을까요?

이 문제를 구성의 방법으로 해결하기 위해서는 재사용 가능한 새 클래스 `align-left` 를 추가하는 방법으로 해결할 수 있습니다. 아래는 `like` 클래스를 왼쪽으로 정렬하기 위해 `align-left` 클래스를 추가한 예시입니다.

```css
.profile {
  background-color: white;
}

.profile img {
  display: block;
  width: 100%;
  height: auto;
}

.profile div p {
  font-size: 1rem;
}

.align-left {
  text-align: left;
}
```

```html
<div class="profile">
  <img
    class="profile_image"
    src="https://cdn.example.com/chanhyle"
    alt="profile"
  />
  <div>
    <h2>chanhyle</h2>
    <p class="profile_body">hello world!</p>
  </div>
  <div class="like align-left">
    <button class="btn">Like</button>
  </div>
  <button class="btn">Close</button>
</div>
```

left, right와 같은 스타일 정보가 HTML에 나타나는 것은 확실히 관심사가 분리되지 않은 것입니다. 그렇기 때문에 이에 대해 불편해하는 사람들도 있을 것입니다. 하지만 우리는 재사용성을 위해 (2) CSS에 의존하는 HTML을 작성하고 있습니다. 스타일을 수정하기 위해서 CSS가 아닌 HTML을 수정하고 있습니다. 이러한 맥락에서 `like` 와 `align-left` 클래스는 의미론적인지 아닌지가 중요한 것이 아닙니다. 두 클래스 모두 HTML 표시에 어떤 영향을 미치는지에 따라 이름이 붙여진 것이며, 특정 표시 결과를 얻기 위해 HTML에서 사용되고 있다는 점에 주목해야 합니다.

`align-left` 처럼 재사용성이 높고 빠르게 사용할 수 있는 클래스를 유틸리티(Utility) 클래스라고 부릅니다.

### 5. 유틸리티-우선 CSS

유틸리티 클래스를 떠올린 후, 필요한 여러 가지 종류의 유틸리티 클래스를 작성하였습니다.

- text sizes, colors, weights
- border colors, widths and positions
- background colors
- flexbox utilities
- padding, margin

놀라운 점은 아래와 같이 새로운 CSS를 작성하지 않고 유틸리티 클래스만으로도, 완전한 구성 요소를 만들 수 있다는 것이었습니다!

```html
<div class="card rounded shadow">
  <img class="block fit" src="https://cdn.example.com/chanhyle" alt="profile" />
  <div>
    <h2>chanhyle</h2>
    <p class="py-3 px-4 border-b">hello world!</p>
  </div>
  <div class="py-2 px-4 border-r border-dark-soft align-left">
    <button class="py-2 px-4">Like</button>
  </div>
  <button class="py-2 px-4">Close</button>
</div>
```

처음에는 사용된 클래스의 수 때문에 당황스러울 수도 있지만, 만약 이를 유틸리티 클래스 대신 컨텐츠에 기반한 클래스 이름을 사용했다면 `.image-card-with-a-full-width-section-and-a-split-section` 처럼 되었을 수도 있습니다. 대신 우리는 위처럼 작은 컴포넌트들로 나누어 사용할 수 있습니다. `card--rounded` 클래스를 새로 만드는 것보다는 `rounded` 유틸리티 클래스를 재사용하는 것이, `card--shadow` 클래스를 새로 만드는 것보다는 `shadow` 유틸리티 클래스를 재사용하는 것이 조금 더 자연스러울 것입니다.

유틸리티 클래스의 가장 큰 장점 중 하나는 팀의 모든 개발자가 항상 고정된 옵션 세트에서 값을 선택한다는 것입니다. 변수나 믹스인을 통해 일관성을 강화하는 방법도 있지만, 새로운 CSS를 만들고 수정하는 것은 여전히 새로운 복잡성을 만들어냅니다. 대신, 프로젝트의 모든 사람이 선별된 제한된 옵션 세트에서 스타일을 선택하면 CSS가 프로젝트 크기에 따라 선형적으로 증가하는 것을 막으면서 일관성을 유지할 수 있습니다.

유틸리티 클래스만으로도 복잡한 구성 요소를 만드는 것이 가능하지만, 반드시 그래야 하는 것은 아닙니다. 프로젝트에 따라서 `py-2 px-4 border-r border-dark-soft` 로 사용하지 않고 `like` 로 간단하게 (1) HTML에 의존하는 CSS를 작성하는 것이 더 간단할 때가 있습니다.

그럼에도 내가 유틸리티-우선 CSS라고 부르는 이유는 유틸리티 클래스로 모든 것을 작성하고, 반복되는 패턴이 나타날 때만 추출하기 때문입니다. 예를 들어, `profile`과 `article`에 있는 좋아요 컨테이너가 같은 `py-2 px-4 border-r border-dark-soft` 유틸리티 클래스로 구성되어 있다면, `like` 라는 새로운 클래스로 추출하는 방법을 사용할 수 있습니다. 하지만 만약 `like` 클래스를 미리 클래스로 추출해 놓았지만(조기 추상화) 결국 재사용되지 않는다면 낭비일 뿐만 아니라 코드를 복잡하게 만들 수 있습니다. 유틸리티 클래스로 먼저 빌드하고 중복이 발견될 때 클래스로 추출하는 방법을 사용한다면, 조기 추상화는 발생하지 않을 것입니다.

마지막으로 유틸리티 클래스를 사용하는 것이 HTML 요소에 인라인으로 스타일 태그를 추가하는 것과 같다고 생각할 수 있지만, 경험상으로는 매우 다릅니다. 인라인 스타일은 선택하는 값에 제한이 없습니다. 이는 (1) HTML에 의존하는 CSS를 작성할 때 직면하는 문제와 같습니다. 재사용성이 떨어지고 중복된 코드가 많아집니다. 하지만 유틸리티 클래스는 제한된 값 중 하나를 선택하도록 강요하기 때문에 재사용성을 확보할 수 있습니다. 또한 디자인이 더 일관되게 보이는 효과도 얻을 수 있습니다.

## 요약

## Notes
