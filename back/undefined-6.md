# 쿠키공유

## Credential

자 저번에 로그아웃 구현시 만든 미들웨어에서 로그인 되었지만, 로그인해달라고 하는 건 서버에서 

쿠키를 받지 못하고, 로그인 유무를 확인 할 수 없었기 때문입니다. 

그 이유는 CORS와 같습니다.  CORS문제는 언제 생성되었죠?

* 요청 도메인과 서버 도메인이 달라서 브라우저에서 막은것. 

쿠키도 마찬가지 입니다. 도메인이 다르면 쿠키가 서버로 전달되지 않습니다. 

해결법도 프록시 등 여러가지가 있겠지만, 우리는 저번에 CORS에서 활용한, cors라이브러리를 활용해 문제를 해결해 보도록 하겠습니다. 

## 서버측

```javascript
app.use(cors({ origin: "*" }));
```

전에 모든 도메인의 요청을 허용하기 위해 \*을 넣어주었는데요, credentials를 추가해주어야 합니다.

```javascript
app.use(
  cors({
    origin: "http://localhost:3000",
    credentials: true, //쿠키공유
  })
);
```

여기서 \*을 도메인으로 바꿔준 이유는 쿠키공유를 하게 되면 보안을 위해 cors에서 정확한 도메인을 입력하라는 에러가 나오게 됩니다. 

`origin:true`를 입력해도 괜찮습니다.



## 프론트

saga에서 axios요청을 보내게 되는데 

다음과 같이 3번째 인자에 `withCredentials:true` 를 입력해 쿠키를 함께 보낼 수 있습니다. 

GET요청과 같이 data를 보내지 않는 요청에서는 두번째 인자입니다.

```javascript
async function addCommentAPI(data) {
  const response = await axios.post(`/post/${data.postId}/comment`, data, {
    withCredentials: true,
  });
  return response.data;
}
```

하지만 , 수백개의 요청을 보내게 될 수도 있는데, 이런 방법은 너무 비 효율적입니다. 

saga/index.js에서 한번에 할수도 있습니다. 

```text
axios.defaults.baseURL = "http://localhost:3030";
axios.defaults.withCredentials = true;
```

자 이제 쿠키를 공유할 수 있기 때문에 서버에서도 로그인 쿠키를 받을 수 있게 되었습니다. 

이제 새로고침시에 로그인이 풀려버리는 문제를 해결할 수 있겠죠??



