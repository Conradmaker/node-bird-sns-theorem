# 빌드

자 이제 드디어 완성이 된것 같습니다! 

배포를 하기 전에 빌드를 해주어야 겠죠? 

지금까지 개발모드에서는 실시간으로 빌드를 해줬기 때문에 속도가 매우 느렸습니다.. 

배포를 하게되면 달라진 속도를 볼 수 있습니다. 

그전에 Webpack설정을 해주어야 합니다. 

next도 웹팩이 내장되어 있기때문에 next.confog.js파일을 root에 만들어주면 된답니다. 

webpack을 다루는 내용은 아니기 때문에 공식문서나 다른 게시물을 통해 학습해주세요.

```javascript
const withBundleAnalyzer = require("@next/bundle-analyzer")({
  enabled: process.env.ANALYZE === "true",
});

module.exports = withBundleAnalyzer({
  compress: true, //파일 압축 플러그인
  webpack(config, { webpack }) {
    const prod = process.env.NODE_ENV === "production";
    const plugins = [...config.plugins];
    // if (prod) {
    //   plugins.push(new CompressPlugin());
    // }
    return {
      ...config,
      mode: prod ? "production" : "development",
      devtool: prod ? "hidden-source-map" : "eval",
      plugins,
    };
  },
});
```

compress는 파일을 gzip형태로 압축해주는 역할을 하며 파일의 용량이 매우 줄어드게 될 것입니다. 

## @next/bundle-analyzer

위에서 웹팩설정을 할때 번들애널라이저를 추가해 주었는데 이또한 npm을 통해 설치해주어야 합니다. 

빌드를 하게되면 페이지를 띄워주게 됩니다. 

그 페이지는 잠시뒤에 확인해보겠습니다. 

## cross-env

```javascript
process.env.ANALYZE === "true"
process.env.NODE_ENV === "production"
```

process.env에 접근하는 방법이 dotenv를 통해 진행했었는데, 

사실 front에서도 dotenv를 설치해 다음방식으로 접근해줘도 괜찮습니다. 

```javascript
const dotenv = require('dotenv')
dotenv.config();
```

하지만 더 쉬운 방법이 있습니다. 

pakage.json에서 직접 설정해주는 방법인데요. 

```javascript
"scripts": {
    "dev": "next",
    "build": "ANALYZE=true NODE_ENV=production next build"
  },
```

npm run build를 통해 접근이 가능해요. 

하지만, 이 방법은 window OS에서는 사용이 불가능하답니다....

그래서 cross-env라는 라이브러리를 설치해주어야 합니다. 

```javascript
npm i cross-env
```

그뒤 build명령어의 앞에 cross-env를 추가해주면 끝납니다!

```javascript
 "scripts": {
    "dev": "next",
    "build": "cross-env ANALYZE=true NODE_ENV=production next build"
  },
```

npm run build를 통해 빌드를 진행해주면

@next/bundle-analyzer에 의해 다음과 같은 페이지가 뜨게 됩니다. 

![server](../.gitbook/assets/image%20%2816%29.png)

![client](../.gitbook/assets/image%20%2815%29.png)

아래있는 client페이지는 상당히 중요한데, 이 크기가 줄어들수록 좋기 때문입니다. 

보통 react-dom과 같은 큰 덩어리와 \(concatenated\)라고 되어있는 것들은 통합된것이기 때문에 건들일 것이 없을것입니다. 

하지만, moment를 쓰게 됬다면 매우큰 박스가 생성되었을 것입니다. 

그런것들을 정리해주는 작업을 해줘서 용량을 줄여나가면 됩니다. 

보통 페이지당 1m가 초과되면 좋지 않습니다. 

그 이상부터는 모바일에서 끊김이 발생하기 때문입니다.. 

자 이제 build된 결과물을 AWS를 통해 배포하는 과정을 진행해 보아요!

