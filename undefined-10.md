# 이미지 업로드

프론트가 어떻게 구성되어 있는지 먼저 살펴보겠습니다. 

이미지와 같은 미디어를 form에서 서버로 전송할 때에는 

Form 에 `encType="multipart/form-data"` 속성을 넣어서 multipart데이터로 전송을 해줍니다. 

```javascript
    <Form
      style={{ margin: "10px 0 20px" }}
      encType="multipart/form-data"
      onFinish={onSubmit}
      name="image"
    >
```

input 요소를 한번 살펴보아요.

```javascript
        <input
          type="file"
          mutiple
          hidden
          ref={imageInput}
          onChange={onChangeImages}
        />
```

이미지를 선택하게 되면, `onchange`이벤트를 통해 `onChangeImages`함수를 발동시켜 줄 것입니다. 

`onChangeImages`는 다음과 같습니다. 

```javascript
 const onChangeImages = useCallback((e) => {
    console.log("images", e.target.files); //e.target.files안에 선택한 이미지가 들어있다.
    const imageFormData = new FormData();
    //배열이 아니기 때문에 배열의 forEach를 빌려쓰는것
    [].forEach.call(e.target.files, (f) => {
      imageFormData.append("image", f);
    });
    dispatch({
      type: UPLOAD_IMAGES_REQUEST,
      data: imageFormData,
    });
  });
```

이렇게 되면 imageFormData안에 `{image: 이미지정보}` 이런식으로 데이터가 담길것입니다. 

이미지를 올리면 FormData저 형식으로 올라가게 되는데, 우리백앤드는 json과 url만 받을 수 있게끔 설정되어 있습니다.

먼저 multipart를 처리할 수 있는걸 설정해줘야 하는데,

multer를 통해 조금 더 쉽게 구현해보도록 하겠습니다. 

그전에 알아두어야 알 것이 Router별로 처리해야 하는 정보가 다를 수도 있기 때문에 우리는 app.js가 아니라 post.js에서만 multer를 사용하겠습니다. 

먼저 multer를 깔아준뒤 기본적인 설정들을 해줍니다. 

* uploads폴더에 사진들을 저장할 것이다. 
* fs를 통해 fileSystem에 접근해 없으면 폴더를 생성
* 일단 지금은 드라이브에 저장하지만 후에 서버로 변경해줄 것.
* 파일명이 중복되면 덮어써지기 때문에 시간을 추가해줄 것

```javascript
const multer = require("multer");
const path = require("path"); //노드 기본 제공
const fs = require("fs"); //파일시스템 조작 (기본제공)

try {
  fs.accessSync("uploads"); //업로드 폴더가 있는지 검사를 하고,
} catch (e) {
  //없으면
  console.log("폴더가 없으므로 생성합니다.");
  fs.mkdirSync("uploads");
}

const upload = multer({
  //어디에 저장
  storage: multer.diskStorage({
    destination(req, res, done) {
      //저장위치
      done(null, "uploads");
    },
    filename(req, file, done) {
      // 같은 이름을 가지게 되면 덮어써지기 때문에 시간을 넣어준다.
      const ext = path.extname(file.originalname); //확장자 추출(.png)
      const basename = path.basename(file.originalname, ext); //파일명만 가져올 수 있다.(conrad)
      done(null, basename + new Date().getTime() + ext); //conrad15184812928.png
    },
  }),
  limits: { fileSize: 20 * 1024 * 1024 }, //20MB로 제한
});
```

컴퓨터로 저장하게 되면 생기게 되는 단점들이 몇가지 있습니다.

* 나중에 서버를 옮길때마다 데이터를 복사하기 때문에 서버용량을 잡아먹는다. 
* 그건 다 비용이다. 
* 왠만하면 백앤드 서버를 거치게 되면 자원을 먹기 때문에 프론트에서 바로 올려줘도 된다.

이번에는 data가 오게되면 처리해줄 라우터를 만들어 보겠습니다 . 

그전에 알아두어야 할 사실이 있습니다. 

왜 업로드만 먼저 진행하는 걸까요?

이미지나 동영상을 업로드 하는 방식은 몇가지 있는데, 

한방에 보내는 방식, 두번 보내는 방식이 있습니다. 

### 한방에 보내는 방식

```text
{
    content:'안녕',
    image:010101000101111011011011
}
```

* 이미지를 올리는데 시간도 많이 걸리고, 
* 업로드가 끝나야만 후처리\(머신러닝\) 등이 가능하다. 

### 두번에 보내는 방식

첫번째 보낼때는 이미지만 먼저 데이터를 서버에 업로드 해놓고,

```text
{
    image:010101000101111011011011
}
```

그리고 서버는 파일명을 리턴해주는데, 

프론트에서 파일명을 알고 있기때문에 리사이징이나 미리보기등을 사람이 컨텐츠 작성하는 동안 진행 할 수 있습니다. 

단점도 있긴 한데, 

* 이미지 올리고 마음이 바뀌어서 게시글을 취소하면, image가 서버에 남겨진다. 
* 프론트, 백앤드에서 구현하기가 까다롭다. 
* 그러나 이미지또한 자산이 되기때문에 서버에 남겨져도 손해는 아니다. 

이런 특징들을 가지고 있습니다. 

두번째 방식이기에 이렇게 진행하겠죠?

```javascript
router.post(
  "/images",
  isLoggedIn,
  upload.array("image"),
  async (req, res, next) => {
    console.log(req.files); //업로드된 파일들req.files에 담김
    res.json(req.files.map((v) => v.filename));
  }
);
```

이제 서버에 업로드 되는 과정은 모두 끝났습니다. 

이제 그 경로를 이용해 미리보기를 구현해보겠습니다. 

