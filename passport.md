# Passport를 이용한 로그인 구현

로그인 구현은 너무너무너무너무 힘든 일입니다. 

메인기능들의 시작점이기도 하며, 정보가 오고가기때문에 보안도 중요하기 때문이죠.

그런 어려운 로그인을 그나마! 쉽게?! 만들어주는 passport를 통해 로그인을 구현해보도록 해요. 

> 그래도 복잡하긴 복잡합니다..

## 라우터 셋팅

일단 `POST/user/login` URI를 통해 router로 받아줄 것입니다. 

```javascript
router.post('/login',(req,res,next)=>{
    //일단 이렇게만 만들어 놓으세요
})
```

## passport

passport와 passport-local을 설치해주세요

그뒤 root디렉토리에 passport폴더를 만든뒤, index.js ,local.js를 만들어주세요 

* index.js -  passport설정
* local.js - 로그인 전략 설정

### index.js

세션에 모든 정보를 들고있기에는 무겁기 때문에 id만 가지고 있다가 db에 요청해서 정보를 가져와야 하는데 지금은 일단 틀만을 만들어 줄 것입니다.

passport안에서는 done\(\)을 next\(\)와 같이 사용한다는 것 알아두세요

```javascript
const passport = require("passport");
//로그인 전략 가져오기
const local = require("./local");
const { User } = require("../models");

module.exports = () => {
  //id를 가지고 있음
  passport.serializeUser(() => {});
  //후에 프론트에서 쿠키를 받으면 쿠키검사후 id가 들어가는 부분
  passport.deserializeUser(() => {});
  
  local(); //전략실행
};

```

### local.js

!!주석을 잘 읽어주세요!!

```javascript
//passport-local중 Strategy를 LocalStrategy로 불러온다.
const { Strategy: LocalStrategy } = require("passport-local");
const passport = require("passport");
const { User } = require("../models");
const bcrypt = require("bcrypt");

module.exports = () => {
  passport.use(
    new LocalStrategy(
      // 'email','password'는 req.body안에 들어있는 것
      { usernameField: "email", passwordField: "password" },
      async (email, password, done) => {
        //전략
        try {
          //입력받은 email이 DB에 있는지 찾는다.
          const user = await User.findOne({
            where: { email },
          });
          //ID가 없을경우
          if (!user) {
            //       서버에러, 성공여부,          클라이언트측에러
            return done(null, false, { reason: "존재하지 않는 아이디입니다." });
          }
          //id가 있다면 입력한 비밀번호를 DB의 해쉬 비번과 비교한다.
          const result = await bcrypt.compare(password, user.password); //(입력된값, DB에서 찾은 user비번)
          //만약 같다면  result = true가 나옴
          if (result) {
            return done(null, user); //user를 넘겨준다.
          }
          //일치하지 않는다면
          return done(null, false, { reason: "비밀번호 틀림" });
        } catch (e) {
          //에러처리
          console.error(e);
          return done(e);
        }
      }
    )
  );
};

```

## 쿠키 & 세션

쿠키와 세션을 관리해주기 위해 cookie-parser와, express-session을 설치해 주세요

```javascript
npm i cookie-parser express-session
```

그리고 app.js에서 셋팅을 해줍니다. \(passport도 함께 불러와요\)

```javascript
const express = require("express");
const userRouter = require("./routes/user");
const dotenv = require("dotenv");
const db = require("./models");
const cors = require("cors");
//세션, cookie-parser불러오고,
const session = require("express-session");
const cookieParser = require("cookie-parser");
//passport, passport설정 불러오기
const passport = require("passport");
const passportConfig = require("./passport");

const app = express();
const PORT = 3030;

//db싱크
db.sequelize
  .sync()
  .then(() => console.log("연결성공"))
  .catch(console.error("에러발생"));

dotenv.config();
passportConfig(); //passport 실행

//쿠키, 세션, 로그인
app.use(cookieParser(process.env.COOKIE_SECRET)); //해석에 사용할 비밀키
app.use(
  session({
    saveUninitialized: false,
    resave,
    secret: process.env.COOKIE_SECRET, //비밀키
  })
);
app.use(passport.initialize());
app.use(passport.session());



//미들웨어
app.use(express.json()); // json형식받기
app.use(express.urlencoded({ extended: true })); //form-submit을 했을때
app.use(cors({ origin: "*" }));

//Router
app.get("/", (req, res) => {
  res.send("페이지");
});
app.use("/user", userRouter);

app.listen(PORT, () => {
  console.log(PORT, "에서 서버실행중");
});

```

process.env.COOKIE\_SECRET은 비밀키이기 때문에 .env에서 가져온 것입니다. 

.env도 설정해주세요

```javascript
COOKIE_SECRET=비밀키
DB_PASSWORD=디비비밀번호
```

이제 passport/index.js만 설정해주면 됩니다. 

```javascript
const passport = require("passport");
const local = require("./local");
const { User } = require("../models");

//passport설정
//세션에 모든 정보를 들고있기에는 무겁기 때문에 id만 가지고 있다가 db에 요청해서 정보
module.exports = () => {
  passport.serializeUser((user, done) => {
    done(null, user.id);
  });
  passport.deserializeUser(async (id, done) => {
    try {
      const user = await User.findOne({ where: { id } });
      done(null, user);
    } catch (e) {
      console.error(e);
      done(e);
    }
  });

  local();
};
```

* 프론트에서 서버로는 cookie만 보냅니다. \(이상한 문자로\)
* 서버가 쿠키파서,익스프레스 세션으로 쿠키 검사후에 id:1을 발견하면
* id:1이 deserializeUser에 들어가서
* req.user로 사용자 정보가 들어가게 됩니다. 
* 즉 요청 보낼때마다 deserizlizeUser가 실행\(db요청1번씩 실행\)
* 실무에서는 desezializeUser결과물을 캐싱합니다. 

아 마지막으로 router도 설정을 해주어야 겠네요

### router에서 정보 전달

미들웨어안에 미들웨어가 들어가기 때문에 미들웨어확장을해주어야 합니다. 

```javascript
const express = require("express");
const { User } = require("../models");
const bcrypt = require('bcrypt');
const passport = require("passport");

const router = express.Router();


//로그인
router.post('/login',(req,res,next)=>{
    passport.authenticate('local',(err,user,info)=>{ //done으로 보낸것
        //에러가 있어서 보내졌다면
        if(err){
            console.error(err)
            return next(err);
        }
        //local.js에서 이메일이나비번이 없어 reason을 적어줬다면,
        if(info){
            return res.status(401).send(info.reason)
        }
        //둘다 없다면 (성공) 'user'는 local에서 받은 user
        return req.login(user, async(loginErr)=>{
            //먼저 에러처리
            if(loginErr){
                console.error(loginErr);
                return next(loginErr);
            }
            return res.status(200).json(user)//user를 json으로 보내준다. 
        })
    })
})

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
    res.status(201).send('회원가입성공'); 
  } catch (e) {
      console.error(e);
      next(e);
  }
});

module.exports = router;

```

자 지금까지 하면 로그인은 구현이 되지만, User모델에 있는 정보 즉, 

email,nickname,password가 모두 보내지게 됩니다. 

필요한 정보는 추가해주고, password를 없애보도록 할까요?



