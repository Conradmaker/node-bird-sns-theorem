# 로그인 정보 커스텀

```javascript
const express = require("express");
const { User } = require("../models");
const bcrypt = require('bcrypt');
const passport = require("passport");

const router = express.Router();

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
    })(req,res,next) //미들웨어 확장법
})
```

이렇게 로그인정보를 넘겨줬습니다. 

하지만, password가 섞여있고 필요한 정보가 추가되지 않아있습니다. 

성공시에 user를 커스텀해주도록 하겠습니다. 

```javascript
const express = require("express");
const { User } = require("../models");
const bcrypt = require('bcrypt');
const passport = require("passport");

const router = express.Router();

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
            // 내가 원하는 로그인 데이터 넣고 빼주기
            const fullUserWithoutPassword = await User.findOne({
                where:{id:user.id}, //user.id가 같은 유저를 선택해
                //비밀번호 제거
                attribute:{exclude:['password']},
                //관계데이터 추가
                include:[
                    {model:Post},
                    {model:User,as:'Following'}, //모델생성시 as써줬으면 여기서도 써야한다.
                    {model:User,as:'Followers'},
                ],
            });
            return res.status(200).json(fullUserWithoutPassword)//커스텀한 user를 json으로 보내준다. 
        });
    })(req.res.next);
})
```

이제 로그인 과정이 모두 끝났습니다.

복잡하지만, 순서를 잘 보고 따라오면 이해하기 그나마 괜찮을 것같습니다...



