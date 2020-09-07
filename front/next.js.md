# next.js

## Hello Next.js !

### 프로젝트 생성

```text
yarn init

yarn add next react react-dom

yarn dev //먼저 아래 문구를 추가해줘야 합니다.
```

### pakage.json에 다음 추가

```javascript
"scripts": {
    "dev": "next"
}
```

### pages

pages폴더안에 index.js생성

반드시 next.js는 pages폴더 안에 있어야만 합니다.\(코드스플릿팅\)

```jsx
import React from "react"; //있어도 없어도 무관

export default function index() {
  return <div>Hello.next</div>;
}

```



## 페이지와 레이아웃

### page

pages 폴더안에 페이지를 하나씩 추가하면 됩니다.

{% tabs %}
{% tab title="profile.js" %}
```jsx
import React from "react";

export default function profile() {
  return <div>내 프로필</div>;
}

```
{% endtab %}

{% tab title="signup.js" %}
```
import React from "react";

export default function signup() {
  return <div>회원가입</div>;
}

```
{% endtab %}
{% endtabs %}

개발 서버를 다시 켜보면 파일명과 매칭되어서 react-router의 기능을 그대로 사용할 수 있습니다.

* index.js : [http://localhost:3000/](http://localhost:3000/)
* profile.js: [http://localhost:3000/](http://localhost:3000/)profile/
* signup.js: [http://localhost:3000/](http://localhost:3000/)signup/

만약 페이지명을 바꾸고 싶다면 파일명을 \[profile\].js 이렇게 만들면 됩니다.



### Components

리액트에 사용했던 재사용이 가능한 컴포넌트는 따로 폴더를 만들어 만들 수 있습니다.

아래와 같이 말이죠

{% tabs %}
{% tab title="components" %}
```jsx
import React from "react";

export default function AppLayout({ children }) {
  return return (
    <div>
      <h1>공통메뉴</h1>
      {children}
    </div>
  );
}

```
{% endtab %}

{% tab title="index.js" %}
```jsx
import AppLayout from "../components/AppLayout";

export default function index() {
  return <AppLayout>Hello.next</AppLayout>;
}

```
{% endtab %}
{% endtabs %}

![](../.gitbook/assets/image%20%283%29.png)

### Link

물론 Link 기능도 가지고 있습니다.

{% tabs %}
{% tab title="Component" %}
```jsx
import React from "react";
import Link from "next/Link"; //import해주기

export default function AppLayout({ children }) {
  return (
    <div>
      <Link href="/">
        <a>Conrad</a>
      </Link>
      <Link href="/profile">
        {/* Link태그 안에서 href 해준뒤 내부에 a태그 */}
        <a>프로필</a>
      </Link>
      <Link href="/signup">
        <a>회원가입</a>
      </Link>
      {children}
    </div>
  );
}

```
{% endtab %}

{% tab title="page" %}
```jsx
import React from "react";
import AppLayout from "../components/AppLayout";

export default function index() {
  return (
    <AppLayout>
      <h1>Hello.next</h1>
    </AppLayout>
  );
}

```
{% endtab %}
{% endtabs %}

![&#xBAA8;&#xC591;&#xC774; &#xC88B;&#xC9C0;&#xB294; &#xC54A;&#xC9C0;&#xB9CC;...](../.gitbook/assets/image%20%2812%29.png)

간단하게 네비게이션을 만들수 있습니다!

{% hint style="info" %}
처음 빌드하면 Link이동시 딜레이가 있는데, 이는 개발모드이기 때문입니다.

production모드로 변환되면 딜레이가 없어지니 걱정 안해도됩니다.
{% endhint %}

### esLint

이번엔 esLint를 적용시켜보겠습니다.

먼저 설치를 해주세요

```jsx
yarn add eslint -D
yarn add eslint-plugin-react -D //코드점검용
yarn add react-hooks -D 
```

다음에는 .eslintrc 파일을 만들어주세요

```jsx
{
  "parserOption": {
    "ecmaVersion": 2020,
    "sourceType": "module",
    "ecmaFeature": {
      "jsx": true
    }
  },
  "env": {
    "browser": true,
    "node": true,
    "es6": true
  },
  "extends": ["eslint:recommend", "plugin:react/recommend"], //너그럽다..
  "plugins": ["import", "react-hooks"]
}

```

