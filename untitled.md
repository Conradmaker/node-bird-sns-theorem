# SSR 쿠키공유

게시글은 성공했지만, 로그인은 성공하지 않못했습니다. 

왜냐하면 브라우저에서 프론트-&gt;백 으로 갈때 CORS나 credentials 문제가 있었는데, 

지금도 그 문제가 생성되기 때문입니다. 

브라우저가 서버로 요청을 전송할때 header에 쿠키를 담아서 보낸다고 한것 기억나나요? 

하지만 SSR은 브라우저에서 실행되는게 아닌, 프론트서버에서 실행되기 때문에 

쿠키가 전송이 되지 않습니다...

일단 우리는 서버쪽에서는cors를 통해 credentials 를 설정 해주었습니다.  

이번시간에는 SSR시 쿠키를 직접 담아서 프론트서버에서 함께 요청을 보내는 방법을 알아보겠습니다. 

먼저 코드를 한번 볼까요?

```javascript
   //쿠키 전달
    const cookie = context.req ? context.req.headers.cookie : ""; //이 안에 쿠키 들어있음.
   
    //쿠키 공유 방지
    axios.defaults.headers.Cookie = ""; //아닐때는 쿠키 제거
    if (context.req && cookie) {//서버일때 & 쿠키가 있을때만    
      axios.defaults.headers.Cookie = cookie; //쿠키를 넣어주고
    }
```

이렇게 if문을 통해 쿠키의 공유를 막아주는 이유는 다음과 같습니다. 

만약 공유방지 처리를 안해주면 어떻게 될까요? 

앞자리에서 사용자가 로그인을 해서 쿠키를 만들어왔습니다. 

그런데 뒷자리에 있던 사용자가 새로고침을 눌렀는데, 앞자리사용자가 만들어 놓은 쿠키가 공유가 되서 앞자리 사용자로 로그인이 되어버렸습니다..

이런 상황을 방지해주기 위해 쿠키공유를 막아주는 것입니다. 

```javascript
//이렇게 해주면 이부분이 index보다 먼저 실행된다.
//화면을 그리기 전에 서버에서 이부분을 실행
export const getServerSideProps = wrapper.getServerSideProps(
  async (context) => {
    console.log("getServerSideProps start");
    console.log(context.req.headers);
    //쿠키 전달
    const cookie = context.req ? context.req.headers.cookie : ""; //이 안에 쿠키 들어있음.
    //쿠키 공유 방지
    axios.defaults.headers.Cookie = ""; //아닐때는 쿠키 제거
    if (context.req && cookie) {
      //서버일때 & 쿠키가 있을때만
      axios.defaults.headers.Cookie = cookie; //쿠키를 넣어주고
    }

    //console.log(context); //context에는 무엇이 들어있을까?
    //기존에 useEffect에 있던걸 가져옴
    context.store.dispatch({
      type: LOAD_MY_INFO_REQUEST,
    });
    context.store.dispatch({
      type: LOAD_POSTS_REQUEST,
    });
    context.store.dispatch(END); //saga에서 불러옴 (끝날때까지 기다리라고)
    console.log("getServerSideProps end");
    await context.store.sagaTask.toPromise(); //(sagaTask)는 스토어에서 등록함
  }
);

export default Home;

```



