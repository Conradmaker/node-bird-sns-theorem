# 다이나믹 라우터

## getStaticProps

getServerSideProps는 써봤는데 나머지 getStaticProps는 언제 쓸까요?

음 예를 들어보기 위해 특정회원 1명의 정보를 받아올 수 있는 about.js라는 페이지를 만들어볼게요. 

```javascript
import React from "react";
import { useSelector } from "react-redux";
import Head from "next/head";
import { END } from "redux-saga";

import { Avatar, Card } from "antd";
import AppLayout from "../components/AppLayout";
import wrapper from "../store/configureStore";
import { LOAD_USER_REQUEST } from "../reducers/user";

const Profile = () => {
  const { userInfo } = useSelector((state) => state.user);

  return (
    <AppLayout>
      <Head>
        <title>Conrad | NodeBird</title>
      </Head>
      {userInfo ? ( //SSR이 되야만 userInfo가 들어있다.
        <Card
          actions={[
            <div key="twit">
              짹짹
              <br />
              {userInfo.Posts}
            </div>,
            <div key="following">
              팔로잉
              <br />
              {userInfo.Followings}
            </div>,
            <div key="follower">
              팔로워
              <br />
              {userInfo.Followers}
            </div>,
          ]}
        >
          <Card.Meta
            avatar={<Avatar>{userInfo.nickname[0]}</Avatar>}
            title={userInfo.nickname}
            description="하하하하"
          />
        </Card>
      ) : null}
    </AppLayout>
  );
};

export const getStaticProps = wrapper.getStaticProps(async (context) => {
  console.log("getStaticProps");
  context.store.dispatch({
    type: LOAD_USER_REQUEST, //특정한 사용자정보를 가져오는 것.
    data: 1,
  });
  context.store.dispatch(END);
  await context.store.sagaTask.toPromise();
});

export default Profile;
```

사용방법은 ServerSideProps와 다르지 않습니다. 

백앤드도 작성해주세요 \(reducer와 saga는 생략하겠습니다\)

```javascript
//특정 사용자를 가져오는 라우터
router.get("/:id", async (req, res, next) => {
  try {
    const fullUserWithoutPassword = await User.findOne({
      where: { id: req.params.id },
      attributes: { exclude: ["password"] }, //비밀번호 빼고
      //관계데이터
      include: [
        { model: Post, attributes: ["id"] }, // 개시글 말고 아이디만 (몇개 몇명인지만 세니까)
        { model: User, as: "Followings", attributes: ["id"] }, //as 써줬으면 여기서도 써줘야함
        { model: User, as: "Followers", attributes: ["id"] },
      ],
    });
    if (fullUserWithoutPassword) {
      //다른 사람의 데이터이기 때문에 개신정보를 잘 숨겨서 보내준다.
      const data = fullUserWithoutPassword.toJSON(); // Sequelize에서 보내준 데이터를 수정하기 위해
      data.Posts = data.Posts.length;
      data.Followers = data.Followers.length;
      data.Followings = data.Followings.length;
      res.status(200).json(data); //쿠키 있으면 보내고
    } else {
      res.status(403).send("그런 유저 없습니다."); //user/10000이런경우를 위해
    }
  } catch (e) {
    console.error(e);
    next(e);
  }
});

module.exports = router;
```

여기서는 주의해야 할 점이 다른 사용자의 정보를 다뤄야 하기 때문에 프론트 컴포넌트 상에서 `***.length` 와 같이 데이터를 통해 조회하기 보다는 백앤드 서버에서 숫자로 값을 만들어서 보내주었습니다. 

과연 getServerSideProps / getStaticProps 차이점이 무엇일까요? 

> getServerSideProps : 동적 데이터를 로딩할때 사용한다. \(변하는데이터\)
>
> getStaticProps : 정적 데이터를 로딩할때 사용한다. \(변하지 않는 데이터\)

getStaticProps는 빌드할때 처음부터 html로만들어주기 때문에 쓸수 있으면 쓰는게 좋지만, 쓰는 환경은 까다롭다. 

getStaticProps는 서버로 요청할때 html로만들어주기 때문에 서버에 부하가 조금 더하다 .

무슨 말인지 이해가 되었나요? StaticProps를 쓸 수 있는 상황이라면 쓸 수 있는게 좋지만, 

막상 쓸수 있는 곳은 많지 않고, 사용방법도 같기 떄문에 둘다 알아놓는것을 추천합니다. 



## 다이나믹 라우팅

이번에는 특정 게시글만을 불러오는 게시글을 작성해보겠습니다. 

원래 다이나믹 라우팅이라는 기술은 next에서 지원하지 않았기 때문에 백앤드 서버에서 구현해 주어야만 했습니다. 

하지만 next 9 버전부터는 이제 많은 기능이 추가되면서 다이나믹 라우팅도 추가되었죠!!

pages폴더안에 post/\[id\].js 파일을 만들어주세요. 

```javascript
+import { useRouter } from "next/router";
import wrapper from "../../store/configureStore";
import { LOAD_MY_INFO_REQUEST } from "../../reducers/user";
import { END } from "redux-saga";
import axios from "axios";
import { LOAD_POST_REQUEST } from "../../reducers/post";
import PostCard from "../../components/PostCard";
import { useSelector } from "react-redux";
import AppLayout from "../../components/AppLayout";
import { useEffect } from "react";

const Post = () => {
  const { singlePost, loadPostError } = useSelector((state) => state.post);
  const router = useRouter();
  const { id } = router.query; //이렇게 URI데이터를 가져온다 .

  useEffect(() => {
    alert(loadPostError);
  }, [loadPostError]);
  return (
    <>
      {singlePost ? (
        <AppLayout>
          <PostCard post={singlePost} />
        </AppLayout>
      ) : (
        <AppLayout>해당 아이디에 해당하는 글이 없네요..</AppLayout>
      )}
    </>
  );
};

export const getServerSideProps = wrapper.getServerSideProps(
  async (context) => {
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
      type: LOAD_POST_REQUEST,
      data: context.params.id, //이렇게 URI데이터를 가져옴
    });
    context.store.dispatch(END);
    await context.store.sagaTask.toPromise();
  }
);

export default Post;
```

특징은 다음과 같습니다. 

* \[id\].js 에서 대괄호\[ \] 안에 있는 id가 parameter에 담기게 됩니다. 
  * localhost:3000/post/2 -&gt; id:2
*  데이터를 꺼내올때에는 router.query안에 id가 담겨있습니다. 
* SSR에서 데이터를 가져올 때에는 context.params or context.query 안에 담겨있습니다. 

이렇게 페이지별로 데이터를 받아와 서버로 값을 넘겨주면 되겠죠? 

백앤드에서 처리

```javascript
router.get("/:postId", async (req, res, next) => {
  try {
    const post = await Post.findOne({
      where: { id: req.params.postId },
      include: [
        {
          model: User,
          attributes: ["id", "nickname"],
        },
        {
          model: Image,
        },
        {
          model: Comment,
          include: [
            {
              model: User,
              attributes: ["id", "nickname"],
              order: [["createdAt", "DESC"]],
            },
          ],
        },
        {
          model: User, // 좋아요 누른 사람
          as: "Likers",
          attributes: ["id"],
        },
      ],
    });
    if (!post) {
      return res.status(403).send("해당 아이디에 해당하는 글이 없네요..");
    }
    res.status(200).json(post);
  } catch (error) {
    console.error(error);
    next(error);
  }
});
```



