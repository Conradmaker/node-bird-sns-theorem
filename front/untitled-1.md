# Untitled

### shortid

배열데이터\(더미데이터\)에 아이디를 넣어줄때 정말 랜덤한 값을 찾아줘야 하는 경우들이 있는데 이때 유용한 라이브러리가 shortid이다

```text
yarn add shortid
```

```text
import shortId from "shortid";

id: shortId.generate()
```

위처럼 사용할 파일에서 임포트한뒤 사용할 데이터에 호출해주면 된다.



더미데이터에 사용하기 좋은 라이브러리가 또 faker이다.

각종 데이터를 준다.



### immer

만약 아래와 같은 상태가 있다면

```text
mainPosts: [
    {
      id: 1,
      User: {
        id: 1,
        nickname: "Conrad",
      },
      content: "첫번째게시글 #sssss",
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
```

불변성을 유지해주기가 쉽지 않은일이다....

실제로 댓글하나 다는데 불변성을 유지하기 위해 아래의 코드가 필요하다

```text
 case ADD_COMMENT_SUCCESS: {
      //댓글에 접근하기 위해서는 게시글을 먼저 찾고 그 안에 코맨츠배열에 접근해서 추가해야 하는데 불변성을 유지한다.
      const postIndex = state.mainPost.findIndex(
        (y) => y.id === action.data.postId
      );
      const post = state.mainPost[postIndex];
      const Comments = [dummyComment(action.data.content), ...post.Comments];
      const mainPosts = [...state.mainPosts];
      mainPosts[postIndex] = { ...post, Comments };
      return {
        ...state,
        addCommentLoading: false,
        addCommentDone: true,
        addCommentError: null,

        //mainComments: [dummyComment, ...state.mainComments], //앞에 추가해야 게시글이 위에 나타난다.
      };
    }
```

immer를 쓰면 다음과 같다.

```text
 case ADD_COMMENT_SUCCESS: {
        const post = draft.mainPosts.find((v) => v.id === action.data.postId);
        post.Comments.unshift(dummyPost(action.data.content));
        draft.addCommentLoading = false;
        draft.addCommentDone = true;
        break;
      }
```

우선 설치

```text
yarn add immer
```

```text
import produce from "immer";
```

아래가 기본 형식이다.

```text
export default function reducer(state = initialState, action) {
  return produce(state,draft=>{
    
  })
  switch (action.type) {
```

draft를 스테이트로 보고 편하게 바꾸면 immer가 알아서 바꿔준다.

단 state를 건들면 안되고 draft만 바꿔줘야 한다.

예를들어 아래의 코드를 바꿔주면

```text
 case ADD_POST_REQUEST:
        return {
          ...state,
          addPostLoading: true,
          addPostDone: false,
          addPostError: null,
        };
```

```text
 case ADD_POST_REQUEST:
        return:
        draft.addPostLoading = true;
        draft.addPostDone = false;
        draft.addPostError = null;
        break;
```

Faker

더미데이터를 만들때 조금더 현실같은 데이터를 넣어주는 라이브러리이다.

깔아주시고

```text
yarn add faker
```

임포트해주소

```text
import faker from "faker";
```

```text
initialState.mainPosts = initialState.mainPosts.concat( //concat에는 항상대입을!
  Array(20)
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
          src: faker.image.imageUrl(),
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
    }))
);
```

이렇게 간단하게 mainPosts20개를 만들어버렸죠...

나중에 성능 최적화까지 생각해본다면 몇천개씩 만들어서 테스트해도 좋습니다.

자세한 내용이나 종류는 깃허브 공식문서에 나와있습니다.



리덕스툴킷

리덕스팀에서 만든툴인데 편하게 리덕스를 작성할 수 있다.

코드량을 조금씩 줄일 수 있다.

