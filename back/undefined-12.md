# 해쉬테그

전에 컴포넌트를 구성할때 정규표현식으로 해쉬태그를 배열로 추출했습니다. 

이번에도 정규표현식을 이용해줄 것입니다 . 

저번시간에 작성한 게시글 작성 라우터를 다시 수정해주어야 겠네요. 

### 해쉬태그 추출

먼저 보내진 글에서 해쉬태그를 정규표현식을 이용해 추출해줍니다. 

```javascript
//게시글 등록 POST /post
router.post("/", isLoggedIn, upload.none(), async (req, res, next) => {
  try {
    //해쉬태그 추출
    const hashtags = req.body.content.match(/#[^\s#]+/g);
    
    const post = await Post.create({
      content: req.body.content,
      UserId: req.user.id,
    });
    if (req.body.image) {
      if (Array.isArray(req.body.image)) {
        const images = await Promise.all(
          req.body.image.map((image) => Image.create({ src: image }))
        );
        await post.addImages(images);
      } else {
        const image = await Image.create({ src: req.body.image });
        await post.addImages(image);
      }
    }
    const fullPost = await Post.findOne({
      where: { id: post.id },
      include: [
        { model: Image },
        { model: Comment },
        { model: User, attributes: ["id", "nickname"] }, 
        { model: User, as: "Likers", attributes: ["id"] }, 
      ],
    });
    res.status(201).json(fullPost);
  } catch (e) {
    console.error(e);
    next(e);
  }
});
```

### 해쉬태그가 있다면 해쉬태그를 hashtags태이블에 저장

그걸 구성하는 코드는 다음과 같습니다. 

```javascript
if (hashtags) {
      const result = await Promise.all(
        hashtags.map((tag) =>
          Hashtag.findOrCreate({
            //있으면 등록 없으면 등록 X
            where: { name: tag.slice(1).toLowerCase() }, //slice로 #때준다.
          })
        )
      ); // [[노드, true], [리액트, true]]
      await post.addHashtags(result.map((v) => v[0]));
    }
```

findOrCreate는 태이블안에 이름이 같은 해쉬태그가 있다면 저장하지 않기 위해 사용하는 Sequelize매소드입니다. 

### 게시글 업로드 완성코드

```javascript
//게시글 등록 POST /post
router.post("/", isLoggedIn, upload.none(), async (req, res, next) => {
  try {
    //해쉬태그 추출
    const hashtags = req.body.content.match(/#[^\s#]+/g);
    
    const post = await Post.create({
      content: req.body.content,
      UserId: req.user.id,
    });
    //해쉬태그가 있다면
    if (hashtags) {
      const result = await Promise.all(
        hashtags.map((tag) =>
          Hashtag.findOrCreate({
            //있으면 등록 없으면 등록 X
            where: { name: tag.slice(1).toLowerCase() }, //slice로 #때준다.
          })
        )
      ); // [[노드, true], [리액트, true]]
      await post.addHashtags(result.map((v) => v[0]));
    }
    
    if (req.body.image) {
      if (Array.isArray(req.body.image)) {
        const images = await Promise.all(
          req.body.image.map((image) => Image.create({ src: image }))
        );
        await post.addImages(images);
      } else {
        const image = await Image.create({ src: req.body.image });
        await post.addImages(image);
      }
    }
    const fullPost = await Post.findOne({
      where: { id: post.id },
      include: [
        { model: Image },
        { model: Comment },
        { model: User, attributes: ["id", "nickname"] }, 
        { model: User, as: "Likers", attributes: ["id"] }, 
      ],
    });
    res.status(201).json(fullPost);
  } catch (e) {
    console.error(e);
    next(e);
  }
});
```



