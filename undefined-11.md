# 업로드 이미지 미리보기 / 삭제

네트워크에서 response를 보면 경로가 front서버 즉 3000번대로 되있는데, 

이미지는 백앤드서버 즉 포트 3030에 저장되어있습니다. 

이를 백앤드 서버로 변경시켜주면 이미지를 미리 볼 수 있습니다. 

```javascript
 <img
   src={`http://localhost:3030/${v}`} //미리보기 경로
   style={{ width: "200px" }}
   alt={v}
 />
```

v에는 백앤드 서버에서 보내준 파일명이 담겨있는데, 그 앞에 경로를 적어주면 접근할 수 있겠죠?

이렇게 front 컴포넌트에 경로를 수정해주는 것만 아니라,

express에서도 정적 파일에 대한 관리가 필요한데, static을 사용할 수 있습니다. 

app.js로 이동해서 다음과 같이 작성을 추가해주세요

```javascript
const path = require("path");

//'/'는 localhost:3030/을 가르킨다.
//path.join하면 __dirname(back)에 'uploads'를 합쳐준다.
//이렇게 하면 서버쪽 폴더구조가 가려져서 보안에 유리하다.
app.use("/", express.static(path.join(__dirname, "uploads")));
```

또한 이렇게 경로를 path를 이용해 경로를 지정해주는 이유는 

Mac OS와 Window OS가 서로 경로를 표시하는 방법이 다르기 때문에 path를 사용하는 것입니다. 

이러면 이제 사진선택시에 미리보기 이미지가 보일 것입니다. 



## 업로드 이미지 취소 

삭제시에는 보통 프론트상에서는 삭제되지만,  백앤드에는 그냥 남겨놓는 경우가 많습니다. 

왜냐하면 그 데이터 용량에 대한 비용보다는 그 이미지 자원으로써의 가치가 더 높기 때문입니다. 

프론트에서만 삭제를 만들어 보겠습니다. 

동적 작업이기 때문에 action을 하나만 주면 되기 때문에 간단합니다. 

```javascript
  const onRemoveImage = useCallback((index) => {
    dispatch({
      type: REMOVE_IMAGE, //동기액션
      data: index,
    });
  });


{imagePath.map((v, i) => (
    <Button onClick={() => onRemoveImage(i)}>제거</Button>
)}
```

저번시간과 같이 맵 안에 콜백에 데이터를 넣고 싶으면 고차함수로구현해주어야 합니다. 

reducer

```javascript
export const REMOVE_IMAGE = "users/REMOVE_IMAGE";

//아래부분 reducer함수에 추가
      case REMOVE_IMAGE:
        //프론트에서만 빼준다.
        draft.imagePath = draft.imagePath.filter((v, i) => i !== action.data);
        break;
```



