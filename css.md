# CSS 서버사이드 렌더링

.babelrc 설정해준뒤 

npm i babel-plugin-styled-components



```text
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

이게 document 기본형

