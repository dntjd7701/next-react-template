## Next Template

- Next
- React
- TypeScript
- ESLint(gts)
- Styled-components
- commitlint

1. npx create-next-app --typescript
2. npm install -D gts
3. npm install -D eslint-plugin-node eslint-plugin-prettier @typescript-eslint/eslint-plugin
4. .prettierrc.js 생성

```js
// .prettierrc.js
module.exports = {
  ...require('gts/.prettierrc.json'),
};
```

5. .eslintrc.json 수정

```js
// .eslintrc.json
{
  "extends": ["./node_modules/gts/", "next/core-web-vitals"],
  "env": {
    "browser": true,
    "jest": true,
  },
  "settings": {
    "react": {
      "version": "detect"
    }
  }
}
```

6. npm install styled-components
7. npm install -D @types/styled-components

_SSR 사용시_

8. npm install -D babel-plugin-styled-components
9. .babelrc

```
// .babelrc
{
  "presets": ["next/babel"],
  "plugins": [
    [
      "babel-plugin-styled-components",
      {
        "fileName": true,
        "displayName": true,
        "pure": true,
        "ssr": true
      }
    ]
  ]
}
```

10. \_document.tsx

```tsx
// pages/_document.tsx
import Document, {
  Html,
  Head,
  Main,
  NextScript,
  DocumentContext,
} from 'next/document';
import {ServerStyleSheet} from 'styled-components';

class MyDocument extends Document {
  static async getInitialProps(ctx: DocumentContext) {
    const sheet = new ServerStyleSheet();
    const originalRenderPage = ctx.renderPage;
    try {
      ctx.renderPage = () =>
        originalRenderPage({
          enhanceApp: App => props => sheet.collectStyles(<App {...props} />),
        });

      const initialProps = await Document.getInitialProps(ctx);
      return {
        ...initialProps,
        styles: (
          <>
            {initialProps.styles}
            {sheet.getStyleElement()}
          </>
        ),
      };
    } finally {
      sheet.seal();
    }
  }

  render() {
    return (
      <Html>
        <Head></Head>
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    );
  }
}

export default MyDocument;
```

### (번외) Global-styles Normalize, Theme 설정

1. npm install styled-normalize
2. . > styles > global-styles.ts 생성

```ts
// styles/global-styles.ts
import {createGlobalStyle} from 'styled-components';
import {normalize} from 'styled-normalize';

export const GlobalStyle = createGlobalStyle`
    ${normalize}
`;
```

3. pages > \_app.tsx

```tsx
// pages/_app.tsx
import type {AppProps} from 'next/app';
import {ThemeProvider} from 'styled-components';
import {GlobalStyle} from '../styles/global-styles';
import {lightTheme} from '../styles/theme';

function MyApp({Component, pageProps}: AppProps) {
  return (
    <ThemeProvider theme={lightTheme}>
      <GlobalStyle />
      <Component {...pageProps} />
    </ThemeProvider>
  );
}
```

### commitlint 설정하기

1. npm install -D @commitlint/cli @commitlint/config-conventional
2. ./ > commitlint.config.js 생성

```js
module.exports = {extends: ['@commitlint/config-conventional']};
```

### husky

1. npx husky-init
2. npm i
3. npx husky add .husky/commit-msg //윈도우는 / 대신 \ 사용
4. .husky 수정

```
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx --no-install commitlint --edit $1
```

---

- .husky : husky 라이브러리 관련 설정
- components : 컴포넌트 파일, 유닛 테스트 파일
- pages : 페이지 파일, 엔드포인트 테스트 파일
- styles : 글로벌 스타일, 테마 파일 (styled-components)

**파일 설정 내용**

- pages/\_app.tsx : styled-components의 global-style이나 theme 설정
- pages/\_document.tsx : styled-components의 SSR 사용 위한 설정
- .babelrc : styled-components의 SSR 사용 위한 설정
- .eslintrc.json, .prettierrc.js : GTS:Google TypeScript 코드 스타일 사용 위한 설정
- commitlint.config.js : commitlint 라이브러리 사용 위한 컨벤션 설정
