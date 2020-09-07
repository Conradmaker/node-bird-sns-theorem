# 팔로우 & 언팔로우\(프로필 페이지상\)

저번시간에는 게시글에 있는 버튼을 통해 팔로우 언팔로우 기능을 구현하는 작업을 진행하였습니다. 

이번에는 프로필 페이지에 있는 팔로우 언팔로우 리스트를 통해 구현하는 작업을 해보도록 하겠습니다. 

특징은 다음과 같습니다 . 

* 일단 프로필 페이지에서 리스트 로드
* 팔로우/팔로잉 리스트가 같은 컴포넌트이기 때문에 header에 따른 다른 dispatch
* 반복문 속 고차함수 이용
* 저번시간에 팔로잉 \(내가 상대방을 팔로우\) 구현했기 때문에 지우는 것도 이용
* 상대방이 날 팔로우한걸 차단하는 기능\(팔로워 삭제\) 구현

## 프로필 페이지에서 팔로워 / 팔로잉 리스트 로드

프로필 페이지에서는 다음 두 액션을 dispatch해 서버로부터 데이터를 불러옵니다.

* LOAD\_FOLLOWERS\_REQUEST
* LOAD\_FOLLOWINGS\_REQUEST

물론 이에 대한 reducer나 saga도 작성해줘야 합니다. 

컴포넌트가 마운트될때 dispatch가 일어날 수 있도록 useEffect를 사용해주겠습니다.

```javascript
  useEffect(() => {
    dispatch({ type: LOAD_FOLLOWERS_REQUEST });
    dispatch({ type: LOAD_FOLLOWINGS_REQUEST });
  }, [dispatch]);
```

그리고 백앤드 user 라우터에서 목록을 보내주도록 하겠습니다. 

단계는 다음과 같습니다. 

* 조회 요청을 보낸 로그인된 유저정보를 가져온뒤
* 없으면 에러발생,
* 있다면 그 유저의 팔로워를 sequelize관계메소드로 가져와서
* 프론트로 보내준다. 
* 팔로잉 목록도 이와 같습니다. 

### 프로필 팔로워 가져오기

`GET /user/followers`  

```javascript
router.get('/followers',isLoggedIn, async(req,res,next)=>{
    try {
        //조회 요청을 보낸 즉, 로그인된 유저 정보를 먼저 찾는다. 
        const user = await User.findOne({where:{id:req.user.id}});
        if(!user){return res.status(403).send('없는 유저')};
        //있다면 Sequelize관계메소드를 이용해 목록을 가져온다. 
        const followers = await user.getFollowers();
        //결과물을 전송한다.
        res.status(200).json(followers);
    } catch (e) {
        console.error(e);
        next(e);
    }
})
```

### 프로필 팔로잉 가져오기

방법은 팔로워와 동일합니다. 

관계태이블에 as를 통해 Followings/ Followes로 구분했기 때문에 이번에는 Followings를 가져오면 됩니다. 

이번에는 추가적으로 가져올 Following의 유저데이터를 좀 커스텀 해주겠습니다. 

```javascript
router.get('/followings',isLoggedIn,async(req,res,next)=>{
    try {
        //조회 요청을 보낸 즉, 로그인된 유저 정보를 먼저 찾는다. 
        const user = await User.findOne({where: {id:req.user.id}});
        if(!user){return res.statud(404).send('없는유저입니다.')};
        //있다면 Sequelize관계메소드를 이용해 목록을 가져온다. 
        const followings = user.getFollowings({
            model:User,
            attributes:['id','nickname'],
        });
        //커스텀은 데이터들을 보내준다. 
        res.status(200).json(followings);
    } catch (e) {
        console.error(e);
        next(e);
    }
})
```

이렇게 데이터들을 줄여주게 된다면 우리는 큰 장점을 가지고 갈 수 있습니다. 

* 서버간 데이터통신의 크기를 줄일 수 있다. 
* 유저의 필요한 정보만 보내지기 때문에 보안을 강화할 수 있다.

## 프로필 페이지 안에서의 언팔로우 / 팔로워차단 구현

먼저 FollowList컴포넌트 입니다.  

```javascript
  const dispatch = useDispatch();
  const onCancel = (id) => {
    if (header === "팔로잉") {
      dispatch({
        type: UNFOLLOW_REQUEST,
        data: id,
      });
    } else {
      dispatch({
        type: REMOVE_FOLLOWER_REQUEST,
        data: id,
      });
    }
  };
```

header에는 '팔로잉' / '팔로우' 두가지 property중 하나가 이용되기 때문에 그를 이용한 조건을 걸어주었습니다. 

만약 내가 상대방을 팔로우한 팔로잉이 아니면 나를 팔로우한 상대방에게서 지우는 기능을 구현합니다. 

컴포넌트렌더링에서는 다음과 같이 해줍니다.

```javascript
 {/*반복문 안에서 온클릭같은게 있다면 반복문에 대한 데이터를 넘겨줄때가 있는데 이때 고차함수 유용 */}
<Card
  actions={[
    <StopOutlined key="stop" onClick={() => onCancel(item.id)} />,
//onClick={onCencel(item.id)}이렇게 하고, const onCencel=(id)=>()=>{}이렇게도 가능
  ]}
>
```

그렇다면 이제 상대가 팔로우한 나와의 관계를 다시 취소시켜 볼까요? 

`DELETE user/follower/:id`    

```javascript
router.delete('follower/:id',isLoggedIn, async(req,res,next)=>{
    try {
        
    } catch (e) {
        console.error(e);
        next(e);
    }
})
```

* 여기서 프론트에서 보내주는 URI parameter의 id는 상대방의 id여야 합니다. 
* 상대방 유저의 모델로 들어가 목록중 나를 삭제해 주어야 하기 때문입니다. 

```javascript
// 여기서 받아올 URI parameter의 id는 상대방의 id입니다. 
// 상대방 유저를 조회해 그중 상대방이 팔로우한 목록에서 로그인한 유저를 지워야 하기 때문입니다.
router.delete('follower/:id',isLoggedIn, async(req,res,next)=>{
    try {
        //상대방 User조회
        const user = await User.findOne({where:{id:req.params.id}});
        //상대방이 없다면 에러
        if(!user){return res.status(403).send('상대방 유저가 없습니다.')};
        //상대방입장에서 나를 팔로잉한것이기 때문에 following중 나를 삭제
        await user.removeFollowing(req.user.id);
        //리덕스 상태에서 상대방 아이디를 filter해주기 위해 보내준다.
        res.status(200).json({id:parseInt(req.params.id, 10)}); 
    } catch (e) {
        console.error(e);
        next(e);
    }
});
```

생각보다 복잡하지는 않지만 router가 많아지면 많아질수록 뭔가 햇갈리는 부분들이 많아집니다. 

주의하고 신중히 생각을 하며 작성해야만 합니다. 

