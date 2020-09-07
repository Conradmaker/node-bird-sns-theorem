# 로그인 문제 해결

우리는 지금 로그인할때 두가지 문제를 가지고 있습니다. 

1. 새로고침시에 로그인이 풀려버리는 문제
2. 로그인 성공시 받아지는 데이터가 너무 큰 문제

## 새로고침시 로그인 풀리는 문제

원래는 브라우저에서 쿠키를 가지고 있기 때문에 서버로 다시 쿠키를 보내주기만 한다면 다시 로그인을 할 수 있습니다. 

그렇 어떻게 구현해야 할까요?

새로고침을 한 뒤에 쿠키를 서버로 보내주면 되겠죠? 

우리는 credentials를 설정해 줬기 때문에 쿠키를 서버로 보낼 준비가 되어있습니다. 

새로운 saga와 reducer를 작성해줘야 겠네요

pages/index.js에서 useEffect를 이용해봅시다. 

### 페이지 접속시 로그인정보 요청

```javascript
  useEffect(() => {
    dispatch({ type: LOAD_MY_INFO_REQUEST });
    dispatch({ type: LOAD_POSTS_REQUEST });
  }, [dispatch]);
```

페이지 마운트될때 dispatch를 해준뒤 ,

### Reducer

```javascript
export const LOAD_MY_INFO_REQUEST = "users/LOAD_MY_INFO_REQUEST";
export const LOAD_MY_INFO_SUCCESS = "users/LOAD_MY_INFO_SUCCESS";
export const LOAD_MY_INFO_FAILURE = "users/LOAD_MY_INFO_FAILURE";

export const initialState = {
  loadUserDone: false,
  loadUserError: null,
  loadUserLoading: false,
}

export default function reducer(state = initialState, action) {
  return produce(state, (draft) => {
    switch (action.type) {
      case LOAD_MY_INFO_REQUEST:
        draft.loadUserLoading = true;
        draft.loadUserDone = false;
        draft.loadUserError = null;
        break;
      case LOAD_MY_INFO_SUCCESS:
        draft.loadUserLoading = false;
        draft.loadUserDone = true;
        draft.loadUserError = null;
        draft.me = action.data;
        break;
      case LOAD_MY_INFO_FAILURE:
        draft.loadUserError = action.error;
        draft.loadUserLoading = false;
        draft.loadUserDone = false;
        break;
    }
  }
}
```

### saga

```javascript
async function loadUserAPI() {
  const response = await axios.get("/user");
  return response.data;
}

function* loadUser() {
  try {
    const data = yield call(loadUserAPI);
    yield put({
      type: LOAD_MY_INFO_SUCCESS,
      data,
    });
  } catch (e) {
    yield put({
      type: LOAD_MY_INFO_FAILURE,
      error: e.response.data,
    });
  }
}

function* watchLoadUser() {
  yield takeEvery(LOAD_MY_INFO_REQUEST, loadUser);
}

export default function* userSaga() {
  yield all([
    fork(watchLoadUser),
  ]);
}
```

### 서버측 구현

```javascript
router.get('/',async(req,res,next)=>{
    try{
        if(req.user){//passport에서 유저 접근
            //사용자가 있다면
            const fullUserWithoutPassword = await User.findOne({
                where:{id:user.id},
                attributes:{exclude:['password']},//비밀번호 빼고
                //관계데이터 추가
                include:[
                    {model:Post},
                    {model:User,as:"Followings"},
                    {model:User,as:"Followers"}
                ]
            });
            res.status(200).json(fullUserWithoutPassword); //쿠키 있으면 보내고
        }else{
            res.status(200).json(null);//없으면 보내지 않는다.
        }
    }catch(e){
        console.error(e);
        next(e);
    }
})
```

지금까지 구현한것만 해도 사실 문제는 없습니다. 

하지만 아래의 문제도 해결해야 합니다.

## 로그인시 받아지는 데이터 수정

현재 로그인하면 사용자 데이터가 날라올때 Post,User에 대한 정보들이 모두 날라오는데, 

이는 모바일로 가게되면 엄청난 데이터 낭비입니다. \(만약 개시글이 만개 있다고 가정해보자\)

그렇기 때문에 데이터를 걸러서 보내주면 되겠죠??? 

어차피 우리는 프론트에서 팔로워수, 팔로잉수, 개시글수를 위해 정보를 가져오니까id만 가져오겠습니다.

```javascript
router.get('/',async(req,res,next)=>{
    try{
        if(req.user){//passport에서 유저 접근
            //사용자가 있다면
            const fullUserWithoutPassword = await User.findOne({
                where:{id:user.id},
                attributes:{exclude:['password']},//비밀번호 빼고
                //****관계데이터 추가**** [id만 가져오기]
                include:[
                    {model:Post, attributes: ["id"] },
                    {model:User,as:"Followings", attributes: ["id"] },
                    {model:User,as:"Followers", attributes: ["id"] }
                ]
            });
            res.status(200).json(fullUserWithoutPassword); //쿠키 있으면 보내고
        }else{
            res.status(200).json(null);//없으면 보내지 않는다.
        }
    }catch(e){
        console.error(e);
        next(e);
    }
})
```

지금은 새로고침 하면 로그인화면이 잠깐 보였다가 보이는데,

이건 SSR이 필요하기 때문에, 나중에 구현해보도록 하겠습니다. 

참고로 서버가 재시작되면 서버에 저장된 세션이 사라지기 떄문에 로그인이 풀려버리게 됩니다. 

