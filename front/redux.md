# Redux

원래 리덕스를 넥스트에서 사용하는 것은 상당히 복잡합니다.

next-redux-wrapper를 사용하면 비교적 쉽게 사용할 수 있습니다.

설치해줍시다.

```text
yarn add next-redux-wrapper
```

그리고 루트 디렉토리 안에 store폴더를 만든뒤 configureStore.js파일을 생성해주세요 

버전은 6버전입니다.

기본적인 틀은 다음과 같습니다.

```javascript
import { createWrapper } from "next-redux-wrapper";

const configureStore = () => {};

const wrapper = createWrapper(configureStore);

export default wrapper;

```

다음은 리덕스에 대한 자세한 내용을 볼 수 있도록 wrapper안에 옵션을 주겠습니다.

```javascript
const wrapper = createWrapper(configureStore, {
  debug: process.env.NODE_ENV === "development",
});
```

다음은 \_app.js로 이동해서 제일 상위 컨테이너를 리덕스로 감싸주겠습니다.

```javascript
import React from "react";
import Head from "next/head";
import "antd/dist/antd.css";
import wrapper from "../store/configureStore";


function _app({ Component }) {
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
//감싸준다.
export default wrapper.withRedux(_app);

```

여러 컴포넌트에서 공통적으로 쓰이는 데이터가 있는데, 

컴포넌트는 분리가 되어있기 때문에 그 데이터들을 부모컴포넌트에서 데이터를 받아 자식으로 줘야 합니다.

redux는 중앙에서 데이터를 받아 컴포넌트가 필요로 할때 그 데이터를 뿌려주는데, 

react의 ContextAPI, Redux , graphQL , MobX 등이 있습니다.

각각의 장단점이 있는데 

Redux는 앱이 안정적이지만, 코드량이 많습니다.

MobX는 코드량은 적지만, 난이도가 요구되죠..

간단한 작업은 ContextAPI를 이용하면 되는데, 

ContextAPI를 이용하면 비동기 작업을 처리하기에 있어서 어렵기 때문에 

이번에는 Redux를 사용해보겠습니다.

### 적용법

Next에서는 우리가 아는 리덕스와 다른 방법을 통해 적용시킵니다.

store폴더를 생성한뒤 configureStore파일을 만들고 다음과 같이 작성해주세요

```javascript
import { createWrapper } from "next-redux-wrapper";
import { createStore } from "redux";
import reducer from "../reducers";

const configureStore = () => {
 
  const store = createStore(reducer);
  return store;
};
const wrapper = createWrapper(configureStore, {
  debug: process.env.NODE_ENV === "development",
});
export default wrapper;

```

그리고 _app.js 파일에 들어가서_\_app 컴포넌트를 감싸줍니다.

```javascript
import React from "react";
import Head from "next/head";
import "antd/dist/antd.css";
import wrapper from "../store/configureStore";

function _app({ Component }) {
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
//감싸준다.
export default wrapper.withRedux(_app);

```

리덕스 적용이 완료되었습니다.

리덕스의 액션함수와 리듀서를 만들어 볼까요?

```javascript
const LOG_IN = "users/LOG_IN";
const LOG_OUT = "users/LOG_OUT";

export const logIn = () => ({ type: LOG_IN });
export const logOut = () => ({ type: LOG_OUT });

const initialState = {
  user: {
    isloggedIn: false,
    user: null,
    signUpData: {},
    loginData: {},
  },
  post: {
    mainPosts: [],
  },
};

export default function users(state = initialState, action) {
  switch (action.type) {
    case LOG_IN:
      return { ...state, user: { ...state.user, isloggedIn: true } };
    case LOG_OUT:
      return { ...state, user: { ...state.user, isloggedIn: false } };
    default:
      return state;
  }
}

```

### 개발자 도구

```javascript
yarn add redux-devtools-extension
```

개발자 도구를 다운받은 뒤 스토어에 적용해주면 됩니다.

```javascript
import { createWrapper } from "next-redux-wrapper";
import { createStore, applyMiddleware, compose } from "redux";
import reducer from "../reducers";
import { composeWithDevTools } from "redux-devtools-extension";

const configureStore = () => {
  const middlewares = [];
  const enhancer =
    process.env.NODE_ENV === "production"
      ? compose(applyMiddleware(...middlewares)) //배포용
      : composeWithDevTools(applyMiddleware(...middlewares)); //개발용
  const store = createStore(reducer, enhancer);
  return store;
};
const wrapper = createWrapper(configureStore, {
  debug: process.env.NODE_ENV === "development",
});
export default wrapper;

```

미들웨어는 나중에 적용시키기 위한 공간을 만들어 둔 것입니다.

![](../.gitbook/assets/image%20%282%29.png)

꼭 리듀서에서 불변성을 지켜줘야만 히스토리가 유지되어 동작할 수있는것입니다.

### 리듀서분해

{% tabs %}
{% tab title="index.js" %}
```javascript
import { HYDRATE } from "next-redux-wrapper";
import user from "./user";
import post from "./post";
import { combineReducers } from "redux";

const rootReducer = combineReducers({
  // HYDRATE SSR을 위해 일부러 만들어 진 것입니다.
  index: (state = {}, action) => {
    switch (action.type) {
      case HYDRATE:
        return { ...state, ...action.payload };
      default:
        return state;
    }
  },
  user,
  post,
});
export default rootReducer;

```
{% endtab %}

{% tab title="user.js" %}
```
const LOG_IN = "users/LOG_IN";
const LOG_OUT = "users/LOG_OUT";

export const logIn = () => ({ type: LOG_IN });
export const logOut = () => ({ type: LOG_OUT });

export const initialState = {
  isloggedIn: false,
  user: null,
  signUpData: {},
  loginData: {},
};

export default function reducer(state = initialState, action) {
  switch (action.type) {
    case LOG_IN:
      return { ...state, isloggedIn: true };
    case LOG_OUT:
      return { ...state, isloggedIn: false };
    default:
      return state;
  }
}

```
{% endtab %}

{% tab title="post.js" %}
```
export const initialState = {
  mainPosts: [],
};

export default function reducer(state = initialState, action) {
  switch (action.type) {
    default:
      return state;
  }
}

```
{% endtab %}
{% endtabs %}

컴포넌트를 구성하기 전에postReducer도 구성해볼게요

```javascript
export const initialState = {
  mainPosts: [
    {
      id: 1,
      User: {
        id: 1,
        nickname: "제로초",
      },
      content: "첫번째게시글",
      Images: [
        {
          src:
            "https://media.vlpt.us/images/wooder2050/post/d2764478-dc72-4cc9-9128-f66bfb8b3aa3/reactintroduction.png",
        },
        {
          src:
            "https://media.vlpt.us/images/wooder2050/post/d2764478-dc72-4cc9-9128-f66bfb8b3aa3/reactintroduction.png",
        },
        {
          src:
            "https://media.vlpt.us/images/wooder2050/post/d2764478-dc72-4cc9-9128-f66bfb8b3aa3/reactintroduction.png",
        },
      ],
      Comments: [
        {
          User: {
            nickname: "nero",
          },
          content: "와우",
        },
        {
          User: {
            nickname: "hero",
          },
          content: "오아아아",
        },
      ],
    },
  ],
  imagePaths: [],
  postAdded: false,
};
//액션생성
const ADD_POST = "ADD_POST";
export const addPost = () => ({
  type: ADD_POST,
});
//가짜데이터
const dummyPost = {
  id: 2,
  content: "더미입니다",
  User: {
    id: 1,
    nickname: "conrad",
  },
  Images: [],
  Comments: [],
};

export default function reducer(state = initialState, action) {
  switch (action.type) {
    case ADD_POST:
      return {
        ...state,
        mainPosts: [dummyPost, ...state.mainPosts], //앞에 추가해야 게시글이 위에 나타난다.
        postAdded: true,
      };
    default:
      return state;
  }
}

```

더미데이터를 활용하였습니다.

{% hint style="info" %}
처음에 FE 개발자는 BE개발자와  잘 논의하여서 더미데이터를 잘 만들어 놓는것이 중요합니다.

또한 그 더미데이터는 조금은 바뀔수도 있다는걸 알고 있어야 합니다.
{% endhint %}

