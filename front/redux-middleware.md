# Redux middleware

미들웨어는 리덕스에서 더 많은 기능을 할 수 있게 해주는 것이다.

그중 Redux-thunk , Redux-saga가 많이 쓰이게 되는데, 

비동기 작업에 있어서 효율적이다..

Redux-thunk는 비동기 액션함수 하나에서 dispatch를 여러번 즉 비동기 액션을 줄 수 있다.

쉽게 말해 dispatch여러개를 한번에 묶어서 사용할 수 있게 해준다.

```text
yarn add redux-thunk
```

먼저 redux-thunk를 설치해준다.

그 뒤 스토어를 만드는 곳에서 middleware를 적용해준다.

```jsx
import { createWrapper } from "next-redux-wrapper";
import { createStore, applyMiddleware, compose } from "redux";
import reducer from "../reducers";
import { composeWithDevTools } from "redux-devtools-extension";
import ReduxThunk from "redux-thunk"; //import

const configureStore = () => {
  const middlewares = [ReduxThunk]; //적용
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

미들웨어는 3단 함수인데, 사실 아래의 코드가 전부이다.

```jsx
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => (next) => (action) => {
    if (typeof action === 'function') {  //액션이 함수인경우 
      return action(dispatch, getState, extraArgument);  //비동기처리를 한다.
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
© 2020 GitHub, Inc.
```

thunk를 사용하는 이유는 

로그인을 원하고 로그아웃을 한다고 해서 바로 되는 것은 아니라,

백앤드서버를 거치기 때문에 Request Action이 되게 되는데, 

비동기 요청은 LOADING, SUCCESS, ERROR 3단계로 보통 대다수가 이루어진다.

예를들면 아래와 같이 구현할 수 있다.

```jsx
export const getPosts = () => async dispatch => {
  dispatch({ type: GET_POSTS }); // 요청이 시작됨
  try {
    const posts = await postsAPI.getPosts(); // API 호출
    dispatch({ type: GET_POSTS_SUCCESS, posts }); // 성공
  } catch (e) {
    dispatch({ type: GET_POSTS_ERROR, error: e }); // 실패
  }
};
```

{% hint style="info" %}
물론 dispatch 말고도 getState를 받아와서 함수안에서 상태를 조회할 수도 있다.
{% endhint %}

그렇다면 , thunk의 기능은 여기서 끝난다.

그렇다면 Saga를 더 많이 쓰이는 이유는 뭘까?

Saga에서는 delay를 만들어주는 기능이 있으며,

실수로 클릭을 두번했을경우 첫번째 클릭은 무시하고, 마지막 클릭만 실행할 수도 있다.

이러한경우 Front server에서 DDOS공격을 할수도 있고, 

최악의경우 Self -ddos공격을 할 수도 있다.

saga의 throttle을 이용해 그걸 막을수 있으니 한번 배워봐야 겠다.

next에서는 saga의 적용법이 조금 다르기 때문에 아래를 설치해주자

```jsx
yarn add next-redux-saga
```

{% tabs %}
{% tab title="config store" %}
```jsx
import { createWrapper } from "next-redux-wrapper";
import { createStore, applyMiddleware, compose } from "redux";
import reducer from "../reducers";
import { composeWithDevTools } from "redux-devtools-extension";
import createSagaMiddleware from "redux-saga";
import rootSaga from "../sagas"; //미리 import

function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => (next) => (action) => {
    if (typeof action === "function") {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const configureStore = () => {
  const sagaMiddleware = createSagaMiddleware(); //만드들기
  const middlewares = [sagaMiddleware]; //적용
  const enhancer =
    process.env.NODE_ENV === "production"
      ? compose(applyMiddleware(...middlewares)) //배포용
      : composeWithDevTools(applyMiddleware(...middlewares)); //개발용
  const store = createStore(reducer, enhancer);
  store.sagaTask = sagaMiddleware.run(rootSaga); //추가(rootSaga는 직접 작성)
  return store;
};
const wrapper = createWrapper(configureStore, {
  debug: process.env.NODE_ENV === "development",
});
export default wrapper;

```
{% endtab %}

{% tab title="최상위page" %}
```jsx
import React from "react";
import Head from "next/head";
import "antd/dist/antd.css";
import wrapper from "../store/configureStore";
import withReduxSaga from "next-redux-saga"; //next와 redux-saga를 연결해주는 library

function _app({ Component }) {
  return (
    <>
      <Head>
        <meta charSet="utf-8"></meta>
        <title>Wongeun</title>
      </Head>
      <Component />
    </>
  );
}
//감싸준다.
export default wrapper.withRedux(withReduxSaga(_app)); //saga로 한번더 덮어준다.
```
{% endtab %}
{% endtabs %}

다음은 sagas폴더 생성



function\* = generator함수

generator함수는 .next\(\)를 붙여줘야 실행된다. 

yeild가 있는곳에서 멈추기 때문에 중단점이 있는 함수이다. 

그래서 saga에서는 while\(true\) 반복문도 무한반복이 되지 않는다.

실제로 JS에서 무한의 개념을 표현하고 싶을 때 generator를 사용한다. 

또한 특정 이벤트\(.next\(\)\)가 실행되었을때 실행되기 때문에 EventListner로 볼수있다.

saga에 대한 자세한 내용은 내 gitbook에 있는 내용과 동일

```jsx
import { all, fork, take, call, put } from "redux-saga/effects";
import axios from "axios";
function logInAPI(data) {
  //제네레이터 아님!!
  const response = axios.post("/api/login");
  return response.data;
}
function addPostAPI(data) {
  const response = axios.post("/api/post");
  return response.data;
}
function* addPost(action) {
  yield put({
    type: "ADD_POST_REQUEST",
  });
  try {
    const data = yield call(addPostAPI, action.data);
    yield put({
      type: "ADD_POST_SUCCESS",
      data,
    });
  } catch (e) {
    yield put({
      type: "ADD_POST_ERROR",
      data: e.response.data,
    });
  }
}
function* logIn(action) {
  yield put({
    type: "LOG_IN_REQUEST",
  });
  try {
    const data = yield call(logInAPI, action.data);
    yield put({
      type: "LOG_IN_SUCCESS",
      data,
    });
  } catch (e) {
    yield put({
      type: "LOG_IN_ERROR",
      data: e.response.data,
    });
  }
}

function* logOut() {
  yield put({
    type: "LOG_OUT_REQUEST",
  });
  try {
    const data = yield call(logInAPI);
    yield put({
      type: "LOG_OUT_SUCCESS",
      data,
    });
  } catch (e) {
    yield put({
      type: "LOG_OUT_ERROR",
      data: e.response.data,
    });
  }
}
function* watchLogin() {
  yield takeEvery("LOG_IN_REQUEST", logIn); //LOG_IN이라는 액션이 실행될때까지 기다린다. 실행되면 logIn실행
}
function* watchLogOut() {
  yield takeEvery("LOG_OUT_REQUEST", logOut);
}
function* watchAddPost() {
  yield takeEvery("ADD_POST_REQUEST", addPost);
}

export default function* rootSaga() {
  // all은 배열을 받아 한방에 실행해준다
  yield all([
    //fork는 함수를 실행한다.
    fork(watchLogin), //call이랑 구분되지만 지금은 상관없다..
    fork(watchLogin), //차이점은 call은 비동기 실행이지만, fork는 동기이다.
    fork(watchLogin),
    //한마디로 fork, call은 함수를 실행할 수 있게 해주는데 all은 그것들을 동시에 실행할 수 있게 만듬.
  ]);
}

```



