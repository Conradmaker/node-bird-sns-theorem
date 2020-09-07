# 게시글 불러오기

이번에는 게시글을 불러오는 기능을 구현해보도록 하겠습니다. 

그 전까지는 더미데이터를 가지고 불러왔지만, 이제는 진짜 우리가 DB에 입력한 게시글을 불러와보아요.

일단 우리의 게시글들은 InfiniteScrolling으로 불러왔습니다. 

두가지 방법이 있겠는데요!!

* limit & offset 방식의 infinite scrolling
* limit & lastId 방식의 infinite scrolling이 있겠습니다.

먼저 프론트부분을 셋팅해볼까요

## limit & offset 방식

사실 실무에서 이 방식은 잘 사용되지는 않습니다. 

일단 코드를 살펴볼까요

```javascript
//limit & offset 방식의 infiniti scrolling
const express = require("express");
const { Post } = require("../models/");

const router = express.Router();

router.get("/", async (req, res, next) => {
  try {
    //DB여러개 가져올떄
    const posts = await Post.fingAll({
      limit: 10, //10개만 가져와라
      offset: 10, //11~20만큼 가져와라
      order: [["createdAt", "DESC"]], //늦게 생성된 순서로
    });
    res.status(200).json(posts);
  } catch (e) {
    console.error(e);
    next(e);
  }
});
```

이 방법은 기본적으로 SQL기능을 통해 구현하는 방법인데 치명적인 단점이 있습니다. 

* 중간에 이용자가 게시글을 지우거나 새로 만들면 버그가 발생합니다.
* 새로운 게시글을 추가하면 하나가 중복되어서 가져와지며,
* 하나를 삭제하면 하나가 안불러와질 수도 있습니다.

우리는 대신 lastId방식을 사용할 것입니다. 

## limit & Lastid

우리는 복수posts와 단수post를 구분하기 위해 routes폴더에 posts.js를 불러와 줄 것입니다. 

lastid는 SQL방식이 아닌 자바스크립트를 이용한 방법을 사용할 것입니다. 

일단 DB를 찾아서 가져와주세요

```javascript
const express = require("express");
const { Post, Comment } = require("../models");

const router = express.Router();
router.get("/", async (req, res, next) => {
  try {
    //DB여러개를 가져올때
    const posts = await Post.findAll({
      limit: 10, //10개씩 가져온다.
      order: [
        ["createdAt", "DESC"], //늦게 작성된 순서로
        [Comment, "createdAt", "DESC"], //댓글도 작성역순으로
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

이제 프론트에서 요구하는 관계데이터들을 가져와줘야 겠죠?

```javascript
const express = require("express");
const { Post, Comment, User, Image } = require("../models");

const router = express.Router();
router.get("/", async (req, res, next) => {
  try {
    //DB여러개를 가져올때
    const posts = await Post.findAll({
      limit: 10, //10개씩 가져온다.
      order: [
        ["createdAt", "DESC"], //늦게 작성된 순서로
        [Comment, "createdAt", "DESC"], //댓글도 작성역순으로
      ],
      include: [
        { model: User, attributes: ["id", "nickname"] },
        { model: Image },
        {
          //댓글은 작성자까지 가져와줍니다.
          model: Comment,
          include: [{ model: User, attributes: ["id", "nickname"] }],
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

자 이제 프론트에서 리듀서만 더미가 아닌 실제 데이터로 바꿔주면 되겠죠?

