---
title: "[Github 블로그] Minimal Mistakes 코드 블록을 복사하는 기능 추가하기"
date: 2024-10-19
published: true
toc: true
toc_sticky: true
last_modified_at: 2024-10-19
categories:
  - Github-블로그
tags:
  - minimal mistakes
  - github.io
  - code block
  - copy to clipboard
  - markdown
  - md
---

## 1. 도입하게 된 계기

이번 포스트에서는 누가 처음 개발했는지는 모르겠지만 아주 유용한 기능, copy to clipboard 기능을 Minimal Mistakes에서 구현하는 방법에 대해 알아보려고 해요.

Markdown의 코드 블록을 *드래그-복사*하는 과정을, *단 한 번의 클릭*으로 가능하게 해주는 기능이에요.

언젠가부터 이 기능이 없으면 큰 불편함과 허전함을 느끼게 되었어요.

Minimal Mistakes에는 코드 블록을 복사하는 플러그인이나 기능이 없었고, 구현도 매우 간단해서 직접 추가하기로 하였어요.

구현 과정은 Reference에 있는 두 포스트를 참고하였어요.

## 2. 구현 과정

_Minimal Mistakes의 rouge highlighter 기능을 사용한다는 전제로 구현하였어요._

요약하면 다음과 같아요.

- (1) 복사 버튼 HTML 파일 만들기
- (2) CSS를 추가하여 위치와 스타일 적용하기
- (3) 클립보드에 코드를 복사하는 Javascript 코드를 추가하기
- (4) Jekyll Liquid 태그를 이용하여 코드 블록에 복사 버튼 추가하기

### (1) 복사 버튼 HTML 파일 만들기

`📂 /_includes/code-header.html`

{% include code-header.html %}

```html
<div class="code-header">
  <button class="copy-code-button" title="Copy code to clipboard">
    <img class="copy-code-image" src="/assets/images/copy.png" />
  </button>
</div>
```

프로젝트의 `_includes` 디렉토리 안에 `code-header.html` 이라는 이름으로 새로운 HTML 파일을 생성해 주세요.

`<button>` 내부는 간단하게 이미지만 사용하였어요.

### (2) CSS를 추가하여 위치와 스타일 적용하기

`📂 /_sass/minimal-mistakes/_page.scss`

{% include code-header.html %}

```css
.code-header {
  position: relative;
  top: 50px;
  right: 10px;
  z-index: 1;
}

.copy-code-button {
  float: right;
  cursor: pointer;
  background-color: $code-background-color;
  padding: 5px 5px;
  border-radius: 5px;
  width: 30px;
  height: 35px;
  margin-top: -40px;
  border: none;
}

.copy-code-button:focus {
  outline: none;
}

.copy-code-button:hover {
  background-color: #0f1214;
}

.copy-code-image {
  filter: invert(1);
}
```

`/_sass/minimal-mistakes/`에 있는 `_page.scss` 파일에 위 코드를 추가해 주세요.

위치와 스타일을 조정하고 hover 기능도 추가해 줬어요.

### (3) 클립보드에 코드를 복사하는 Javascript 코드를 추가하기

`📂 /assets/scripts/copyCode.js`

{% include code-header.html %}

```js
const codeBlocks = document.querySelectorAll(
  ".code-header + .highlighter-rouge"
);
const copyCodeButtons = document.querySelectorAll(".copy-code-button");

copyCodeButtons.forEach((copyCodeButton, index) => {
  const code = codeBlocks[index].innerText;
  let id;

  copyCodeButton.addEventListener("click", () => {
    window.navigator.clipboard.writeText(code);

    const img = copyCodeButton.querySelector("img");
    img.src = "/assets/images/check.png";

    if (id) {
      clearTimeout(id);
    }

    id = setTimeout(() => {
      img.src = "/assets/images/copy.png";
    }, 2000);
  });
});
```

`/assets/scripts`에 `copyCode.js` 이름으로 파일을 생성해 주세요.

버튼을 클릭하면 copy 이미지를 check 이미지로 바꾸는 이벤트 리스너를 추가하고, 연속 클릭에도 상태를 유지하기 위해 `clearTimeout` 처리도 해줘요.

그리고 생성한 `copyCode.js` 코드가 실행될 수 있도록 `/_includes/footer.html`의 최하단에 아래 코드를 추가해 주세요.

`📂 /_includes/footer.html`

{% include code-header.html %}

```html
<!-- 파일 최하단에 추가해요. -->
<script src="/assets/scripts/copyCode.js"></script>
```

### (4) Jekyll Liquid 태그를 이용하여 코드 블록에 복사 버튼 추가하기

{% include code-header.html %}

````markdown
{% raw %}{% include code-header.html %}{% endraw %}

```jsx
export default function App() {
  return <div>hello</div>;
}
```
````

마지막으로 포스트를 작성하면서 코드 블록 위에 `{% raw %}{% include code-header.html %}{% endraw %}` 를 추가하면, 복사 버튼이 적용돼요.

매번 Liquid 태그를 붙이는 것이 힘들다면, 코드 에디터의 Code Snippet 기능을 이용하면 조금 더 편하게 사용할 수 있어요.

아래는 VSCode의 Code Snippet 기능을 적용한 화면이에요.

<br/>
<div style="display: flex; justify-content: center;" >
  <img src="/assets/images/snippet.gif" alt="snippet" >
</div>
<br/>

## 3. 결과 화면

<br/>
<div style="display: flex; justify-content: center;" >
  <img src="/assets/images/copy.gif" alt="copy" >
</div>
<br/>

## 4. Reference

- [https://www.aleksandrhovhannisyan.com/blog/jekyll-copy-to-clipboard/](https://www.aleksandrhovhannisyan.com/blog/jekyll-copy-to-clipboard/)
- [https://kosate.github.io/blog/blogs/how-to-add-copy-button-into-jekyll-blogs/](https://kosate.github.io/blog/blogs/how-to-add-copy-button-into-jekyll-blogs/)
