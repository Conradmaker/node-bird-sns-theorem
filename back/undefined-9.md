# 좋아요 기능 추가

좋아요 기능에는 먼저 알아두어야 할 것이 있습니다. 

## Sequelize메소드

우리는 Sequelize에서 모델을 생성할 때 관계를 설정해둔 모델이 있었습니다. 

Sequelize에서 관계를 설정하면 자동으로 add, get, remove같은 메소드들이 생성되는데 다음과 같습니다. 

```javascript
  Post.associate = (db) => {
    //belongsTo는 UserId칼럼을 만들어줘서 참조관계를 만들어준다.
    db.Post.belongsTo(db.User); //post.addUser,post.getUser, post.setUser
    db.Post.belongsToMany(db.Hashtag, { through: "PostHashtag" }); //post.addHashtags
    db.Post.hasMany(db.Comment); //post.addComments,post.getComments
    db.Post.hasMany(db.Image); //post.addImages,post.getImages
    db.Post.belongsToMany(db.User, { through: "Like", as: "Likers" }); //다대다 관계 (Like 이름설정,별칭: 좋아요누른사람)
    db.Post.belongsTo(db.Post, { as: "Retweet" }); //addRetweet
  };
```

## 좋아요 만들기

그리고 Router를 작성해볼까요?

프론트서버에서 parameter로 게시글의 id를 가져와서 그 게시글의 좋아요에 사용자의 ID를 추가하는 방식으로 해보겠습니다. 

과정은 다음과 같습니다.

* 먼저 URI parameter로 받은 게시글 id를 이용해 존재하는 게시글 여부 확인
* 없는 글이면 에러전송
* 있다면 Sequelize에서 생성해준 메소드를 이용해 로그인된 사용자 id 추가
* 리덕스 적용을 위해 프론트로 필요데이터를 날려준다. 

URI는 `PATCH /post/:id/like` 로 받아오겠습니다.

```javascript
router.patch("/:id/like", isLoggedIn, async (req, res, next) => {
  try {
    //좋아려 하려고 하는 게시글 조회
    const post = await Post.findOne({ where: { id: req.params.id } });
    //해당 게시글이 없다면 에러발생
    if (!post) {
      return res.status(403).send("게시글이 없어요!");
    }
    //게시글Liker에 로그인되있는 사용자 아이디 추가
    await post.addLikers(req.user.id);
    // 프론트 리덕스 적용을 위해 포스트아이디와 유저아이디 보내줌
    res.status(201).json({ PostId: post.id, UserId: req.user.id });
  } catch (e) {
    console.error(e);
    next(e);
  }
});
```

## 좋아요 취소 만들기

좋아요 기능을 구현했으니 취소기능도 구현해봐야겠죠?

방식은 좋아요때와  같습니다. 

```javascript
router.delete("/:id/unlike", isLoggedIn, (req, res, next) => {
  try {
    //취소할 게시글 조회
    const post = await Post.findOne({ where: { id: req.params.id } });
    if(!post){
      return res.status(403).send('게시글이 없어요!');
    }
    //게시글 Likers에 있는 내 id 제거
    await post.removeLikers(req.user.id);
    // 프론트 리덕스 적용을 위해 포스트아이디와 유저아이디 보내줌
    res.status(201).json({ PostId: post.id, UserId: req.user.id });
  } catch (e) {
    console.error(e);
    next(e);
  }
});
```

쉽죠???

