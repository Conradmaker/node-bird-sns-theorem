# 서버만들기

이 부분은 아주 기본적인 node.js를 알고있는 전제 하입니다.

먼저 express.js와 nodemon을 설치해주세요 

```text
npm i express nodemon
```

app.js를 만들어볼까요 

```javascript
const express = require("express");

const app = express();
const PORT = 3030;

app.get("/", (req, res) => {
  res.send("페이지");
});

app.listen(PORT, () => {
  console.log(PORT, "에서 서버실행중");
});
```



