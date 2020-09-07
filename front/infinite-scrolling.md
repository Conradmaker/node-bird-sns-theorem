# infinite scrolling

더미데이터의 메인포스트를 빈배열로 만들어준뒤 아래와 같이 해주세요

```jsx
export const generateDummyPost = (number) => {
  Array(number)
    .fill()
    .map(() => ({
      id: shortId.generate(), //shortid 사용
      User: {
        id: shortId.generate(), //shortid 사용
        nickname: faker.name.findName(), //faker사용
      },
      content: faker.lorem.paragraph(),
      Images: [
        {
          src: faker.image.image(),
        },
      ],
      Comments: [
        {
          User: {
            id: shortid.generate(),
            nickname: faker.name.findName(),
          },
          content: faker.lorem.sentence(),
        },
      ],
    }));
};
initialState.mainPosts = initialState.mainPosts.concat(generateDummyPost(10));
```

```jsx
//상태
  hasMorePost: true,

  loadPostsLoading: false,
  loadPostsDone: false,
  loadPostsError: null,

//액션
export const LOAD_POSTS_REQUEST = "LOAD_POSTS_REQUEST";
export const LOAD_POSTS_SUCCESS = "LOAD_POSTS_SUCCESS";
export const LOAD_POSTS_FAILURE = "LOAD_POSTS_FAILURE";

//리듀서
 case LOAD_POSTS_REQUEST:
        draft.loadPostsLoading = true;
        draft.loadPostsDone = false;
        draft.loadPostsError = null;
        break;
      case LOAD_POSTS_SUCCESS:
        draft.loadPostsLoading = false;
        draft.loadPostsDone = true;
        draft.loadPostsError = null;
        draft.mainPosts = action.data.concat(draft.mainPosts); //앞에 추가해야 게시글이 위에 나타난다.
        draft.hasMorePost = draft.mainPosts.length < 50;
        break;
      case LOAD_POSTS_FAILURE:
        draft.loadPostsLoading = false;
        draft.loadPostsDone = false;
        draft.loadPostsError = action.error;
        break;
        
        
//사가

function loadPostAPI(data) {
  const response = axios.post("/api/post");
  return response.data;
}

function* loadPost(action) {
  try {
    yield delay(1000);
    const id = shortid.generate();
    // const data = yield call(loadPostAPI, action.data);
    yield put({
      type: LOAD_POSTS_SUCCESS,
      data: data: generateDummyPost(10),
    });
  } catch (e) {
    yield put({
      type: LOAD_POSTS_FAILURE,
      data: e.response.data,
    });
  }
}

function* watchLoadPost() {
  yield takeEvery(LOAD_POSTS_REQUEST, loadPost);
}

export default function* postSaga() {
  yield all([
    fork(watchAddComment),
    fork(watchAddPost),
    fork(watchLoadPost),
    fork(watchRemovePost),
  ]);
}

```

인덱스 페이지로 가서

```jsx
  useEffect(() => {
    dispatch({ type: LOAD_POSTS_REQUEST });
  }, [dispatch]);
```

추가해주면 컴포넌트가 마운트될때 로드되고, 스크롤이벤트를 추가해줍니다

```jsx
function onScroll() {
      console.log(
        window.scrollY,                         //얼마나 내렸는지
        document.documentElement.clientHeight, //화면에 보이는길이
        document.documentElement.scrollHeight //총길이
        //위 3가지가 가장 많이 쓰인다 
        // 총길이 = 얼마나 내렸는지 + 화면에 보이는길이
      );
    }
```

```jsx
useEffect(() => {
    function onScroll() {
      console.log(
        window.scrollY,
        document.documentElement.clientHeight,
        document.documentElement.scrollHeight
      );
      if (
        window.scrollY + document.documentElement.clientHeight ===
        document.documentElement.scrollHeight
      ) {
        dispatch({ type: LOAD_POSTS_REQUEST });
      }
    }
    window.addEventListener("scroll", onScroll);
    return () => {
      window.removeEventListener("scroll", onScroll);
      //리턴해서 다시 삭제 안해주면 메모리에 쌓이게 된다.
    };
  }, []);
```

## react virtualize

모바일에는 컴포넌트가 너무 많이 렌터되면 메모리를 잡아먹게되는데, 

인스타그램과 같이 리스트가 많이 나오는 것들은 효율적으로 화면에는 3개씩만 보여줘서, 

화면에서는 랙이 없어지도록 도와준다.

검색 ㄱㄱ!



