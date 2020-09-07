# 준비

설정을 먼저 해줘야 합니다. 

next 에서 제공하는 메서드 4개가 있는데, 리덕스와 함께 사용하면 몇가지 문제가 있기 때문에 

next-redux-wrapper에서 제공하는 것들로 SSR을 구형하겠스빈다. 



getInitialProps는 뭔가 곧 없어질 것 같기에 9번전부터 새로나온

getStaticProps getServerSideProps, getStaticPath



일단 

```javascript
//이렇게 해주면 이부분이 index보다 먼저 실행된다.
//화면을 그리기 전에 서버에서 이부분을 실행
export const getServerSideProps = wrapper.getServerSideProps((context) => {
  console.log(context); //context에는 무엇이 들어있을까?
  //기존에 useEffect에 있던걸 가져옴
  context.store.dispatch({
    type: LOAD_MY_INFO_REQUEST,
  });
  context.store.dispatch({
    type: LOAD_POSTS_REQUEST,
  });
});

export default index;

```

하지만 이렇게 해줘도 redux state는 비어있게 된다. 여기서 나오게 되는데 HYDRATE인데, 

HYDRATE는 위에서 작성한 내용이 실행되면 결과를 받는다. 

![&#xC5EC;&#xAE30;&#xC11C; payload&#xC548;&#xC5D0; &#xBC1B;&#xC544;&#xC9C4;&#xB2E4;.](.gitbook/assets/image%20%2813%29.png)

하지만, 지금도 정보가 안들어 있다. 

