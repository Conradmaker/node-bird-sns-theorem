# swr

지금까지 리덕스를 사용하면서, 타입정하고, 액션만들고, 리듀서만들고, 사가만들고, axios만들고 너무너무 할게 많고 반복되었습니다. 

그래서 이번에는 적어도 게시글이나 정보들을 LOAD하는 즉, get메소드에 있어서만큼은 작업을 조금은 편하게 할 수 있는 next개발팀의 swr라이브러리에 알아보도록 해요. 

참고로 swr은 서버사이드 렌더링도 된답니다!

먼저 설치를 해줘야 합니다. 

```text
npm i swr
```



