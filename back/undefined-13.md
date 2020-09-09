# 리트윗

```javascript
//리트윗
router.post("/:id/retweet", isLoggedIn, async (req, res, next) => {
  //:id는 parameter
  try {
    //백앤드에서 철저히 검사를 해줘야 한다. (개시글이 있는 개시글인지)
    const post = await Post.findOne({
      where: { id: req.params.id },
      include: [{ model: Post, as: "Retweet" }],
    });
    if (!post) {
      return res.send.status(403).send("존재하지 않는 개시글입니다");
    }
    if (
      req.user.id === post.UserId || //자기 게시글 리트윗 막기
      (post.Retweet && post.Retweet.UserId === req.user.id) //자기 게시글 리트윗한 게시글 리트위 막기
    ) {
      return res.status(403).send("자기 글을 리트윗 안되요!");
    }
    //그러나 다른 사람 게시글을 리트윗한 게시글을 리트윗하면 리트윗하면 원본게시글 리트윗됨
    const retweetTargetId = post.RetweetId || post.id;
    const exPost = await Post.findOne({
      where: {
        UserId: req.user.id,
        RetweetId: retweetTargetId,
      },
    });
    if (exPost) {
      return res.status(403).send("이미 리트윗 함");
    }
    const retweet = await Post.create({
      UserId: req.user.id,
      RetweetId: retweetTargetId,
      content: "retweet", //원래는 null이여야 하지만, model에서 null을 허용안해서
    });
    const retweetWithPrevPost = await Post.findOne({
      where: { id: retweet.id },
      include: [
        {
          model: Post,
          as: "Retweet",
          include: [
            { model: User, attributes: ["id", "nickname"] },
            { model: Image },
          ],
        },
        { model: User, attributes: ["id", "nickname"] },
        {
          model: User, // 좋아요 누른 사람
          as: "Likers",
          attributes: ["id"],
        },
        { model: Image },
        {
          model: Comment,
          include: [{ model: User, attributes: ["id", "nickname"] }],
        },
      ],
    });
    //include가 너무 많아지면 DB읽는 속도가 느려져서 분리해줘야 한다. (Comment같이)
    res.status(201).json(retweetWithPrevPost); //생성되었다고 프론트로
  } catch (e) {
    console.error(e);
    next(e);
  }
});
```

