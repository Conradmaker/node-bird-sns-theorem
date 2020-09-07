---
description: Ant-design / Styled-components
---

# Style

### 설치

먼저 패키지를 설치해 줍니다.

@ant-design/icons 를 따로 설치해줘야 하는 이유는 보통 icon과 같은 이미지 패키지의 크기가 크기 때문입니다.

```text
yarn add antd styled-component @ant-design/icons
```

```text
  "dependencies": {
    "@ant-design/icons": "^4.2.1",
    "antd": "^4.5.1",
    "next": "^9.5.0",
    "react": "^16.13.1",
    "react-dom": "^16.13.1",
    "styled-components": "^5.1.1"
  },
```

Ant-design이나 Bootstrap은 보통 실무에서는 관리자페이지와 같이 디자인의 중요성이 조금은 떨어지는 페이지에 사용됩니다.

{% embed url="https://ant.design/components/overview/" %}

위 페이지에서 컴포넌트 목록을 확인할수 있습니다.

### 적용시키기

Menu component를 가져오게 된다면

{% code title="AppLayout.js" %}
```jsx
import React from "react";
import Link from "next/Link"; //import해주기
import { Menu } from "antd";

export default function AppLayout({ children }) {
  return (
  <>
    <Menu mode="horizontal">
      <Menu.Item>
        <Link href="/">
          <a>Conrad</a>
        </Link>
      </Menu.Item>
      <Menu.Item>
        <Link href="/profile">
          {/* Link태그 안에서 href 해준뒤 내부에 a태그 */}
          <a>프로필</a>
        </Link>
      </Menu.Item>
      <Menu.Item>
        <Link href="/signup">
          <a>회원가입</a>
        </Link>
      </Menu.Item>
      
    </Menu>
{children}
</>
  );
}
```
{% endcode %}

이렇게 작업하면 되지만

![CSS&#xAC00; &#xAE68;&#xC9D1;&#xB2C8;&#xB2E4;.](../.gitbook/assets/image%20%2810%29.png)

next는 안에 webPack이 내장되어있는데, css를 보는순간 js에 넣어주게됩니다..

pages의 공통으로 적용할 것이이 있을때는,  모든 page에서 일일이 써주는 것이 아니라

pages폴더 안에 \_app.js를 만들어 공통적인 요소를 하나로 처리해주면 됩니다.

```jsx
import React from "react";
import "antd/dist/antd.css";

export default function App({ Component }) {
  return <Component />;
}
```

이렇게 되면 \_app.js는 다른 페이지들의 부모요소가 됩니다. \(HOC가 되는것이죠\)

Component는 다른 요소들 즉 출력할 페이지를 의미하게 됩니다.

이뿐아니라 모든페이지에 공통되는 것들을 적어주면 된답니다.

{% hint style="info" %}
\_app.js  : 모든 페이지의 공통되는 것들

Layout : 일부 페이지에 공통인 것들
{% endhint %}

### head영역 설정하기

next는 HTML의 head태그를 지원합니다.

또한 아래와 같이 HTML의 title이나 meta도 지정해 줄 수 있습니다.

```jsx
import React from "react";
import Head from "next/head";
import "antd/dist/antd.css";

export default function _app({ Component }) {
  return (
    <div>
      <Head>
        <meta charSet="utf-8"></meta>
        <title>Wongeun</title>
      </Head>
      <Component />
    </div>
  );
}
```

만약 공통되지 않는 Head라면 index.js에 넣어주시면 됩니다.



### CSS수정하기

아래와 같이 검색창을 넣어주세여

```jsx
<>
      <Menu mode="horizontal">
        <Menu.Item>
          <Link href="/">
            <a>Home</a>
          </Link>
        </Menu.Item>
        <Menu.Item>
          <Link href="/profile">
            <a>프로필</a>
          </Link>
        </Menu.Item>
        <Menu.Item>
          <Link href="/signup">
            <a>회원가입</a>
          </Link>
        </Menu.Item>
        // 검색추가
        <Menu.Item>
          <Input.Search enterButton></Input.Search>
        </Menu.Item>
      </Menu>
      {children}
    </>
```

![&#xC704;&#xCE58;&#xAC00; &#xBB54;&#xAC00; &#xC774;&#xC0C1;&#xD558;&#xC8E0;?](../.gitbook/assets/image%20%284%29.png)

```jsx
<Input.Search
            enterButton
            style={{ verticalAlign: "middle" }}
          ></Input.Search>
```

이렇게 css를 줄 수도 있습니다.

![](../.gitbook/assets/image%20%285%29.png)

### 반응형

이런 CSS시스템은 Grid 시스템을 가지고 있는데 

모바일페이지를 만드는 방법에는 두가지가 있습니다.

{% hint style="info" %}
1. 반응형 - 화면크기에 따라 컴포넌트들이 재배치 되는 것.
2. 적응형 - 모바일페이지 따로, 데스크탑 페이지 따로 만드는 것.
{% endhint %}

Ant-Design에서는 반응형 Grid시스템을 제공합니다.

row / column 두가지 컴포넌트를 지원하기 떄문에 쉽게 적용시킬 수 있습니다.

작은화면 -&gt; 큰화면 순으로 디자인하는 것이 편하며,

총 24칸으로 나누어져 있기 때문에 24로 나누어져야만 하며 24가 넘어가면 안됩니다.

```jsx
<Row gutter={8}>  //column들 간격 8px (4px+4px)
        <Col xs={24} md={6}>
          왼쪽메뉴
        </Col>
        <Col xs={24} md={12}>
          {children}
        </Col>
        <Col xs={24} md={6}>
          오른쪽메뉴
        </Col>
 </Row>
 //모바일에서는 24 24 24
 //md
```

보통 태블릿 : sm  모바일: xs  이렇게 사용합니다.

| xs | `screen < 576px` | number \| object | - |
| :--- | :--- | :--- | :--- |
| sm | `screen ≥ 576px` | number \| object | - |
| md | `screen ≥ 768px` | number \| object | - |
| lg | `screen ≥ 992px` | number \| object | - |
| xl | `screen ≥ 1200px` | number \| object | - |
| xxl | `screen ≥ 1600px` | number \| object | - |

물론 나중에 커스텀 할 수도 있습니다.

gutter는 grid-gap 과 같은 속성입니다.

아이템 사이의 간격을 의미합니다.

```jsx
</Col>
  <Col xs={24} md={6}>
     <a
      href="https://ant.design/components/grid/"
      target="_blank"
      rel="noreferrer noopener"
      >
   made by Conrad
   </a>
</Col>
```

{% hint style="danger" %}
a태그에서 `target="_blank"`를 이용해 새창으로 링크를 띄운다면 

```jsx
rel="noreferrer noopener"
```

속성을 추가해 보안취약점을 보완해야만 합니다.
{% endhint %}

### 로그인창\(왼쪽메뉴\)렌더링

왼쪽메뉴에는 로그인기능을 만들어 줍니다.

백앤드 서버가 없기때문에 `useState`를 이용한 상태관리로 더미데이터를 만들어 줍니다.

```jsx
 const [isloggedIn, setIsLoggedIn] = useStatee(false);
```

우선 UserProfile.js / LoginForm.js 두가지 컴포넌트를 만들어주세요

Presenter과 Container를 나눠서 관리하는 것을 예전부터 많이 진행했지만, 

리액트팀이 Hooks 이후로 나누는걸 추천하지 않는다 하였기에 한 파일에서 진행해봅니다.

{% code title="LoginForm.js" %}
```jsx
 <Form>
      <div>
        <label htmlFor="user-id">아이디</label>
        <br />
        <Input name="user-id" required />
      </div>
      <div>
        <label htmlFor="user-password">비밀번호</label>
        <br />
        <Input
          name="user-password"
          type="password"
          required
        />
      </div>
      <div>
        <Button type="primary" htmlType="submit" >
          로그인
        </Button>

        <Link href="/signup"> //회원가입페이지로 넘어가기
          <a>
            <Button>회원가입</Button>
          </a>
        </Link>
      </div>
    </Form>
```
{% endcode %}

#### useState를 이용한 Input 상태관리

```jsx
  const [id, setId] = useState("");
  const [password, setPassword] = useState("");
  const onChangeId = useCallback((e) => {
    setId(e.target.value);
  }, []);
  const onChangePassword = useCallback((e) => {
    setPassword(e.target.value);
  }, []);
```

이제 상태들을 JSX에 적용시켜주세요

```jsx
import React, { useState, useCallback } from "react";
import { Form, Input, Button } from "antd";
import Link from "next/link";

export default function LoginForm() {
  const [id, setId] = useState("");
  const [password, setPassword] = useState("");
  const onChangeId = useCallback((e) => {
    setId(e.target.value);
  }, []);
  const onChangePassword = useCallback((e) => {
    setPassword(e.target.value);
  }, []);

  return (
    <Form>
      <div>
        <label htmlFor="user-id">아이디</label>
        <br />
        <Input name="user-id" value={id} onChange={onChangeId} required />
      </div>
      <div>
        <label htmlFor="user-password">비밀번호</label>
        <br />
        <Input
          name="user-password"
          type="password"
          value={password}
          onChange={onChangePassword}
          required
        />
      </div>
      <div style={{marginTop:10}}>
        <Button type="primary" htmlType="submit" loading={false}>
          로그인
        </Button>

        <Link href="/signup">
          <a>
            <Button>회원가입</Button>
          </a>
        </Link>
      </div>
    </Form>
  );
}

```

AppLayout 컴포넌트에서 컴포넌트를 렌더링 해주세요.

![&#xC774;&#xB807;&#xAC8C; &#xD654;&#xBA74;&#xC774; &#xC798; &#xB098;&#xD0C0;&#xB0AC;&#xC2B5;&#xB2C8;&#xB2E4;.](../.gitbook/assets/image%20%286%29.png)

한가지 주의해야 할 점이 있습니다. 위에서 작성한 코드중 아래 코드를 봐주세요

```jsx
 <div style={{marginTop:10}}>
```

보통 css에 스타일을 주기 위한 목적으로 간단히 자주 사용되는 방법인데, 이는 큰 담점을 가지고 있습니다.

{% hint style="danger" %}
스타일을 위처럼 객체로 주게되면 객체이기 때문에 바뀐값이 아무것도 없어도 리렌더링 되어버립니다.
{% endhint %}

위 div 태그를 styled-components를 이용해 바꿔주겠습니다.

```jsx
const ButtonWrapper = styled.div`
  margin-top: 10px;
`;

<ButtonWrapper>
```

사실 성능에 큰 지장을 주지는 않기때문에 간단한 작업이면 인라인스타일을 사용하셔도 크게 문제될 것은 없습니다.

### 리랜더링

사실 리랜더링이 되면 처음부터 끝까지 다시 시작되는 것은 맞습니다.

useCallBack, useMemo를 사용해 캐싱하게되면 그걸 막아줄 수 있죠.

React에서는 JSX에서 바뀐부분만 다시 렌더링해줍니다.

즉 이전의 컴포넌트와 비교해서 바뀐 부분만 다시 그려줍니다.

Virtual DOM 즉, 함수형 컴포넌트 안에 return되는 부분을 리액트에서는 처음에 한번 싹 그려준뒤, 

이전 컴포넌트와 다음 컴포넌트를 비교해 바뀐게 있으면 React는 컴포넌트를 리랜더링 해줍니다. 

결국 리랜더링을 적게하면 성능적으로 좋겠죠?

