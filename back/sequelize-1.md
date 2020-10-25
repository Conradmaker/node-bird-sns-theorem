# Sequelize 적용

## 셋팅

models 폴더안에 index.js파일을 봐주세요 

```javascript
const Sequelize = require("sequelize");


Object.keys(db).forEach(modelName => {
  if (db[modelName].associate) {
    db[modelName].associate(db);
  }
});

db.sequelize = sequelize;
db.Sequelize = Sequelize;

module.exports = db;
```

이부분만 남기고 싹다 날려버립니다. 

먼저 DB와 연결을 해줘야 겠죠? 

```javascript
const Sequelize = require("sequelize");
//환경변수
const env = process.env.NODE_ENV || "development";
//env는 현재 개발모드를 나타는중
//DB계정정보 가져오기
const config = require("../config/config")[env];
const db = {}; //일단 db 빈객체를 만들어줍니다

//연결
//config 파일안에 있는 정보들을 이용해 DB와 연결해준다.
const sequelize = new Sequelize(
  config.database,
  config.username,
  config.password,
  config
);

Object.keys(db).forEach((modelName) => {
  if (db[modelName].associate) {
    db[modelName].associate(db);
  }
});

db.sequelize = sequelize;
db.Sequelize = Sequelize;

module.exports = db;
```

## 

## 모델작성

자 이제 모델 \( = 태이블\) 을 만들어 줄 것입니다.  

테이블 구조는 다음과 같습니다

![](../.gitbook/assets/image%20%289%29.png)

봐서는 이해가 어려우니 직접 만들면서 보아요

일단 우리는 model 5개를 만들어줘야 합니다. 위에 ERD에서는 8개인데 왜 5개만 만드냐 하면, 

관계 설정시 다대다관계가 나오면 태이블이 하나가 만들어져야만 합니다. 

일단 볼까요

![](../.gitbook/assets/image%20%288%29.png)

### User모델

```javascript
module.exports = (sequelize, DataTypes) => {
  const User = sequelize.define(
    //대문자 단수가 테이블로 만들어지면, 소문자 복수로 변경된다.
    "User",
    {
      //데이터 타입을 지정
      //                  데이터 타입             Null허용여부  ,   고유값
      email: { type: DataTypes.STRING(30), allowNull: false, unique: true },
      nickname: { type: DataTypes.STRING(30), allowNull: false },
      password: { type: DataTypes.STRING(100), allowNull: false }, //암호화하면 길이 늘어나서 넉넉히
    },
    {
      charset: "utf8",
      collate: "utf8_general_ci", //한글저장
    }
  );

  //이부분은 관계를 설정하는 부분
  User.associate = (db) => {};
  return User;
};
```

아직 관계는 설정하지 않았지만, email, nickname, password의 칼럼을 가지는 태이블을하나 생성한 것입니다. 

나머지도 만들어주세요 . 

관계설정을 해볼까요 일단 User에서는 다음과 같습니다. 

* 한명의 유저는 게시글을 많이 가지고 있을 수 있다. \(1대 다\)
* 한명의 유저는 댓글을 많이 작성할 수 있다. \(1대 다\)
* 유저는 게시글에 좋아요를 많이 누를수도 받을수도 있다. \(다대다\)
* 유저는 유저에게 팔로우를 많이 할수도 당할수도 있다. \(다대다\)
* 다른 유저는 유저에게 팔로우를 많이 할수/당할수 있다\(다대다 4번과 연계\)

```javascript
module.exports = (sequelize, DataTypes) => {
  const User = sequelize.define(
    //대문자 단수가 테이블로 만들어지면, 소문자 복수로 변경된다.
    "User",
    {
      //데이터 타입을 지정
      //                  데이터 타입             Null허용여부  ,   고유값
      email: { type: DataTypes.STRING(30), allowNull: false, unique: true },
      nickname: { type: DataTypes.STRING(30), allowNull: false },
      password: { type: DataTypes.STRING(100), allowNull: false }, //암호화하면 길이 늘어나서 넉넉히
    },
    {
      charset: "utf8",
      collate: "utf8_general_ci", //한글저장
    }
  );

  //이부분은 관계를 설정하는 부분
  User.associate = (db) => {
      //User가 Post를 많이 가질 수 있다.
    db.User.hasMany(db.Post);
    //User는 Comment를 많이 가질 수 있다. 
    db.User.hasMany(db.Comment)
    //다대다 관계는 관계태이블이 생성된다. 
    //다대다 관계(태이블 따로) (Like 이름설정,별칭: 좋아요눌러진 게시물)
    db.User.belongsToMany(db.Postm{through:'Like',as:"Liked"})
    db.User.belongsToMany(db.User, {
        through: "Follow",
        as: "Followers",
        foreignKey: "FollowingId",
      });
      db.User.belongsToMany(db.User, {
        through: "Follow",
        as: "Followings",
        foreignKey: "FollowerId",
      });
  };
  return User;
};
```

다대다 관계에서는 한 칼럼이 여러개의 아이디를 가질 수 없기 때문에 다른 태이블에서 관계들을 묶어줍니다. 

through는 그 태이블의 이름을 설정해 주는 것이고 상대 모델에서도 같게 작성해줘야 합니다. 

as는 생성된 관계 태이블에서 작성한 태이블과 관계된 id의 별칭을 설정해 줍니다. 

foreignKey는 관계테이블의 칼럼명이 userId라면 그 칼럼명을 바꿔줍니다 . 

다른 모델들도 마져 작성해보겠습니다 .

참고로 belongsTo에는 속한테이블이, hasOne,hasMany는 가진 테이블이기 때문에 외래키 칼럼은 belongsTo 즉, 속한 테이블에 생성이 되겠죠?

### Post 모델

```javascript
module.exports = (sequelize, DataTypes) => {
  const Post = sequelize.define(
    //대문자 단수가 소문자 복수로 저장
    "Post",
    //id는 기본으로 들어있음
    { content: { type: DataTypes.TEXT, allowNull: false } },
    {
      charset: "utf8mb4", //mb4-이모티콘
      collate: "utf8mb4_general_ci", //한글,이모티콘저장
    }
  );
  Post.associate = (db) => {
    //belongsTo는 UserId칼럼을 만들어줘서 참조관계를 만들어준다.
    db.Post.belongsTo(db.User); //개시글은 작성자에 속해있다.
    db.Post.belongsToMany(db.Hashtag, { through: "PostHashtag" }); //다대다 관계
    db.Post.hasMany(db.Comment); //게시글이 Comment를 많이 가질 수 있다.
    db.Post.hasMany(db.Image);
    db.Post.belongsToMany(db.User, { through: "Like", as: "Likers" }); //다대다 관계 (Like 이름설정,별칭: 좋아요누른사람)
    db.Post.belongsTo(db.Post, { as: "Retweet" });
  };
  return Post;
};
```

### Comment모델

```javascript
module.exports = (sequelize, DataTypes) => {
  const Comment = sequelize.define(
    //대문자 단수가 소문자 복수로 저장
    "Comment",
    //id는 기본으로 들어있음
    { content: { type: DataTypes.TEXT, allowNull: false } },
    {
      charset: "utf8mb4", //mb4-이모티콘
      collate: "utf8mb4_general_ci", //한글,이모티콘저장
    }
  );
  Comment.associate = (db) => {
    db.Comment.belongsTo(db.User); //댓글은 작성자에 속해있다.
    db.Comment.belongsTo(db.Post); //댓글은 작성자에 속해있다.
  };
  return Comment;
};
```

### Hashtag 모델 

Hash태그 모델은 다대다 관계가 햇갈릴 수 있습니다 . 

node태그가 있다면 node개시글이 많이 있을수도 있고, 

node개시글에는 많은 태그들이 달릴수도 있으니, 다대다 관계라고 이해해 주면 됩니다.

```javascript
module.exports = (sequelize, DataTypes) => {
  const Hashtag = sequelize.define(
    //대문자 단수가 소문자 복수로 저장
    "Hashtag",
    //id는 기본으로 들어있음
    { name: { type: DataTypes.STRING(20), allowNull: false } },
    {
      charset: "utf8mb4", //mb4-이모티콘
      collate: "utf8mb4_general_ci", //한글,이모티콘저장
    }
  );
  Hashtag.associate = (db) => {
    db.Hashtag.belongsToMany(db.Post, { through: "PostHashtag" }); //다대다 관계
  };
  return Hashtag;
};

```

### image모델

```javascript
module.exports = (sequelize, DataTypes) => {
  const Image = sequelize.define(
    //대문자 단수가 소문자 복수로 저장
    "Image",
    //id는 기본으로 들어있음
    { src: { type: DataTypes.STRING(200), allowNull: false } },
    {
      charset: "utf8",
      collate: "utf8_general_ci", //한글,이모티콘저장
    }
  );
  Image.associate = (db) => {
    db.Image.belongsTo(db.Post);
  };
  return Image;
};
```



## 모델 생성

다시 models 폴더에 돌아와서 모델들을 db에 담아줄 것입니다 . 

```javascript
const Sequelize = require("sequelize");
//환경변수
const env = process.env.NODE_ENV || "development";
//env는 현재 개발모드를 나타는중
//DB계정정보 가져오기
const config = require("../config/config")[env];
const db = {}; //일단 db 빈객체를 만들어줍니다

//연결
//config 파일안에 있는 정보들을 이용해 DB와 연결해준다.
const sequelize = new Sequelize(
  config.database,
  config.username,
  config.password,
  config
);

//모델생성
db.Comment = require("./comment")(sequelize, Sequelize);
db.Hashtag = require("./hashtag")(sequelize, Sequelize);
db.Image = require("./image")(sequelize, Sequelize);
db.Post = require("./post")(sequelize, Sequelize);
db.User = require("./user")(sequelize, Sequelize);

Object.keys(db).forEach((modelName) => {
  if (db[modelName].associate) {
    db[modelName].associate(db);
  }
});

db.sequelize = sequelize;
db.Sequelize = Sequelize;

//저장된 db를 내보내줍니다. 
module.exports = db;
```

## Sync

마지막으로 내보낸 db를 가지고  app.js에서 db를 sync해주면 마무리됩니다. 

```javascript
const express = require("express");
const userRouter = require("./routes/user");
const dotenv = require("dotenv");
//db를 받아와서
const db = require("./models");

const app = express();
const PORT = 3030;

dotenv.config();

//DB 싱크
db.sequelize
  .sync() //싱크를 해준뒤 아래에서 에러처리
  .then(() => console.log("연결성공"))
  .catch(console.error("에러발생"));

app.get("/", (req, res) => {
  res.send("페이지");
});

app.use("/user", userRouter);

app.listen(PORT, () => {
  console.log(PORT, "에서 서버실행중");
});

```

자 이제 노드를 통해 app.js를 실행해 보세요. 

터미널에 SQL문들이 마구 출력될 것입니다

