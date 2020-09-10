# CSS 서버사이드 렌더링

styled-components 를 사용할 때 css 서버사이드 렌더링을 하지 않으면, 

초기 로딩시에 styled-components가 적용되지 않는 것을 볼 수 있습니다. 

그렇다면 가장 먼저 CSS를 로드해줘야 하는 것인데, 어떻게 해야 구현이 가능할까요?



먼저 next의 page순서에 대하여 알아야 합니다. 

우리는 가장먼저 실행할 페이지를 \_app.js라는 페이지로 만들어주었습니다. 

하지만 app.js보다 한단계 먼저 실행되는 파일이 있는데 \_document.js라는 파일입니다. 



아쉽게도 \_document.js는 아직 클래스형 컴포넌트로 구성하고 있습니다. 

## Babel

\_document.js를 만들기 전에 babel을 먼저 셋팅해보겠습니다. 

root 디렉토리에 .babelrc를 만들어주세요

```javascript
{
  "presets": ["next/babel"],
  "plugins": [
    [
      "babel-plugin-styled-components",
      {
        "ssr": true,          //ServerSiceRendering
        "displayName": true
      }
    ]
  ]
}
```

바벨을 다루는 내용이 아니기 때문에 넘어가겠습니다. 

.babelrc를 설정해준뒤 

```text
npm i babel-plugin-styled-components
```

를 통해 바벨 플러그인을 설치해 줍니다. 

아래 내용은 SSR을 진행할때의 기본적인 \_document.js의 틀입니다. 

```javascript
import Document, { Html, Main, NextScript } from "next/document";
import Head from "next/head";

export default class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const initialProps = await Document.getInitialProps(ctx);
    return {
      ...initialProps,
    };
  }
  render() {
    return (
      <Html>
        <Head>
          <body>
            <Main></Main>
            <NextScript />
          </body>
        </Head>
      </Html>
    );
  }
}
```

## Polyfill

먼저 **polyfills**라는 것에 대하여 알아볼 것인데요 . 

Ployfills는 현재의 넥스트나 다른 것들의 버전들이 너무 최신이기 때문에 진행해주어야 하는것입니다. 

왜냐하면 너무 최신의 기능을 사용하게 되면, 일부 브라우저 \(예를들면 IE, ie, Internet Explorer ...\)

에서 작동을 안하기 때문입니다. 다른것들보다 쓸것만 선택해서 넣을 수 있기 때문에 요즘 아주 핫하다고 볼 수 있겠어요.

```javascript
import React from "react";
import Document, { Html, Head, Main, NextScript } from "next/document";
import { ServerStyleSheet } from "styled-components"; //스타일드 컴포넌트 SSR제공

export default class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const initialProps = await Document.getInitialProps(ctx);
    return {
      ...initialProps,
    };
  }
  render() {
    return (
      <Html>
        <Head />
        <body>
          <Main />
          //NextScript 보다 위에 있어야 합니다. 
          <script src="https://polyfill.io/v3/polyfill.min.js?features=es6,es7,es8,es9,NodeList.prototype.forEach&flags=gated" />
          <NextScript />
        </body>
      </Html>
    );
  }
}

```

다음은 Next의 get InitialProps와 styled-components에서 기본으로 제공하는 ServerStyleSheet을 통해 적용해주도록 하겠습니다.

아직까지 \_document.js 에서 getInitialProps말고 다른 방법은 있는지 못찾겠습니다...

```javascript
import React from "react";
import Document, { Html, Head, Main, NextScript } from "next/document";
import { ServerStyleSheet } from "styled-components"; //스타일드 컴포넌트 SSR제공

export default class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const sheet = new ServerStyleSheet();
    const originalRenderPage = ctx.renderPage;

    try {
      ctx.renderPage = () =>
        originalRenderPage({
          enhanceApp: (App) => (props) =>
            sheet.collectStyles(<App {...props} />),
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
        <Head />
        <body>
          <Main />
          <script src="https://polyfill.io/v3/polyfill.min.js?features=es6,es7,es8,es9,NodeList.prototype.forEach&flags=gated" />
          <NextScript />
        </body>
      </Html>
    );
  }
}

```

완성입니다. 

이건 styled-components 와 next를 통해 SSR을 진행한다면 그대로 써도 될 듯 합니다. 

