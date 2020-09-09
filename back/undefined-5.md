# 게시글,댓글 작성

자 이번에는 핵심기능인 게시글 작성기능을 구현해 볼 것인데요, 

많이 어렵지는 않습니다. 

먼저 게시글들을 처리하기 위한 라우터를 만들어 볼까요? 

routes폴더 안에 post.js를 만들어주세요 그리고 app.js에서 라우터 등록해주시고요.

일단 프론트에서 받아온 데이터를 DB에 등록해줘야 겠죠?

## 게시글 작성

### 받아온 데이터 db에 등록

```javascript
const express = require("express");
const { Post } = require("../models");

const router = express.Router();

//****게시글 등록 POST/post****
router.post("/", async (req, res, next) => {
  try {
    //일단 받아온 데이터를 이용해 post를 DB에 등록해줍니다.
    const post = await Post.create({
      content: req.body.content,
      UserId: req.user.id, //deserializeUser에서 가져온것(req.user로 접근)
    });
    res.status(201).json(user); //프론트로 응답주기
  } catch (e) {
    console.error(e);
    next(e);
  }
});
```

여기서 끝나지 않습니다. 

왜냐하면 우리는 프론트에서 필요한 Image와 User, Comment를 넣어주지 않았기 때문이죠 . 

### 필요한 정보들 추가해주기

```javascript
const express = require("express");
const { Post, Image, Comment, User } = require("../models");

const router = express.Router();

//게시글 등록 POST/post
router.post("/", async (req, res, next) => {
  try {
    //일단 받아온 데이터를 이용해 post를 DB에 등록해줍니다.
    const post = await Post.create({
      content: req.body.content,
      UserId: req.user.id, //deserializeUser에서 가져온것(req.user로 접근)
    });
    //****정보 추가****
    const fullPost = await Post.findOne({
      where: { id: post.id }, //위에서 방금 생성한 post의 아이디로 찾아서
      include: [
        { model: Image }, //image넣어주고
        { model: Comment }, //댓글들 넣어주고
        { model: User, attributes: ["id", "nickname"] }, //User에서 id,nickname만 넣고
      ],
    });
    res.status(201).json(fullPost); //프론트로 응답주기
  } catch (e) {
    console.error(e);
    next(e);
  }
});

module.exports = router;
```

자 게시글 등록이 끝났습니다!!

댓글도 같은 방법으로 구현해보아요

## 댓글 작성

단, 다른 점이 있습니다. 프론트서버에서 서버로 넘겨줄때 URI에 게시글아이디를 포함시켜서 전송을 해주는 상황입니다. 

`/2/comment` 이런 식으로 말이죠.. 

자 사실 우리는 데이터에 이미 게시글 아이디를 포함시켜서 받았기 때문에 저 URI에 있는 id는 필요가 없지만, 그래도 한번 사용해 보겠습니다. 

일단 먼저 서버에서는 철저한 검사를 해줘야 하기 때문에 개시글이 있는 개시글인지, 삭제가 되지는 않았는지 먼저 구현을 해보겠습니다. 

```javascript
router.post("/:id/comment", async (req, res, next) => {
  try {
    const post = await Post.findOne({
      where: { id: req.params.id }, //URI로 받은 정보는 req.params에 담겨있다.
    });
    if (!post) {
      //만약 받은 id와 일치하는 게시글이 없다면
      return res.send.status(403).send("존재하지 않는 개시글");
    }
  } catch (e) {
    console.error(e);
    next(e);
  }
});
```

다음은 이제 게시글이 존재한다면 댓글을 등록해줘야 겠죠. 게시글과 마찬가지로 db에 정보를 등록한뒤 

필요한 관계정보들을 추가해주는 방식으로 하겠습니다. 

```javascript
router.post("/:id/comment", async (req, res, next) => {
  try {
    const post = await Post.findOne({
      where: { id: req.params.id }, //URI로 받은 정보는 req.params에 담겨있다.
    });
    if (!post) {
      //만약 받은 id와 일치하는 게시글이 없다면
      return res.send.status(403).send("존재하지 않는 개시글");
    }
    //댓글 DB에 생성
    const comment = await Comment.create({
      content: req.body.content,
      PostId: parseInt(req.params.id, 10), //URIparams는 문자열이라 형변환해야함
      UserId: req.user.id,
    });
    //관계데이터 (작성자) 추가하기
    const fullComment = await Comment.findOne({
      where: { id: comment.id },
      include: [{ model: User, attributes: ["id", "nickname"] }],
    });
    res.status(201).json(fullComment);
  } catch (e) {
    console.error(e);
    next(e);
  }
});
```

 자 이렇게 댓글 구현까지 완료되었습니다!!

