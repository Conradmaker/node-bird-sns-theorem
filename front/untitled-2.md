# Untitled

더미 데이터로 로그인하기

이번에는 로그인즉, 가짜로그인을 구현해보겠습니다.

먼저 LoginForm에 submit함수를 만들어주세요

```javascript
 const onSubmitForm = useCallback(() => {
    console.log(id, password);
    setIsLoggedIn(true); //Props로 AppLayout에서 받아옵니다.
  }, [id, password]);

```

{% hint style="info" %}
Ant-design에서는 onSubmit 대신 onFinish를 사용해 

e.preventDefault 즉 새로고침을 막아줄 수 있습니다.
{% endhint %}

AppLayout.js도 바꿔줘야 겠죠?

```javascript
 {isloggedIn ? <UserProfile /> : <LoginForm setIsLoggedIn={setIsLoggedIn}/>}
```

만약 로그인이 true면 프로필, false면 로그인창을 띄우는 화면입니다.



UserProfile.js 만들기

Ant-design의 Card를 이용해 유저 프로필을 만들어 보겠습니다.

Card는 정말 쓸모가 많으니 한번 구경다녀오세요

{% embed url="https://ant.design/components/card/" %}

자, 그럼 이제 UserProfile 컴포넌트를 작성해볼까요

UserProfile 컴포넌트 만들기

```javascript
import React from "react";
import { Card, Avatar, Button } from "antd";

export default function UserProfile() {
  return (
    <Card
      actions={[
        //배열이기때문에 키값을 넣어줘야 한다.
        <div key="twit">
          짹짹
          <br />0
        </div>,
        <div key="following">
          팔로잉
          <br />0
        </div>,
        <div key="follower">
          팔로워
          <br />0
        </div>,
      ]}
    >
      <Card.Meta avatar={<Avatar>CD</Avatar>} title="Conrad" />
      <Button>로그아웃</Button>
    </Card>
  );
}

```

Card에 관한 자새한 내용은 다룰것이 많아서 위에 개발자문서를 보면 이해할 수 있습니다.

이제 홈에서 로그인 버튼을 누르면?

![&#xC774;&#xB7F0; &#xACB0;&#xACFC;&#xCC3D;&#xC774; &#xB098;&#xC624;&#xAC8C; &#xB429;&#xB2C8;&#xB2E4;](../.gitbook/assets/image%20%2811%29.png)

이제 가짜로그아웃 기능도 구현해볼게요

```javascript
import React, { useCallback } from "react";
import { Card, Avatar, Button } from "antd";

export default function UserProfile({ setIsLoggedIn }) {
  const onLogOut = useCallback(() => {
    setIsLoggedIn(false);
  }, []);
  return (
    <Card
      actions={[
        //배열이기때문에 키
        <div key="twit">
          짹짹
          <br />0
        </div>,
        <div key="following">
          팔로잉
          <br />0
        </div>,
        <div key="follower">
          팔로워
          <br />0
        </div>,
      ]}
    >
      <Card.Meta avatar={<Avatar>CD</Avatar>} title="Conrad" />
      <Button onClick={onLogOut}>로그아웃</Button>
    </Card>
  );
}

```

물론 Applayout에서 Props르 전달해줘야 하는걸 잊으면 안됩니다.



이번에는 프로필 페이지를 구성해볼까요?

프로필 페이지는 다음과 같이 구성되어있습니다.

```javascript
import React from "react";
import AppLayout from "../components/AppLayout";
import Head from "next/head";
import FollowList from "../components/FollowList";
import NicknameEditForm from "../components/NicknameEditForm";

export default function profile() {
//가짜 데이터
  const followerList = [
    { nickname: "conrad" },
    { nickname: "Roo" },
    { nickname: "boo" },
  ];
  const followingList = [
    { nickname: "conrad" },
    { nickname: "Roo" },
    { nickname: "boo" },
  ];
  return (
    <div>
      <Head>
        <title>프로필 | NodeBird</title>
      </Head>
      
      <AppLayout>
        //닉네임 수정 컴포넌트
        <NicknameEditForm /> 
        //목록 컴포넌트 2개
        <FollowList header="팔로잉 목록" data={followingList} />
        <FollowList header="팔로워 목록" data={followerList} />
      </AppLayout>
    </div>
  );
}

```

다음은 간단한 NicknameEditForm.js부터 만들어볼까요?

```javascript
import React, { useMemo } from "react";
import { Form, Input } from "antd";

function NicknameEditForm() {
  const style = useMemo( //useMemo를 통한 리랜더링 최적화
    () => ({
      marginBottom: "28px",
      border: "1px solid #d9d9d9",
      padding: "30px",
    }),
    []
  );
  
  return (
    <Form style={style}>
      <Input.Search addonBefore="닉네임" enterButton="수정" />
    </Form>
  );
}

export default NicknameEditForm;

```

다음은 복잡할수도 있는 리스트입니다.

```javascript
import React from "react";
import { List, Button, Card } from "antd";
import { StopOutlined } from "@ant-design/icons"; //아이콘은 용량이 크기 때문에 따로있다.

export default function FollowList({ header, data }) {
  return (
    <List
      style={{ marginBottom: 20 }}
      //list이지만 grid로
      grid={{ gutter: 4, xs: 2, md: 3 }}
      size="small"
      header={<div>{header}</div>} //헤더
      loadMore={    //더보기 버튼이 있는 컨테이너
        <div style={{ textAlign: "center", margin: "10px 0" }}>
          <Button>더 보기</Button>
        </div>
      }
      bordered
      dataSource={data} //props로 전달받은 데이터
      renderItem={(item) => ( //데이터의 아이템들을 통해 map함수와 같은 개념
        <List.Item style={{ marginTop: 20 }}>
          <Card actions={[<StopOutlined key="stop" />]}>
            <Card.Meta description={item.nickname} />
          </Card>
        </List.Item>
      )}
    />
  );
}

```

{% hint style="info" %}
보면 복잡해보일수도 있지만 Ant-Design 공식문서에 다 나와있는 내용들입니다.

머리에 외울필요 없이 문서 보면서 쓰고싶은 부분만 가져다가 쓰면 되는 것이죠.
{% endhint %}

결과물을 한번 볼까요?

![&#xB808;&#xC774;&#xC544;&#xC6C3;&#xC774; &#xC798; &#xC7A1;&#xD78C; &#xBAA8;&#xC2B5;&#xC785;&#xB2C8;&#xB2E4;.](../.gitbook/assets/image%20%281%29.png)

다음에는 커스텀 훅을 이용한 회원가입 페이지를 만들어 보겠습니다.



먼저 컴포넌트입니다.

```jsx
 <AppLayout>
      <Head>
        <title>회원가입 | NodeBird </title>
      </Head>
      <Form onFinish={onSubmit}>
        <div>
          <label htmlFor="user-id">아이디</label>
          <br />
          <Input name="user-id" value={id} onChange={onChangeId} />
        </div>
        <div>
          <label htmlFor="user-nickname">닉네임</label>
          <br />
          <Input
            name="user-nickname"
            value={nickname}
            onChange={onChangeNickname}
          />
        </div>
        <div>
          <label htmlFor="user-password">비밀번호</label>
          <br />
          <Input
            name="user-password"
            type="password"
            value={password}
            onChange={onChangePassword}
          />
        </div>
        <div>
          <label htmlFor="user-check">비밀번호확인</label>
          <br />
          <Input
            name="user-check"
            type="password"
            value={check}
            onChange={onChangeCheck}
          />
        </div>
        {passwordError && (
          <PasswordError>비밀번호가 맞지 않습니다.</PasswordError>
        )}
        <Checkbox name="user-term" checked={term} onChange={onChangeTerm}>
          동의합니다.
        </Checkbox>
        {termError && <div style={{ color: "red" }}>약관에 동의해주세요</div>}
        <div style={{ marginTop: 10 }}>
          <Button type="primary" htmlType="submit">
            가입
          </Button>
        </div>
      </Form>
    </AppLayout>
```

ant-design이기 때문에 자세한 언급은 안하겠습니다.

다음으로는 input의 상태값을 관리해줄건데요

input이 많기 때문에 useState를 여러번 사용해줘야 할것 같습니다.

```jsx
  const [id, setId] = useState("");
  const [password, setPassword] = useState("");
```

하지만 이런식으로 모두 작성하는 것은 비효율적입니다.

해결방법은 두가지가 있는데요

1. 커스텀 훅을 만들어 사용하기
2. useState자체로 해결하기

커스텀훅을 먼저 만들어 볼까요?

{% code title="useInput.js" %}
```jsx
import { useState, useCallback } from "react";

export default function useInput(initialValue = null) {
  const [value, setValue] = useState(initialValue);
  const handler = useCallback((e) => {
    setValue(e.target.value);
  }, []);
  return [value, handler];
}

```
{% endcode %}

훅 안에서 change되는 값들을 상태로 바꿔준뒤 valuedhk 바꾸는 함수를 내보내는 형식입니다.

그렇다면 이렇게 구현할 수 있습니다.

```jsx
  const [id, onChangeId] = useInput("");
  const [nickname, onChangeNickname] = useInput("");
  const [password, onChangePassword] = useInput("");
```

간단해지죠?

다음방법은 useState의 기본값을 객체로 만드는 방법입니다.

```jsx
  const [inputs, setInputs] = useState({
    id: '',
    password: '',
    nickname: ''
  });
```

그뒤 값을 설정하는 함수를 구현해보면

```jsx
 const { id,password ,nickname } = inputs; // 비구조화 할당을 통해 값 추출

  const onChange = (e) => {
    const { value, name } = e.target; // 우선 e.target 에서 name 과 value 를 추출
    setInputs({
      ...inputs, // 기존의 input 객체를 복사한 뒤
      [name]: value // name 키를 가진 값을 value 로 설정
    });
  };
```

자 둘다 좋은 방법이니 편한 방법을 쓰면 될 것 같습니다.

완성된 전체 코드를 보겠습니다.

```jsx
import React, { useCallback, useState } from "react";
import AppLayout from "../components/AppLayout";
import { Form, Input, Checkbox, Button } from "antd";
import Head from "next/head";
import styled from "styled-components";
import useInput from "../hooks/useInput";

const PasswordError = styled.div`
  color: red;
`;

export default function signup() {
  const [id, onChangeId] = useInput("");
  const [nickname, onChangeNickname] = useInput("");
  const [password, onChangePassword] = useInput("");

  const [check, setCheck] = useState("");
  const [passwordError, setPasswordError] = useState(false);
  const [term, setTerm] = useState("");
  const [termError, setTermError] = useState(false);

  const onChangeCheck = (e) => {
    setCheck(e.target.value);
    setPasswordError(e.target.value !== password); //이부분이 달라서 못한다.
  };

  const onChangeTerm = useCallback((e) => {
    setTerm(e.target.checked);
    setTermError(false);
  }, []);

  //한번더 체크
  const onSubmit = useCallback(() => {
    if (password !== check) {
      return setPasswordError(true);
    }
    if (!term) {
      return setTermError(true);
    }
    console.log(id, nickname, password);
  }, [password, check, term]);

  return (
    <AppLayout>
      <Head>
        <title>회원가입 | NodeBird </title>
      </Head>
      <Form onFinish={onSubmit}>
        <div>
          <label htmlFor="user-id">아이디</label>
          <br />
          <Input name="user-id" value={id} onChange={onChangeId} />
        </div>
        <div>
          <label htmlFor="user-nickname">닉네임</label>
          <br />
          <Input
            name="user-nickname"
            value={nickname}
            onChange={onChangeNickname}
          />
        </div>
        <div>
          <label htmlFor="user-password">비밀번호</label>
          <br />
          <Input
            name="user-password"
            type="password"
            value={password}
            onChange={onChangePassword}
          />
        </div>
        <div>
          <label htmlFor="user-check">비밀번호확인</label>
          <br />
          <Input
            name="user-check"
            type="password"
            value={check}
            onChange={onChangeCheck}
          />
        </div>
        {passwordError && (
          <PasswordError>비밀번호가 맞지 않습니다.</PasswordError>
        )}
        <Checkbox name="user-term" checked={term} onChange={onChangeTerm}>
          동의합니다.
        </Checkbox>
        {termError && <div style={{ color: "red" }}>약관에 동의해주세요</div>}
        <div style={{ marginTop: 10 }}>
          <Button type="primary" htmlType="submit">
            가입
          </Button>
        </div>
      </Form>
    </AppLayout>
  );
}

```

길긴 하지만 완성본도 볼까요

![&#xC644;&#xC131;!](../.gitbook/assets/image%20%287%29.png)

