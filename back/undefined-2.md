# 요청받기 전 셋팅

이번에는 완성된 DB를 통해 회원가입을 구현해볼 것입니다. 

`POST/user/` uri를 통해 요청을 보낼수 있도록 front부분을 수정해주세요 

단, 몇가지 문제가 있습니다 . 

* front와 back 서버의 포트가 다르기 때문에 CORS문제가 발생함. 
* request에서 오는 데이터가 json형식이기 때문에 해석이 필요함. 

app.js에서 몇가지 설정을 해줘야 합니다. 

Body-Parser를 사용하지 않고 express에 내장된 Body-Parser와 동일한 매소드를 사용할 것입니다. 

CORS에러 해결은 웹브라우저에서 바로 요청을 보내는 것이 아닌, Front서버를 통해 요청을 보내는 Proxy를 통해 프론트단에서 해결할 수도 있지만, 백앤드 서버에서 해당 포트를 해결하는 방식으로 해결해보겠습니다. 

## cors

cors패키지를 다운받아 주세요

```text
npm i cors
```

방법은 간단합니다 .

router별로 설정해주는 방법도 있지만, 우리는 app.js에서 한번에 설정해줄 것입니다 .

다음과 같이 해주세요

```javascript
const express = require("express");
const userRouter = require("./routes/user");
const dotenv = require("dotenv");
const db = require("./models");
//cors를 불러온뒤
const cors = require("cors");

dotenv.config();
const app = express();
const PORT = 3030;

//db싱크
db.sequelize
  .sync()
  .then(() => console.log("연결성공"))
  .catch(console.error("에러발생"));

app.get("/", (req, res) => {
  res.send("페이지");
});

//미들웨어
//cors설정
app.use(cors({ origin: "*" }));

//Router
app.use("/user", userRouter);

app.listen(PORT, () => {
  console.log(PORT, "에서 서버실행중");
});
```

app.use 를 사용하면 express에서 미들웨어를 사용할 수 있는데, 이를 사용해 

cors를 사용해보면, `cors({origin:" 허용할 호스트 "})`

이런 방법으로 사용할 수 있습니다. \* 을 입력하면 모든 요청에 대해 허용을 의미합니다. 

## Parser

아주 쉽습니다.  다양한 방법들과 설정이 존재하지만, 대부분의 경우 이렇게 사용할 것입니다. 

```javascript
const express = require("express");
const userRouter = require("./routes/user");
const dotenv = require("dotenv");
const db = require("./models");
//cors를 불러온뒤
const cors = require("cors");

dotenv.config();
const app = express();
const PORT = 3030;

//db싱크
db.sequelize
  .sync()
  .then(() => console.log("연결성공"))
  .catch(console.error("에러발생"));

app.get("/", (req, res) => {
  res.send("페이지");
});

//미들웨어
app.use(express.json()); // json형식받기
app.use(express.urlencoded({ extended: true })); //form-submit을 했을때
app.use(cors({ origin: "*" }));

//Router
app.use("/user", userRouter);

app.listen(PORT, () => {
  console.log(PORT, "에서 서버실행중");
});
```

자 이제 셋팅이 끝났으니, 회원가입을 완성해볼까요

