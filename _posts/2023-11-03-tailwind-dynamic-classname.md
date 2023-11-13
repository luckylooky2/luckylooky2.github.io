---
title: "[React][Tailwindcss] tailwind 동적 클래스 이름 사용하기; typescript, className, CSS"
date: 2023-11-03
published: true
toc: true
toc_sticky: true
last_modified_at: 2023-11-04

categories:
  - React
tags:
  - tailwindcss
  - typescript
  - 42HelloWorld
---

## 1. 문제 발생

음성 채팅 서비스 _42HelloWorld_ 프로젝트에서 React와 함께 사용할 CSS Framework로써 **_tailwindcss_**를 선택하였습니다. [React 프로젝트 개발 환경에서 tailwind를 적용하는 방법](https://tailwindcss.com/docs/installation)에는 여러 가지가 있는데, 대표적으로

- (1) `npm`을 이용하여 프로젝트 내부에 패키지를 설치하는 방법

  ```shell
  # 1. 프로젝트에 devDependencies로 패키지 추가
  $ npm install -D tailwindcss

  # 2. tailwind.config.js 파일 생성
  $ npx tailwindcss init
  ```

  ```js
  // 3. tailwind.config.js 작성
  /** @type {import('tailwindcss').Config} */
  module.exports = {
    content: ["./src/**/*.{js,jsx,ts,tsx}"],
    theme: {
      extend: {},
    },
    plugins: [],
  };

  // 4. tailwind.css를 작성
  @tailwind base;
  @tailwind components;
  @tailwind utilities;

  // 5. App.tsx나 index.tsx에서 tailwind.css를 import
  import "./tailwind.css";
  ```

- (2) `index.html` script 태그로 CDN에서 가져오는 방법

  ```html
  <!DOCTYPE html>
  <html lang="ko">
    <head>
      <!-- CDN script 태그를 추가하는 방법-->
      <script src="https://cdn.tailwindcss.com"></script>
      <title>42Hello World</title>
    </head>
    <body>
      <noscript>You need to enable JavaScript to run this app.</noscript>
      <div id="root"></div>
    </body>
  </html>
  ```

이 있습니다.

첫 번째 방법은 상대적으로 설정이 복잡한 대신, 버전이나 사용자 정의 설정 등 자유도가 높습니다. 반면, 두 번째 방법은 자유도가 낮은 대신에 설치 없이 빠르게 사용할 수 있다는 장점이 있습니다. 개발 환경에서는 간단한 두 번째 방법을 선택하였습니다.

정상적으로 프로젝트에 적용했다면, 아래와 같이 `className`을 이용하여 편리하게 tailwind에서 제공하는 CSS 클래스를 적용할 수 있습니다.

```jsx
<div className="flex justify-between items-center"></div>
```

**1차 MVP 배포를 앞두고, 두 번째 방법(CDN)을 첫 번째 방법(패키지)으로 바꾸는 작업이 필요했습니다.** 두 번째 방법을 그대로 사용할 경우, 배포 시에 CDN 서버에 의존성이 생겨 CDN 서버에 이상이 생기면 CSS를 사용하지 못하는 등의 문제가 발생할 수 있기 때문입니다.

위에서 설명한 과정을 거쳐 첫 번째 방법을 적용하였으나, 지금까지 잘 적용되던 CSS가 _일부_ 작동하지 않는 현상이 발생하였습니다. 아래는 투표 UI 중 현재 투표 상황판의 예시입니다.

- 두 번째 방법(CDN)으로 로드한 경우(정상 작동)

  ![Screenshot 2023-11-03 at 1 28 53 AM](https://github.com/luckylooky2/luckylooky2.github.io/assets/85822311/35cbc3cd-8fba-470c-bfed-deb00075f21a)

- 첫 번째 방법(패키지)으로 로드한 경우

  ![Screenshot 2023-11-03 at 1 28 13 AM](https://github.com/luckylooky2/luckylooky2.github.io/assets/85822311/5ac96a35-4b26-40a0-8983-2df47db2c468)

변경 후, 첫 번째 사진처럼 투표 상황판이 1줄에 참여중인 유저의 수만큼 칸이 나뉘어 있어야 하는데, 두 번째 사진에서는 1) 2줄에 걸쳐서 나타난 것을 확인하였습니다. 또한 아직 투표하지 않은 경우에 칸이 회색으로 표시되어야 하는데, 두 번째 사진에서는 2) 회색이 표현되지 않았습니다.

## 2. 원인

### 1) 원인은 동적 클래스 이름

결론적으로 첫 번째 방법(패키지)에서 `grid-cols-{userCountRef.current}` 처럼 클래스 이름을 동적으로 사용하여 문제가 발생했던 것입니다.

작성한 투표 현황 컴포넌트의 타입스크립트 코드는 다음과 같습니다.

```jsx
// VoteStatusBoard.tsx
const VoteStatusBoard: FC<Props> = ({ userCount }) => {
  // ...
  return (
    // 투표 현황판 블록들을 감싸는 컨테이너
    <div
      className={`grid grid-cols-${userCountRef.current} w-full my-1 mx-auto`}
    >
      {voteStatus.map((v, i) => (
        // 투표 현황판 각각의 블록
        <div
          key={`voteBlock-${v}-${i}`}
          className={`mx-[2px] h-[20px] bg-${setVoteBlockColor(v)}-500`}
        />
      ))}
    </div>
  );
};
```

CSS가 제대로 적용되지 않았던 부분을 살펴보면 공통적으로 동적으로 클래스 이름을 구성한 것을 알 수 있습니다.

- [`grid-cols-{n}` 클래스](https://tailwindcss.com/docs/grid-template-columns)는 grid 레이아웃에서 자식 요소들을 한 블럭 안에 가로 방향으로 배치하기 위해 사용합니다. 현재 음성 통화에 참여하고 있는 유저의 수 `userCount` 만큼 블록을 생성하고, 가로 방향으로 배치합니다.

- [`bg-${color}-500` 클래스](https://tailwindcss.com/docs/background-color)는 해당 요소의 배경 색을 지정하기 위해 사용됩니다. 숫자가 작을수록 흰색에 가까워지고, 커질수록 검은색에 가까워집니다. 각각의 유저가 현재 투표를 진행 중이라면 블록의 배경 색이 gray, 찬성했다면 green, 반대했다면 red로 표시됩니다.

현재 참여중인 유저의 수 `userCount` , 투표 현황 `voteStatus` 상태값에 따라 클래스 이름을 동적으로 교체하여 <U>중복 문자열이 적은 코드를 만드려는</U> 의도였습니다. 하지만 제대로 동작하지 않았습니다.

stack overflow 질문 글과 블로그를 통해 [공식 문서](https://tailwindcss.com/docs/content-configuration#dynamic-class-names)에서 원인을 찾았습니다.

> Dynamic class names : The most important implication of how Tailwind extracts class names is that **it will only find classes that exist as complete unbroken strings in your source files**. If you use string interpolation or concatenate partial class names together, Tailwind will not find them and therefore will not generate the corresponding CSS:

tailwind는 소스 파일에서 문자열 합성이나 리터럴 템플릿을 사용하지 않은 "완전한 문자열 클래스 이름"만 발견할 수 있다고 설명합니다. 그리고 발견한 클래스로만 CSS 클래스를 생성한다는 내용입니다.

```jsx
// 잘못된 예시 : 완전한 문자열 className이 아니므로 grid-cols-n 클래스를 생성하지 않습니다.
<div className={`grid grid-cols-${userCountRef.current}`}></div>

// 완전한 문자열 className이므로 grid-cols-n 클래스를 생성할 수 있습니다.
<div className={`grid ${userCountRef.current === 1 ? "grid-cols-1" : "grid-cols-2" }`}></div>
```

### 2) 첫 번째 방법(패키지)는 동적 클래스 이름이 왜 작동하지 않을까요?

그 이유는 tailwind 패키지는 HTML 파일, 자바스크립트 컴포넌트 파일 등에 존재하는 CSS 클래스 이름을 스캔하고 해당하는 CSS 클래스를 **정적 CSS 파일로 생성하기 때문입니다.**

tailwind는 다음과 같은 과정으로 정적 CSS 파일을 생성합니다.

- HTML 파일, 자바스크립트 컴포넌트 파일 등에 존재하는 CSS 클래스 이름을 찾습니다.
- 전체 CSS 클래스 리스트에서 찾지 못한 CSS 클래스 이름을 **제거(purge) 합니다.**
- 남은 CSS 클래스를 정적 CSS 파일로 빌드합니다.

tailwind는 매우 많은 CSS 클래스가 존재하는데, purge를 통해 <U>전체 클래스 리스트 중에 사용되지 않는 클래스를 제거하는 방법으로 정적 CSS 파일을 생성합니다.</U> 이는 불필요한 클래스를 제거하여 CSS 파일 크기를 최적화하기 위한 과정이라고 할 수 있습니다. 특히, 동적 클래스 이름은 모두 누락시키는 방법을 사용하고 있습니다. 그래서 빌드한 파일에는 동적 클래스 이름과 관련된 CSS 클래스가 존재하지 않게 됩니다.

브라우저 Elements 탭에서 동적 클래스 이름의 CSS 클래스가 생성되지 않음을 확인할 수 있습니다. DOM에는 `grid-cols-2` 클래스 이름은 존재하지만 CSSOM에는 `grid-cols-2` 클래스가 존재하지 않습니다. React가 `grid-cols-2` 클래스로 렌더링하려고 했지만, 빌드된 CSS 파일에는 관련 CSS 클래스가 누락되어 있어 사용할 수 없는 것입니다.

![Screenshot 2023-11-03 at 10 52 38 AM](https://github.com/luckylooky2/luckylooky2.github.io/assets/85822311/daeddf63-6a3b-4794-89c9-7a7c17357117)

![Screenshot 2023-11-03 at 10 53 04 AM](https://github.com/luckylooky2/luckylooky2.github.io/assets/85822311/4c7b548b-731c-4cfa-822e-19eaf9598798)

### 3) 두 번째 방법(CDN)은 동적 클래스 이름이 왜 작동했을까요?

그렇다면 CDN으로 가져오는 방법은 어떻게 정상적으로 동작할 수 있었을까요? 그 이유는 정적 CSS 파일을 생성하는 첫 번째 방법(패키지)와는 다르게, **두 번째 방법(CDN)은 동적으로 CSS 클래스를 생성하기 때문입니다.**

CDN을 통해서 가져온 [아래 파일](https://cdn.tailwindcss.com/3.3.5)은 동적으로 CSS 클래스를 생성해주는 자바스크립트 코드입니다.

```html
<script src="https://cdn.tailwindcss.com"></script>
```

이 방법을 사용하면, DOM에 패키지를 이용한 방법에는 없었던 새로운 style 태그가 새로 생긴 것을 발견할 수 있습니다. 그리고 이 style 태그는 화면이 전환될 때 최신화 됩니다. CDN에서 받아온 자바스크립트 코드가 그때그때 동적으로 필요한 CSS 클래스를 생성해준다고 볼 수 있습니다. 아래 사진에서 위 사진에서는 없었던 `grid-cols-2` 클래스가 CSSOM에도 존재하는 것을 보여줍니다.

![Screenshot 2023-11-04 at 12 33 54 AM](https://github.com/luckylooky2/luckylooky2.github.io/assets/85822311/70968fd6-da0e-4338-bbc4-93ef246f8b52)

![Screenshot 2023-11-04 at 12 38 38 AM](https://github.com/luckylooky2/luckylooky2.github.io/assets/85822311/0206d08e-a5e3-4791-900a-1aa957da3916)

`grid-cols-2` 클래스는 빌드된 파일(`index.css`)에서 가져온 것이 아닌 동적으로 생성된 style 태그에서 가져온 것입니다. 동적으로 생성된 CSS 클래스 덕분에 CDN을 사용한 방법에서는 CSS가 깨지지 않았던 것입니다. 결국 CDN을 이용하는 환경에서는, 런타임에 CSS 클래스가 생성되기 때문에 동적 클래스 이름을 사용해도 정상적인 사용이 가능합니다. 하지만 1) CDN 서버에 의존하는 문제가 발생하고 2) CDN 서버에서 로드하는 자바스크립트 코드가 매우 길기 때문에 배포 환경에서는 CDN을 이용하지 말 것을 권장하고 있습니다.

요약하면, 첫 번째 방법(패키지)에서 동적으로 결정되는 클래스 이름은, 사용되지 않는 클래스에 포함되어 모두 제외됩니다. 결국 정적 CSS 파일에 포함되지 않아 제대로 동작하지 않았습니다. 두 번째 방법(CDN)은 런타임에 자바스크립트 코드를 이용하여 동적으로 CSS 클래스를 생성하기 때문에, 동적으로 결정되는 클래스 이름을 사용해도 제대로 동작한 것이라고 볼 수 있습니다.

## 3. 해결

실제 베포 환경에서는 패키지를 이용한 방법을 이용해야 할 텐데, 동적 클래스 이름을 사용하지 못하는 문제를 어떤 방법으로 해결해야 할까요?

첫 번째, 공식 문서에서 권장하는 대로 클래스 이름을 변형하지 않고, 주어진 문자열 그대로 사용하는 것입니다.

![image (2)](https://github.com/luckylooky2/luckylooky2.github.io/assets/85822311/51b4b6d8-ed95-4bfe-951e-272991bca7ee)

투표 현황 컴포넌트에 완전한 클래스 이름을 사용하여 바꾼 코드는 아래와 같습니다. 중복 문자열이 많아져 코드가 길어진 것은 약간 아쉽게 느껴집니다.

```jsx
// VoteStatusBoard.tsx
const VoteStatusBoard: FC<Props> = ({ userCount }) => {
  // ...
  const GridCols = {
    2: "grid-cols-2",
    3: "grid-cols-3",
    4: "grid-cols-4",
  };

  const VoteBlockColor = {
    ongoing: "bg-gray-500",
    yes: "bg-green-500",
    no: "bg-red-500",
  };

  return (
    <div
      className={`grid ${GridCols[totalNumRef.current]} w-full my-1 mx-auto`}
    >
      {voteStatus.map((v, i) => (
        <div
          key={`voteBlock-${v}-${i}`}
          className={`mx-[2px] h-[20px] ${VoteBlockColor[v]}`}
        />
      ))}
    </div>
  );
};
```

두 번째, `tailwind.config.js` 파일에서 safelist를 설정하는 방법입니다.

```js
// tailwind.config.js
module.exports = {
  content: ["./src/**/*.{js,jsx,ts,tsx}"],
  safelist: [
    {
      pattern: /grid-cols-./, // grid-cols-로 시작하는 모든 클래스 이름은 제거되지 않음
    },
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

**safelist에 설정된 CSS 클래스 이름은 빌드타임 purge 과정에서 제거되지 않습니다.** 패턴을 추가하여 동적 클래스 이름이 들어간 모든 CSS 클래스를 남겨두는 방법으로 해결할 수는 있습니다. 하지만 실제로 사용되지 않는 클래스가 포함될 가능성이 크기 때문에, 대부분은 비효율적인 해결 방법이 될 것입니다. 예를 들어, `grid-cols-` 로 시작하는 모든 클래스를 purge 과정에서 제외한다면, 아래 표에서 볼 수 있듯이 `grid-cols-` 로 시작하는 13개 클래스가 CSS 파일에 모두 포함하게 됩니다. 이 중에서 런타임에 실제로 사용하는 클래스는 `grid-cols-1` 부터 `grid-cols-4` 까지라면, 9개의 필요없는 클래스를 포함시킨 것이 됩니다. 패턴이 더 많아지게 되면 성능에 영향을 미칠 수도 있습니다. 이러한 문제 때문에 tailwind에서는 이 방법보다는 완전한 문자열로 사용하는 방법을 권장하고 있습니다.

![Screenshot 2023-11-04 at 2 56 28 AM](https://github.com/luckylooky2/luckylooky2.github.io/assets/85822311/4e5e428b-a283-46e2-bad8-1226064e61a7)

공식 문서를 자세히 봤었더라면 발생하지 않았을 문제라는 생각이 들었습니다. 새로운 라이브러리를 사용하기 전에 공식 문서를 꼼꼼히 읽을 필요가 있다는 것을 이 문제를 해결하면서 느꼈습니다.

## 4. Reference

- [Responsive design with grid columns not working properly(?)](https://github.com/tailwindlabs/tailwindcss/discussions/8272)
- [tailwindcss-dynamic_class_names](https://tailwindcss.com/docs/content-configuration#dynamic-class-names)
- [Tailwind CSS 에서 동적으로 클래스 할당하기](https://velog.io/@arthur/Tailwind-CSS-%EC%97%90%EC%84%9C-%EB%8F%99%EC%A0%81%EC%9C%BC%EB%A1%9C-%ED%81%B4%EB%9E%98%EC%8A%A4-%ED%95%A0%EB%8B%B9%ED%95%98%EA%B8%B0)
