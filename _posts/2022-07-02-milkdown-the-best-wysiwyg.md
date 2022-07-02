---
title: "[React, JS] 완전 이쁜 WYSIWYG의 희망편, 밀크다운"
date: 2022-07-02 22:01 +0900
categories: [Develop, React]
tags: [React, Javascript, WYSIWYG, 마크다운]
---

# 🛫 시작하기에 앞서

![Milkdown Logo](https://github.com/Saul-Mirone/milkdown/raw/main/gh-pages/public/milkdown-homepage.svg)

## 🙃 밀크다운이라고 들어봤어?

밀크다운(Milkdown)은 [Typora](https://typora.io/)에 영감을 받아서 제작된 **WYSIWYG**(What You See Is What You Get) 마크다운(Markdown) 에디터입니다. [Prosemirror](https://prosemirror.net/)와 [Remark](https://github.com/remarkjs/remark)를 기반으로 제작되었습니다.

기본 디자인이 Material 디자인과 Nord 테마로 적용돼 있어서 정말정말정말로 예쁩니다. 😁

![Milkdown Demo](/uploads/milkdown-demo.jpeg)

[온라인 데모](https://milkdown.dev/online-demo)에서 직접 확인할 수 있으니 한 번 체험해 보셔도 좋을 것 같네요!

저는 리액트에서 사용했기에 리액트를 기준으로 설명해 드릴 예정입니다. [밀크다운 도큐먼트](https://milkdown.dev/getting-started)에 **React**를 포함한 **Vue, Svelte, SolidJS, Next.js, angular, Vue2, VanillaJS**에서 사용하는 방법이 나와 있으니 다른 방식으로 개발하시는 분들은 위 **도큐먼트**를 확인해주시면 될 것 같습니다!

# 🚀 밀크다운 시작하기

## ➕ 밀크다운 설치하기

리액트에서 밀크다운을 사용하기 위해서는 기본적으로 `@milkdown/react`, `@milkdown/core`, `@milkdown/prose`, `@milkdown/preset-commonmark`, `@milkdown/theme-nord`를 설치하셔야 합니다.

```bash
npm install @milkdown/react @milkdown/core @milkdown/prose
npm install @milkdown/preset-commonmark @milkdown/theme-nord
```

그 이후에 기본적인 밀크다운 컴포넌트를 생성하는 것은 매우 간단합니다!

```typescript
import React from 'react';
import { Editor, rootCtx } from '@milkdown/core';
import { nord } from '@milkdown/theme-nord';
import { ReactEditor, useEditor } from '@milkdown/react';
import { commonmark } from '@milkdown/preset-commonmark';

export const MilkdownEditor: React.FC = () => {
    const editor = useEditor((root) =>
        Editor.make()
            .config((ctx) => {
                ctx.set(rootCtx, root);
            })
            .use(nord)
            .use(commonmark),
    );

    return <ReactEditor editor={editor} />;
};
```

직접적인 컴포넌트는 `ReactEditor`를 통해서 생성을 하며 **editor** props을 통해서 밀크다운 설정값을 넣어주면 됩니다.

## 🐛 밀크다운에 초기 값 넣기

밀크다운 에디터에서 처음에 초기 값(텍스트)을 넣어서 서식을 제공하거나, 읽기 모드로 변경하여 사용자에게 보여주고 싶을 경우에는 아래와 같은 방식으로 초기 값을 넣어줄 수 있습니다.

```typescript
import { defaultValueCtx } from '@milkdown/core';

// More codes here..

// 초기 텍스트
const defaultValue = "# Hello My Default Value!"

const editor = useEditor((root) =>
        Editor.make()
            .config((ctx) => {
                ctx.set(rootCtx, root);
            })
            // 아래 코드를 추가하여 초기 텍스트 삽입
            .config((ctx) => {
                ctx.set(defaultValueCtx, defaultValue);
            }
// More codes here..
```

## 🎧 밀크다운을 읽기 모드(Read-only)로 변경

**밀크다운에 초기 값 넣기**에서 언급했듯이, 밀크다운을 단순히 편집기로 사용하는 것 이외에도 JSON이나 HTML, Markdown을 가져와서 텍스트를 읽기 전용으로만 읽게 할 수 있습니다. 읽기 모드는 아래 코드로 적용할 수 있습니다.

```typescript
import { editorViewOptionsCtx } from '@milkdown/core';

// More codes here..

const editor = useEditor((root) =>
    Editor.make()
      .config((ctx) => {
        ctx.set(rootCtx, root);
      })
      // 이 부분을 삽입하여 읽기 모드로 변경 가능
      .config((ctx) => {
        ctx.set(editorViewOptionsCtx, { editable: () => false });
      })

// More codes here..
```

## 🗻 Nord 테마 라이트/다크 모드 설정

단순히 `nord` 테마를 쓰면 다크 모드로 설정이 되는데, 수동으로 라이트나 다크 모드로 변경하고 싶으면 코드를 다음과 같이 바꾸면 됩니다.

```diff
- import { nord } from '@milkdown/theme-nord';
+ import { nordLight, nordDark } from '@milkdown/theme-nord';
- .use(nord)
+ .use(nordLight)
# 혹은
+ .use(nordDark)
```

## 🫠 플러그인 삽입

### 유의사항

`plugin-tooltip`이나 `plugin-menu`와 같이 **구글 아이콘**을 사용하는 경우에는 미리 `index.html`에 구글 아이콘을 추가해야 합니다.

```html
<!-- index.html -->
<html>
    <head>
        <link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons" />
        <link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons+Outlined" />
    </head>
    <!-- more codes here.. -->
</html>

```

### plugin-tooltip

툴팁 플러그인은 텍스트를 더블클릭하여 툴팁이 뜨게 할 수 있습니다.

![Markdown Tooltip](/uploads/milkdown-tooltip.png)

해당 플러그인은 다음을 통해서 설치할 수 있습니다.

```bash
npm install @milkdown/plugin-tooltip
```

또한 설치된 플러그인은 `editor`에 다음을 추가하여 삽입할 수 있습니다.

```typescript
import { tooltip } from '@milkdown/plugin-tooltip';

// 에디터 코드
.use(tooltip)
```

### plugin-menu

메뉴 플러그인은 상단에 메뉴를 뜨게 할 수 있습니다.

![Milkdown menu](/uploads/milkdown-menu.png)

해당 플러그인은 다음을 통해서 설치할 수 있습니다.

```bash
npm install @milkdown/plugin-menu
```

또한 설치된 플러그인은 `editor`에 다음을 추가하여 삽입할 수 있습니다.

```typescript
import { menu } from '@milkdown/plugin-menu';

// 에디터 코드
.use(menu)
```

### 그 이외의 수많은 플러그인

그 이외에도 밀크다운에서는 수많은 플러그인을 지원하고 있으니 한 번 체크하는 것을 추천해 드립니다.

*   [밀크다운 공식 플러그인](https://milkdown.dev/using-plugins)

*   [밀크다운 커뮤니티 플러그인](https://github.com/Saul-Mirone/awesome-milkdown)

## 🙊 밀크다운을 vscode에서 사용하기

![Milkdown-vscode](/uploads/milkdown-vscode.png)

밀크다운을 vscode에 설치해서 마크다운을 깔끔하게 수정할 수 있습니다. 설치 방법은 다음과 같습니다.

    이름: Milkdown
    ID: mirone.milkdown
    설명: Edit markdown in a WYSIWYG way, powered by milkdown.
    버전: 0.0.15
    게시자: mirone
    VS Marketplace 링크: <https://marketplace.visualstudio.com/items?itemName=mirone.milkdown>

해당 링크에 들어가서 밀크다운을 설치하거나 좌측 플러그인을 선택하여 **마켓플레이스**에 진입한 뒤, milkdown을 검색하여 설치합니다.

이후 맥의 경우 `command + shift + p` , 윈도우의 경우 `ctrl + shift + p`를 눌러 명령 팔레트를 연 뒤에 **Milkdown : Open with milkdown**을 눌러서 밀크다운을 실행합니다.

# 😌 결론

오늘은 밀크다운을 사용하는 방법을 알아보았습니다. 강력한 플러그인들과 **WYSIWYG**의 장점을 한 번에 사용할 수 있어서 너무 마음에 드는 라이브러리였습니다!

## 참조

*   [Milkdown](https://milkdown.dev/)

*   [Saul-Mirone/milkdown: 🍼 Plugin driven WYSIWYG markdown editor framework. (github.com)](https://github.com/Saul-Mirone/milkdown)

