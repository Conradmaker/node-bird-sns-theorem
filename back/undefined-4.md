# 로그아웃구현

로그아웃을 구현하는 작업은 상당히 간단합니다. 

* passport를 통해 로그인 했기 때문에 logout매소드를 호출하고
* 서버에 있는 세션을 삭제하면

로그아웃이 완료됩니다. 

먼저 라우터를 만들어 볼까요?

```javascript
//routes/user.js

router.post('/logout',(req,res)=>{
    req.logout();
    req.session.destroy();
    res.send('로그아웃 성공')
})
```

완료되었습니다. 

추가적으로 로그아웃은 로그인된사용자만,

로그인과 회원가입은 로그인되지 않은 사용자만 사용할 수 있도록 

로그인 유무를 확인하는 커스텀 미들웨어를 만들어보겠습니다. 

routes폴더에 middlewares.js를 만들고 다음과 같이 작성합니다. 

```javascript
//로그인 되어있는 경우
exports.isLoggedIn = (req, res, next) => {
  if (req.isAuthenticated()) {
    //passport에서 제공하는 로그인 유무 확인
    next(); //다음 미들웨어로
  } else {
    res.status(401).send("로그인이 필요합니다.");
  }
};

exports.isNotLoggedIn = (req, res, next) => {
  if (!req.isAuthenticated()) {
    next();
  } else {
    res.status(401).send("로그인하지 않은 사용자만 접근이 가능합니다.");
  }
};
```

isAuthenticated\(\)는 passport에서 제공하는 로그인 유무를 확인하는 메소드입니다. 

next\(\)를 실행하면 다음 미들웨어로 넘어가지만, \(\)안에 뭔가가 있다면, 다음 에러처리 미들웨어로 넘어갈 수 있습니다. 

### 에러처리 미들웨어

에러처리 미들웨어로 간다는것은 다음 미들웨어로 가지 않고, app.js router다 거치고 마지막에 있는 미들웨어로 가는 것인데 형식은 다음과 같습니다.

```javascript
app.use((err,req,res,next)=>{
})
```

여기서는 에러페이지를 커스텀 할수 있지만, 기본적으로 express에는 에러처리 미들웨어가 내장되어 있습니다.

마지막으로 위에서 작성한 미들웨어를 적용시켜 주세요

```javascript
const {isLoggedIn,isNotLoggedIn} =require('./middlewares')

router.post('/logout',isLoggedIn,(req,res)=>{
    req.logout();
    req.session.destroy();
    res.send('로그아웃 성공')
})
```

사실 지금까지 작성한게 맞기는 하고, 동작도 하지만, 로그인 했는데 안했다고 나오고, 그런 경우가 있을 것입니다. 이런건 다다음 시간 쿠키공유에서 해결해보겠습니다.

