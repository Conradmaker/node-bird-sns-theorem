# Hashtag 게시글 불러오기

트위터나 인스타를 하다보면 해시태그를 누르면 같은 해시태그를 가진 게시글들이 주르륵 나오는 것 아시죠?? 

이번에는 그 기능을 구현해 보겠습니다!

일단 컴포넌트에서 Hashtag를 누르면 라우팅을 통해 해당 링크로 넘어가주는 기능을 구현했던거 

기억이 나실까요?? \(안날것이라고 생각합니다..\)

### 클릭시 라우팅

```javascript
import React from "react";
import Link from "next/link";

export default function PostCardContent({ postData }) {
  return (
    <div>
      {/* //정규표현식 */}
      {postData.split(/(#[^\s#]+)/g).map((v, i) => {
        if (v.match(/(#[^\s#]+)/g)) {
          return (
            <Link href={`/hashtag/${v.slice(1)}`} key={i}>
              {/* 인덱스로 키를 지정해줬다 바뀔일이 없어서 */}
              <a>{v}</a>
            </Link>
          );
        }
        return v;
      })}
    </div>
  );
}
```

`/hashtag/모시기` 이 링크로 넘어가기 때문에 pages폴더에도

 `pages/hashtag/[tag].js`

이런식으로 다이나믹 라우팅을 해줄 것입니다. 

```javascript
// hashtag/[tag].js
import React, { useCallback, useEffect } from "react";
import { useDispatch, useSelector } from "react-redux";
import { useRouter } from "next/router";
import { END } from "redux-saga";

import axios from "axios";
//HashTag 게시글 불러오는 Request
import { LOAD_HASHTAG_POSTS_REQUEST } from "../../reducers/post";
import PostCard from "../../components/PostCard";
import wrapper from "../../store/configureStore";
import { LOAD_MY_INFO_REQUEST } from "../../reducers/user";
import AppLayout from "../../components/AppLayout";

const Hashtag = () => {
  const dispatch = useDispatch();
  const router = useRouter();
  const { tag } = router.query;
  const { mainPosts, hasMorePosts, loadHashtagPostsLoading } = useSelector(
    (state) => state.post
  );

  useEffect(() => {
    const onScroll = () => {
      if (
        window.pageYOffset + document.documentElement.clientHeight >
        document.documentElement.scrollHeight - 300
      ) {
        if (hasMorePosts && !loadHashtagPostsLoading) {
          dispatch({
            type: LOAD_HASHTAG_POSTS_REQUEST,
            lastId:
              mainPosts[mainPosts.length - 1] &&
              mainPosts[mainPosts.length - 1].id,
            data: tag,
          });
        }
      }
    };
    window.addEventListener("scroll", onScroll);
    return () => {
      window.removeEventListener("scroll", onScroll);
    };
  }, [mainPosts.length, hasMorePosts, tag]);

  return (
    <AppLayout>
      {mainPosts.map((c) => (
        <PostCard key={c.id} post={c} />
      ))}
    </AppLayout>
  );
};

export const getServerSideProps = wrapper.getServerSideProps(
  async (context) => {
    console.log(context);
    const cookie = context.req ? context.req.headers.cookie : "";
    console.log(context);
    axios.defaults.headers.Cookie = "";
    if (context.req && cookie) {
      axios.defaults.headers.Cookie = cookie;
    }
    context.store.dispatch({
      type: LOAD_MY_INFO_REQUEST,
    });
    context.store.dispatch({
      type: LOAD_HASHTAG_POSTS_REQUEST,
      data: context.params.tag, // /hashtag/리액트 => data:리액트
    });
    context.store.dispatch(END);
    await context.store.sagaTask.toPromise();
    return { props: {} };
  }
);

export default Hashtag;

```

router.query.tag 에 데이터가 담겨 있으니 비구조화 할당을 통해 tag를 꺼내주고, infiniteScrolling에 data로 넣어주었으며, 

서버사이드 렌더링시에도 context.params.tag로 action에 data로 담아주었습니다. 

## Redux 상태 재활용

자 이제 Reducer를 작성해 줄 것인데요, 솔찍이 너무 많아요.. 상태가.. 

물론 리팩터링을 해주어도 되기는 하지만, 일단 상태가 많아봤자 좋을게 없으니 재활용을 해보도록 하죠 !!

음.. 페이지 로드시 처음 게시글을 가져오는 것과, hashtag로 처음 게시글을 불러오는 것이 상태가 다를 필요가 없어보입니다. 같은 기능을 수행하는 것이고, 둘다 mainPost라는 배열에 데이터를 넣어주니까요. 

```javascript
export const LOAD_HASHTAG_POSTS_REQUEST = "LOAD_HASHTAG_POSTS_REQUEST";
export const LOAD_HASHTAG_POSTS_SUCCESS = "LOAD_HASHTAG_POSTS_SUCCESS";
export const LOAD_HASHTAG_POSTS_FAILURE = "LOAD_HASHTAG_POSTS_FAILURE";



//reducer
      case LOAD_USER_POSTS_REQUEST: //상태 재사용
      case LOAD_HASHTAG_POSTS_REQUEST:
      case LOAD_POSTS_REQUEST:
        draft.loadPostsLoading = true;
        draft.loadPostsDone = false;
        draft.loadPostsError = null;
        break;
      case LOAD_USER_POSTS_SUCCESS: //상태 재사용
      case LOAD_HASHTAG_POSTS_SUCCESS:
      case LOAD_POSTS_SUCCESS:
        draft.loadPostsLoading = false;
        draft.loadPostsDone = true;
        draft.loadPostsError = null;
        draft.mainPosts = draft.mainPosts.concat(action.data); //앞에 추가해야 게시글이 위에 나타난다.   //어차피 index에서만 쓰이니까
        draft.hasMorePost = action.data.length === 10;
        break;
      case LOAD_USER_POSTS_FAILURE: //상태 재사용
      case LOAD_HASHTAG_POSTS_FAILURE:
      case LOAD_POSTS_FAILURE:
        draft.loadPostsLoading = false;
        draft.loadPostsDone = false;
        draft.loadPostsError = action.error;
        break;
```

* 액션타입만 생성해준뒤 ,
* 리듀서 안에서는 기존  사용하던 상태를 재활용 해주었습니다.  

다음은 Saga입니다 

```javascript
async function loadHashtagPostsAPI(data, lastId) {
  const response = await axios.get(
                //encodeURIComponent = URI에 한글들어가면 에러 때문에
    `/hashtag/${encodeURIComponent(data)}?lastId=${lastId || 0}` 
  );
  return response.data;
}
.
.
.
function* loadHashtagPosts(action) {
  try {
    const data = yield call(loadHashtagPostsAPI, action.data, action.lastId);
    console.log(data);
    yield put({
      type: LOAD_HASHTAG_POSTS_SUCCESS,
      data,
    });
  } catch (err) {
    console.error(err);
    yield put({
      type: LOAD_HASHTAG_POSTS_FAILURE,
      error: err.response.data,
    });
  }
}
.
.
.

function* watchLoadHashtagPosts() {
  yield takeEvery(LOAD_HASHTAG_POSTS_REQUEST, loadHashtagPosts);
}
.
.
.

export default function* postSaga() {
  yield all([
    fork(watchLoadHashtagPosts),
    .
    .
    .
  ]);
}

```

### 서버 라우터

자 딱 아까 리듀서에서 재활용한것처럼 첫페이지 게시글 불러오기와 기능이 비슷하기 때문에 

posts.js라우터에서 복붙을 먼저 해주세요.

그리고 hash태그가 같은 게시글을 찾을 수 있도록 만들어 주는 것입니다. 

routes/hashtag.js를 다음과 같이 만들어주세요 

주석을 꼼꼼히!

```javascript
const express = require("express");
const { Hashtag, User, Post, Image, Comment } = require("../models");
const { Op } = require("sequelize");

const router = express.Router();

router.get("/:hashtag", async (req, res, next) => {
  //hashtag ?
  try {
    const where = {};
 //초기 로딩이 아닐경우 이부분(초기로딩이면 0(false)) lastId보다 작은걸 불러와라
    if (parseInt(req.query.lastId, 10)) {   //이조건도 만족하고 
      where.id = { [Op.lt]: parseInt(req.query.lastId, 10) }; //보다 작은걸 불러오면 된다.
    }
    //DB여러개 가져올떄
    const posts = await Post.findAll({
      where,            //이조건도 만족하고
      limit: 10, //10개만 가져와라
      order: [
        ["createdAt", "DESC"], //게시글늦게 생성된 순서로
        [Comment, "createdAt", "DESC"], //댓글늦게 생성된 순서로
      ],

      include: [
        {
          model: Hashtag,// 한글을 변환해서 보내줬기 떄문에 다시 변화 
          where: { name: decodeURIComponent(req.params.hashtag) },//이조건도 만족하고
        }, 
        { model: User, attributes: ["id", "nickname"] }, //작성자
        { model: User, as: "Likers", attributes: ["id"] }, //좋아요 누른사람
        { model: Image },
        {
          model: Comment, //댓글작성자를 다시
          include: [{ model: User, attributes: ["id", "nickname"] }],
        },
        {
          //리트윗 게시글 포함하도록
          model: Post,
          as: "Retweet",
          include: [
            { model: User, attributes: ["id", "nickname"] },
            { model: Image },
          ],
        },
      ],
    });
    res.status(200).json(posts);
  } catch (e) {
    console.error(e);
    next(e);
  }
});

module.exports = router;
```



