# 닉네임 변경 & 게시글 삭제

## 닉네임 변경

닉네임을 변경하는 것은 아주 간단합니다. 

* 프론트에서 서버로 변경할 닉네임을 날려준다. 
* 서버에서 DB를 Update한다.
* 프론트로 다시 변경된 결과와 닉네임을 날려준다. 
* 받을 닉네임을 state에 적용해준다. 

자 직접 적용해볼까요?

먼저 프론트 서버의 닉네임 폼의 구조를 볼게요

antd를 적용했기 때문에 다음과 같습니다. 

```javascript
 const onSubmit = () => {
    dispatch({
      type: CHANGE_NICKNAME_REQUEST,
      data: nickname,
    });
  };
  
 <Form style={style}>
      <Input.Search addonBefore="닉네임" enterButton="수정" />
      <Input.Search
        addonBefore="닉네임"
        enterButton="수정"
        value={nickname}
        onSearch={onSubmit}
        onChange={onChange}
      />
    </Form>
```

자 dispatch를 할때 data에 nickname이 들어있고, axios를 통해 저 data를 보내주는 형식입니다.

백앤드 router를 짜볼까요?

URI는 `PATCH /user/nickname` 입니다.

먼저 구조를 잡아준뒤

```javascript
router.patch('/nickname',isLoggedIn,(req,res,next)=>{
  try {
    
  } catch (e) {
    console.error(e);
    next(e);
  }
})
```

Sequelize에서 UPDATE문을 구현하는 것은 매우 간단합니다. 

* update메소드를 이용해 변경할것을 입력한뒤
* 동시에 id를 통해 변경하고자 하는 데이터를 조건을 걸어주면 됩니다. 

```javascript
router.patch('/nickname',isLoggedIn,(req,res,next)=>{
  try {
    //update는 수정 메소드
    await User.update(
      {nickname: req.body.nickname}, //받아온 데이터를 nickname에 넣어주고
      {where:{id:req.user.id}}   //로그인된 유저를 수정한다는 것
    );
    //리덕스 상태수정을 위해 성공시 nickname을 보내준다.
    res.status(200).json({nickname:req.body.nickname});
  } catch (e) {
    console.error(e);
    next(e);
  }
})
```

너무 쉽죠? 

## 게시글 삭제

이번에는 게시글 삭제입니다.  

post.js라우터로 이동해주세요.

삭제나 수정을 할때에는 다른작업보다 관리를 철저히 해줘야 하는데,

이는 개인정보의 도용이나 다른 아이디의 도용을 통해 악성 게시물을 작성하거나 삭제할 수도 있기 

때문입니다. 

여기서는 간단히 조회시에 조건만을 이용하도록 하겠습니다. 

URI는 `DELETE post/:id` 입니다.

참고로 Sequelize에서 데이터 삭제를 할때는 destroy라는 메소드를 사용합니다 .

```javascript
router.delete('/:id', isLoggedIn, async(req,res,next)=>{
  try {
    //destroy : Sequelize삭제 메소드
    await Post.destroy({
      where:{
        id:req.params.id,
        //게시글의 작성자와 로그인된 사용자가 같아야함
        UserId: req.user.id, 
      }
    });
    res.status(200).json({id:parseInt(req.params.id, 10)}); //params는 문자열이라 형변환
  } catch (e) {
    console.error(e);
    next(e);
  }
})
```

URI parameter로 담겨져 오는 데이터는 모두 문자열이라는 사실을 잊지 마세요!

