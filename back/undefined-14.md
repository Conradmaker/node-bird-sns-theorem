# 인피니트 스크롤링 \(완성\)

저번시간에 lastId방식을 완성시키지 않았었습니다.  

lastId는 다음과 같이 구현해 줄 것입니다. 

* 프론트에서 마지막 메인포스트의 아이디데이터를 백앤드서버로 보내준다. 
* 백앤드 서버에서는 그보다 아이디가 작은 10개를 보내준다. 
* 34개의 글이있다면 마지막에는 4개의 데이터만 올것이다. 
* 만약 10개의 데이터가 왔다면 hasmorePosts를 true로 아니면 false로해서 더이상 로드를 막는다.

일단 useEffect안에서 lastId를 정해줄 것입니다. 

```javascript
  useEffect(() => {
    function onScroll() {
      if (
        window.pageYOffset + document.documentElement.clientHeight >
        document.documentElement.scrollHeight - 300
      ) {
        if (hasMorePost && !loadPostsLoading) {
          const lastId = mainPosts[mainPosts.length - 1]?.id;
          dispatch({ type: LOAD_POSTS_REQUEST, lastId });
        }
      }
    }
    window.addEventListener("scroll", onScroll);
    return () => {
      window.removeEventListener("scroll", onScroll);
      //리턴해서 다시 삭제 안해주면 메모리에 쌓이게 된다.
    };
  }, [hasMorePost, loadPostsLoading, mainPosts]);
```

```javascript
const lastId = mainPosts[mainPosts.length - 1]?.id;
//mainPost가 처음에는 없기때문에 ? 안전장치
```

리듀서함수에는 다음 내용으로 바꿔주고

```javascript
draft.mainPosts = draft.mainPosts.concat(action.data); //앞에 추가해야 게시글이 위에 나타난다.
draft.hasMorePost = action.data.length === 10;
```

saga에서 서버로 요청을 보낼때 다음과 같이 해줍니다.

```javascript
async function loadPostAPI(lastId) {
  //data가 undifined인 경우 0
  const response = await axios.get(`/posts?lastId=${lastId || 0}`); //queryString (data 캐싱도 할 수 있다.)
  return response.data;
}
```

처음에 요청을 보낼때는 data가 undifined이기 때문에 0을 넣어줬습니다. 

이제 라우터로 넘어가게 되면

일단 LastId방식을 사용하기 위해서는 Op라는 Sequelize매소드가 필요합니다. 

```javascript
const { Op } = require("sequelize"); //오피
```

```javascript
router.get("/", async (req, res, next) => {
  try {
    const where = {};
    if (parseInt(req.query.lastId, 10)) {
      //초기 로딩이 아닐경우 이부분(초기로딩이면 0) lastId보다 작은걸 불러와라
      where.id = { [Op.lt]: parseInt(req.query.lastId, 10) }; //보다 작은걸 불러오면 된다.
    }
    //DB여러개 가져올때
    const posts = await Post.findAll({
      where, //위에서 작성한 조건 추가
      limit: 10, //10개만 가져와라
      order: [
        ["createdAt", "DESC"], //게시글늦게 생성된 순서로
        [Comment, "createdAt", "DESC"], //댓글늦게 생성된 순서로
      ],

      include: [
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
```

자 이것으로, 백앤드 서버가 완성되었습니다!!!!

다음부터는 next.js를 활용한 SSR \(서버사이드 렌더링을 해보겠습니다\)

