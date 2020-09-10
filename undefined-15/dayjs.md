# dayjs

이번에는 간단히 날짜를 나타낼수 있는 라이브러리인 dayjs를 알아보겠습니다. 

사실 거의 유사한 라이브러리인 moment를 많이 들어보셨을 것이에요.

아래 npm-trends를 한번 볼까요?

![](../.gitbook/assets/image%20%2814%29.png)

아직까지는 moment가 사용량이 많기는 하지만, dayjs도 꾸준히 증가하는 중입니다. 

star갯수를 보세요. 2018년에 나온 dayjs가 거의 따라잡은 모습을 볼 수 있습니다. 

사실 사용법은 그냥 같다고 봐도 무방합니다. 왜 이렇게 dayjs가 인기가 좋을까요? 

이유는 크기의 차이입니다.  2.8kb vs 70.4kb는 매우 큰 차이입니다.  \(언어팩때문임\)

실제로 빌드해보면 그 차이를 실감할 수 있습니다. 

한번 사용해 볼까요? 

postcard가 뭔가 허전하니 날짜를 추가해주도록 하죠!

먼저 설치를 해주어야 합니다!

```text
npm i dayjs
```

그뒤 불러와야 겠죠??

단, 한국어를 사용하고 싶다면 한국어도 함께 불러와주어야 합니다. 

이게 moment와 비교해서 가장 큰 장점이라고 생각합니다. 

```javascript
import dayjs from "dayjs";
import "dayjs/locale/ko";

dayjs.locale("ko"); //한국설정
```

그뒤에 사용하고 싶은 공간에 넣어주면 끝입니다. 

```javascript
<div style={{ float: "right" }}>
    {dayjs(post.createdAt).format("YYYY.MM.DD")}
</div>
```

`dayjs(날짜)`가 기본 형식인데, format으로 형식을 설정해 줄 수도 있으며, 지난날짜등 여러가지 매소드가 포함되어 있으니 공식문서에서 확인해보기를 추천합니다. 

