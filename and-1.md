# 팔로우 & 언팔로우\(게시글상에서\)

우리는 두개의 페이지에서 팔로우와 언팔로우를 할 수 있습니다. 

* profile 페이지
* index 페이지 게시글상

이번에는 먼저 게시글 상에서 팔로우와 언팔로우를 할 수 있도록 기능을 구현해보도록 하겠습니다 . 

여기서 작성한 언팔 팔로우는 나 -&gt; 상대방으로 가는 팔로우 기능에 대한 내용이며, profile페이지에서도 같은 router로 사용됩니다. 

## 팔로우기능

* 상대유저가 존재하는지 확인
* 없는 유저면 에러발생
* 있다면, 관계데이터 Follower추가
* 프론트서버로 전송

URI는 `POST user/:id/follow` 입니다.

언제나 그렇듯 기본 틀을 만들어야겠죠?

```javascript
router.patch('/:id/follow',isLoggedIn,async(req,res,next)=>{
    try {
        
    } catch (e) {
        console.error(e);
        next(e);
    }
});
```

이번에는 유저가 있는 유저인지를 체크하고, 없다면 에러를 발생시켜줍니다. 

```javascript
router.patch('/:id/follow',isLoggedIn,async(req,res,next)=>{
    try {
    const exUser = await User.findOne({
        where: {id: req.params.id},
    });
    if(!exUser){return res.status(403).send('없는 유저입니다.')};
    } catch (e) {
        console.error(e);
        next(e);
    }
});
```

다음은 있는 유저일 경우 팔로워에 추가해주는 것입니다. 

Sequelize에서 생성한 메소드를 이용해 쉽게 구현해 줄 수 있습니다 .

```javascript
router.patch('/:id/follow',isLoggedIn,async(req,res,next)=>{
    //팔로우 하려는 사용자가 존재하는지 확인
    try {
    const exUser = await User.findOne({
        where: {id: req.params.id},
    });
    //없다면 에러발생
    if(!exUser){return res.status(403).send('없는 유저입니다.')};
    //있다면 팔로워 추가
    await exUser.addFollowers(req.user.id);
    //리덕스 상태관리를 위해 필요한 데이터 프론트로 전송
    res.status(200).json({id:parseInt(req.params.id,10)});
    } catch (e) {
        console.error(e);
        next(e);
    }
});
```

## 언팔로우도 같은 방식으로 구현해 줄 수 있습니다. 

URI는 `DELETE user/:id/follow` 입니다.

```javascript
router.delete('/:id/follow',isLoggedIn, async(req,res,next)=>{
    try {
        const exUser = await User.findOne({
            where:{id: req.params.id},
        });
        if(!exUser){return res.status(403).send('없는 유저입니다.')};
        //유저가 있다면 팔로워에서 제거
        await exUser.removeFollowers(req.user.id);
        //프론트로 필요한 데이터 전송
        res.status(200).json({id: parseInt(req.params.id,10)});
    } catch (e) {
        console.error(e);
        next(e);
    }
})
```



