# 이미지 & 텍스트 업로드 완성

먼저 upload시에 이미지로 인해 전송되는 data의 형식이 multipart/formData로 변경되었기 때문에 

이전에 구현한 게시글 업로드를 수정해주어야 합니다. 

PostForm.js에 가서 submit함수를 수정해볼까요

```javascript
  const onSubmit = () => {
    //게시글 없으면 경고
    if (!text || !text.trim()) {
      return alert("게시글을 작성하세요");
    }
    const formData = new FormData();
    imagePath.forEach((q) => {
      formData.append("image", q); //키가 image 백에서 req.body.image
    });
    formData.append("content", text); //키가 content 백에서 req.body.content
    //이미지가 없으면 formData를 쓸 필요가 없지만, multer nono을 써보기 위해
    dispatch({ type: ADD_POST_REQUEST, data: formData });
  };
  
  //data:{image:~~~~, content:~~~~~};
```

이런식이라고 봐도 무방합니다. 

백앤드 라우터에서는 아까 만든 upload에 none\(\)이라는 매소드를 사용할 것입니다. 

전에 작성한 이미지 업로드용 라우터는 남겨두고 게시글 작성 라우터를 수정하는 것입니다!

```javascript
//게시글 등록 POST /post
router.post("/", isLoggedIn, upload.none(), async (req, res, next) => {
  try {
    const post = await Post.create({
      content: req.body.content,
      UserId: req.user.id,
    });
    //image는 front에서append로 정해준 키
    if (req.body.image) {
      if (Array.isArray(req.body.image)) {
        //이미지를 여러개 올리면 image:[1.png,2.png]
        const images = await Promise.all(
          //한번에 모두 완료되면 올려준다.
          req.body.image.map((image) => Image.create({ src: image })) //Image 모델에 넣어준다.
        );
        await post.addImages(images); //Sequelize매소드로 images를 넣어준다.
      } else {
        //이미지를 하나만 올리면 image:1.png
        const image = await Image.create({ src: req.body.image });
        await post.addImages(image);
      }
    }
    //DB에는 IMAGE를 올리지 않고 서버에 가지고 있다가 DB에는 경로만 저장하는게 좋음 (DB는 캐싱 불가)
    //Image 추가해주기
    const fullPost = await Post.findOne({
      where: { id: post.id },
      include: [
        { model: Image },
        { model: Comment },
        { model: User, attributes: ["id", "nickname"] }, //작성자
        { model: User, as: "Likers", attributes: ["id"] }, //좋아요 누른사람
      ],
    });
    res.status(201).json(fullPost); //생성되었다고 프론트로
  } catch (e) {
    console.error(e);
    next(e);
  }
});
```

사람들이 가끔 DB에 이미지를 올리지 왜 서버에 올릴까? 라고 생각하는 사람들도 있는데, 

이는 DB에 이미지를 업로드 하게되면 DB처리속도가 매우 느려지게 되며, 

이미지와 같은 파일들은 캐싱이 가능한데 , DB에 업로드하게 된다면 캐싱이 불가능해집니다. 

그런 이유로 DB에는 경로만 적어주고 , 경로를 보내주는 것입니다. 

