# 검색기능

여러가지 방법으로 검색기능을 구현할 수 있습니다. 

음 이번에는 Link 와 Push를 통한 두가지 방식을 알아볼 것입니다. 

각각 다른 상황에 맞게 사용할 수 있겠죠? 

* push는 검색창과 같이 실행시키면 함수안에서 router가 설정 경로로 이동시켜줍니다. 
* link는 a태그와 같습니다. 

## Router.push

레이아웃 안에는 다음과 같은 검색창이 있습니다. 

```javascript
 const [searchInput, onChangeSearchInput] = useInput("");
  <Menu.Item>
          <SearchInput
            enterButton
            style={{ verticalAlign: "middle" }}
            value={searchInput}
            onChange={onChangeSearchInput}
            onSearch={onSearch} //검색을 하는 함수
          ></SearchInput>
</Menu.Item>
```

자, 그렇다면 onSearch를 호출하면 안에서 해당 경로로 이동을 시켜주면 되겠죠? 

```javascript
 import Router from "next/router";
 
   const onSearch = useCallback(() => {
    Router.push(`/hashtag/${searchInput}`);
  }, [searchInput]);
```

이렇게 되면 예를들어 리액트를 입력한다고 하면,

 `hashtag/리액트` 이런 경로로 이동이 되겠죠?

자 전에 동적라우팅을 통해 만든 hashtag/\[tag\].js 기억나시나요? 그렇습니다. 이렇게하면  검색기능이 구현이 완료되었습니다!

```javascript
import React, { useCallback } from "react";
import Link from "next/link";
import { Menu, Input, Row, Col } from "antd";
import LoginForm from "./loginForm";
import styled from "styled-components";
import UserProfile from "./UserProfile";
import { useSelector } from "react-redux";
import useInput from "../hooks/useInput";
import Router from "next/router";

const SearchInput = styled(Input.Search)`
  vertical-align: center;
`;
export default function AppLayout({ children }) {
  const { me } = useSelector((state) => state.user);
  const [searchInput, onChangeSearchInput] = useInput("");

  const onSearch = useCallback(() => {
    Router.push(`/hashtag/${searchInput}`);
  }, [searchInput]);

  return (
    <>
      <Menu mode="horizontal">
.
.
.
        <Menu.Item>
          <SearchInput
            enterButton
            style={{ verticalAlign: "middle" }}
            value={searchInput}
            onChange={onChangeSearchInput}
            onSearch={onSearch}
          ></SearchInput>
        </Menu.Item>
      </Menu>
      <Row gutter={8}>
        <Col xs={24} md={6}>
          {me ? <UserProfile /> : <LoginForm />}
        </Col>
        <Col xs={24} md={12}>
          {children}
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
      </Row>
    </>
  );
}
```

## Link 

자 이번에는 next에 내장되어있는 link를 통해 검색? 기능을 구현해보겠습니다. 

게시글에 있는 아바타를 누르게 되면 해당 사람의 유저정보, 게시글을 볼 수 있도록 만들어보겠습니다. 

아 이번에는 저번시간에 만들어놓은 pages/user/\[id\].js 동적 라우팅페이지를 활용하면 되겠죠? 

기본적인 Link사용법은 이미 컴포넌트를 구성할때 알아보았으니 바로 사용해보도록 하지요. 

```javascript
<Card.Meta
      avatar={
             <Link href={`/user/${post.Retweet.User.id}`}>
                <a>
                   <Avatar>{post.Retweet.User.nickname[0]}</Avatar>
                </a>
             </Link>
             }
   title={post.Retweet.User.nickname}
   description={<PostCardContent postData={post.Retweet.content} />}
/>
```

 ```/user/${post.Retweet.User.id}```  라는 링크를 만들어 주었는데 이는 곧 선택한 유저아이디가 4라면,

`localhost:3000/user/4` 라는 페이지로 이동되고, 이 페이지는 동적라우팅페이지로 만들어져 있죠? 

그렇다면 그 페이지에서 SSR을 통해 데이터를 가져오는 방식입니다. 

