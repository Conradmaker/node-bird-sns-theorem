# 준비

서버사이드 렌더링에 들어가기에 앞서 설정을 먼저 해줘야 합니다. 

next 에서 제공하는 메서드 4개가 있는데, 리덕스와 함께 사용하면 몇가지 문제가 있기 때문에 

next-redux-wrapper에서 제공하는 것들로 SSR을 구성하겠습니다. 

getInitialProps는 뭔가 곧 없어질 것 같기에 9버전부터 새로나온

getStaticProps getServerSideProps, getStaticPath를 사용할 것인데요? 

차이점은 사용하면서 익혀보도록 하겠습니다!

일단 

메인 페이지 index.js입니다. 

원래는 useEffect를 통해 마운트 될때 dispatch로 action을 실행해 주었는데, 이제는 마운트 되기 전에 프론트서버 안에서 dispatch를 해주도록 하겠습니다. 

```javascript
//이렇게 해주면 이부분이 index보다 먼저 실행된다.
//화면을 그리기 전에 서버에서 이부분을 실행
export const getServerSideProps = wrapper.getServerSideProps((context) => {
  console.log(context); //context에는 무엇이 들어있을까?
  //기존에 useEffect에 있던걸 가져옴
  context.store.dispatch({
    type: LOAD_MY_INFO_REQUEST,
  });
  context.store.dispatch({
    type: LOAD_POSTS_REQUEST,
  });
});

export default index;
```

하지만 이렇게 해줘도 redux state는 비어있게 됩니다..

 여기서 나오게 되는데 HYDRATE인데, HYDRATE는 저번에 store를 구성해줄때 이미 만들어 놓았죠? 

HYDRATE는 위에서 작성한 내용이 실행되면 결과를 받아 payload안에 넣어줍니다. 

![&#xC5EC;&#xAE30;&#xC11C; payload&#xC548;&#xC5D0; &#xBC1B;&#xC544;&#xC9C4;&#xB2E4;.](../.gitbook/assets/image%20%2813%29.png)

하지만, 지금도 정보가 안들어 있다고, state들이 한번더 감싸져 있습니다. 

한번rootReducer를 볼까요? 

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
```

이걸 확장시켜서 state안에 직접 payload값을 넣어주어야 합니다. 

우선 Reducer의 기본 형태로 만들어 주겠습니다. 

```javascript
const rootReducer2 = (state,action)=>{
  switch (action.type) {
    case value:
    
    default:
    
  }
}
```

default에 기본으로 반환할 값을 combineReducer를 통해 넣어줍니다. 

```javascript
const rootReducer = (state, action) => {
  switch (action.type) {
    case value:
    default:
      const combinedReducer = combineReducers({
        user,
        post,
      });
      return combinedReducer(action,state)
  }
};
```

그 뒤 HYDRATE 케이스에 상태값을 SSR시 hydrate에 담겨지는 payload값으로 해줍니다. 

```javascript
const rootReducer = (state, action) => {
  switch (action.type) {
    case HYDRATE:
      console.log("HYDRATE", action);
      return action.payload;
    default: {
      const combinedReducer = combineReducers({
        user,
        post,
      });
      return combinedReducer(state, action);
    }
  }
};
```

> 자 이제 상태가 감싸져 있는 문제는 해결이 되었습니다.

이번에는 데이터가 비어있는 문제를 해결해보겠습니다. 

index.js로 돌아와서 dispatch를 해주었는데 담겨있지 않은 이유는 데이터들이 돌아오기 전에 코드가 모두 끝났기 때문입니다. 

이런 경우 비동기 작업을해주어야 겠죠? 



```javascript
//이렇게 해주면 이부분이 index보다 먼저 실행된다.
//화면을 그리기 전에 서버에서 이부분을 실행
export const getServerSideProps = wrapper.getServerSideProps((context) => {
console.log("getServerSideProps start");
  context.store.dispatch({
    type: LOAD_MY_INFO_REQUEST,
  });
  context.store.dispatch({
    type: LOAD_POSTS_REQUEST,
  });
  
   context.store.dispatch(END); //saga에서 불러옴 (끝날때까지 기다리라고)
   console.log("getServerSideProps end");
   await context.store.sagaTask.toPromise(); //(sagaTask)는 스토어에서 등록함
});

export default index;
```

END를 dispatch하는 것은 모든 Dispatch가 끝날때 까지 기다리라는 것이고, 

context.store에 들어있는 sagaTask는 스토어에서 등록한 것입니다. 

```javascript
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
  debug: false, //process.env.NODE_ENV === "development",
});
```

자 이제 확인을 해보면 페이지가 로드되기 전에 게시글 데이터들이 들어있을 것입니다. 

하지만, 로그인은 유지가 되지 않는데요, 이유는 다음장에서 살펴보겠습니다. 

