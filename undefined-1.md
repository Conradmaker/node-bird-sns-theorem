# 회원가입

## 회원가입

### 라우저 설정

우선 `POST /user/`   을 통해 요청을 받기 때문에 user.js라우터에서 요청을 처리해 줄것입니다. 

```javascript
const express = require("express");

const router = express.Router();

// 'POST /user/' 를 통해 요청을 받을 경우 라우터
router.post("/", (req, res, next) => { //next는 미들웨어 작업
  
});

module.exports = router;
```

이제 저 안에다가 작성을 해볼까요? 

먼저 비동기 작업이기 떄문에 무조건 에러처리를 해줘야 합니다 . 

### 순서

try-catch문을 사용한뒤 , 순서는 다음과 같습니다. 

1. 요청받은 email 과 DB에 저장된email이 중복되는지 체크
2.  중복값이 없다면 에러코드와 같이 메세지를 날려준다.
3. 중복값이 없다면, bcrypt를 통해 받은 비밀번호를 암호화\(해쉬화\)
4. 비동기 작업으로 User모델에 담아준다.
5. 작업이 완료되면, 201상태와 함께 성공메세지를 보내고,
6. 작업중간 에러가 발생하면 catch문에서 error처리

이제 시작해 볼까요? 

### 1.요청받은 email 과 DB에 저장된email이 중복되는지 체크

```javascript
const express = require("express");
//User모델을 불러온뒤
const { User } = require("../models");

const router = express.Router();

// 'POST /user/' 를 통해 요청을 받을 경우 라우터
router.post("/", (req, res) => {
  try {
    //1.중복체크
    const exUser = await User.findOne({
        where:{//where은 조건
            email:req.body.email, //같은 email이 있는지
        }
    })
  } catch (e) {
      
  }
});

module.exports = router;
```

### 2.중복값이 없다면 에러코드와 같이 메세지를 날려준다.

위에서 중복값이 있다면 exUser가 true가 되었겠죠?

```javascript
const express = require("express");
const { User } = require("../models");

const router = express.Router();

router.post("/", (req, res) => {
  try {
    const exUser = await User.findOne({
        where:{
            email:req.body.email,
        }
    })
    //2.중복된 아이디가 있다면 에러보내주기
    if(exUser){
        res.status(403).send("이미 사용중인 이메일입니다.")
    }
  } catch (e) {
      
  }
});

module.exports = router;

```

### 3.중복값이 없다면, bcrypt를 통해 받은 비밀번호를 암호화\(해쉬화\)

bcrypt를 설치해주세요

```javascript
const express = require("express");
const { User } = require("../models");
const bcrypt = require('bcrypt')

const router = express.Router();

router.post("/", (req, res) => {
  try {
    const exUser = await User.findOne({
        where:{
            email:req.body.email,
        }
    })
    if(exUser){
        res.status(403).send("이미 사용중인 이메일입니다.")
    }
    //2.중복된 아이디가 없다면 받은 비밀번호 문자열 해쉬화
    const hashedPwd = await bcrypt.hash(req.body.password, 11)
  } catch (e) {
      
  }
});

module.exports = router;
```

bcrypt.hash\( \)의 첫번째 인자는 변경할 비밀번호, 두번째 인자는 얼마나 강하게 할것인지를 말하는데,

두번째 인자에는 보통 10~13사이의 값을 담아줍니다. 서버 컴퓨터의 성능에 따라 값이 높아지면 처리시간이 길어질 수 있습니다. 

### 4.비동기 작업으로 User모델에 담아준다.

```javascript
const express = require("express");
const { User } = require("../models");
const bcrypt = require('bcrypt')

const router = express.Router();

router.post("/", (req, res) => {
  try {
    const exUser = await User.findOne({
        where:{
            email:req.body.email,
        },
    });
    if(exUser){
        res.status(403).send("이미 사용중인 이메일입니다.")
    }
    const hashedPwd = await bcrypt.hash(req.body.password, 11);
    //3.User모델에 담아주기
    await User.create({
        email:req.body.email,
        nickname: req.body.nickname,
        password: hashedPwd
    });
  } catch (e) {
      
  }
});

module.exports = router;

```

### 

### 5.작업이 완료되면, 201상태와 함께 성공메세지를 보내고,

### 6.작업중간 에러가 발생하면 catch문에서 error처리

```javascript
const express = require("express");
const { User } = require("../models");
const bcrypt = require('bcrypt')

const router = express.Router();

router.post("/", (req, res) => {
  try {
    const exUser = await User.findOne({
        where:{
            email:req.body.email,
        }
    })
    if(exUser){
        res.status(403).send("이미 사용중인 이메일입니다.")
    }
    const hashedPwd = await bcrypt.hash(req.body.password, 11);
    await User.create({
        email:req.body.email,
        nickname: req.body.nickname,
        password: hashedPwd
    });
    res.status(201).send('회원가입성공'); //201 - 잘생성됨
  } catch (e) {
      console.error(e);
      next(e);  //next를 통해 다음 미들웨어로 보내준다.
  }
});

module.exports = router;
```

생각보다 간단하죠? 

지금까지 회원가입 구현이었습니다.

### 

### 

### 



